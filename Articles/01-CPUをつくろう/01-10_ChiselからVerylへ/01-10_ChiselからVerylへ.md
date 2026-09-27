# ChiselからVerylへ――CPUを別の言語へ移植した話

> タグ: CPU自作 / Chisel / Veryl / HDL / 移植

ここまでの記事では、Chiselで回路を作ってきました。森羅万象プロジェクトの実際の開発では、CPU本体やUART、SPIなどをChiselで実装したあと、設計をVerylへ移植しました。

この記事はVerylの完全な文法解説ではありません。同じ回路を別のハードウェア記述言語で表すとはどういうことか、そして私たちがなぜ移植を選んだのかを紹介します。

## 移植した時期

コミット履歴では、2024年12月7日にVeryl用ディレクトリとシミュレーション環境が追加され、12月22日にCore、ALU、UART、IOBusなどの移植がまとまって入りました。その後もLoad/Storeや各命令のテスト、UART、SPIを順次整備し、2025年4月5日の記録で主要部のPure Veryl化を一区切りとしています。

これは一度に全コードを書き換えたのではなく、Chisel版を参照しながらモジュールごとに移した作業です。

## Verylとは

[Veryl](https://veryl-lang.org/)は、SystemVerilogを土台に設計されたハードウェア記述言語です。VerylのソースはSystemVerilogへ変換でき、既存のシミュレータや論理合成ツールと組み合わせられます。

ChiselとVerylの大まかな違いは次の通りです。

| 観点 | Chisel | Veryl |
| --- | --- | --- |
| 記述の土台 | Scala上のハードウェア構築言語 | SystemVerilogを土台にしたHDL |
| 変換先 | 回路表現を経てVerilog/SystemVerilog | SystemVerilog |
| 得意な表現 | Scalaによる抽象化や回路生成 | HDLとして回路構造を直接表す |
| 共通点 | どちらも最終的にデジタル回路を記述する | どちらもクロック、信号幅、組み合わせ・順序回路を扱う |

どちらが常に優れているという比較ではありません。プロジェクトの目的、参加者の経験、ツール、デバッグ方法に合わせて選びます。

## 同じALUを比べる

Chiselでは、ALUの入出力を `IO` で宣言し、`MuxLookup`などで出力を選べます。

```scala
class Alu extends Module {
  val io = IO(new Bundle {
    val command = Input(UInt(3.W))
    val a       = Input(UInt(32.W))
    val b       = Input(UInt(32.W))
    val out     = Output(UInt(32.W))
  })

  io.out := MuxLookup(io.command, 0.U)(Seq(
    1.U -> (io.a + io.b),
    2.U -> (io.a - io.b)
  ))
}
```

Verylでは、モジュールのポートと `case` 式をHDLらしい形で書けます。

```veryl
module Alu (
    i_command: input  logic<3>,
    i_a      : input  logic<32>,
    i_b      : input  logic<32>,
    o_out    : output logic<32>,
) {
    assign o_out = case i_command {
        3'd1   : i_a + i_b,
        3'd2   : i_a - i_b,
        default: 32'b0,
    };
}
```

見た目は変わっても、作りたい回路は同じです。

- `command`は3 bit
- `a`と`b`は32 bit
- `command=1`なら加算
- `command=2`なら減算
- それ以外なら0

移植では、行を機械的に置き換えるだけでは足りません。それぞれの言語で、どの記述が組み合わせ回路やレジスタになるかを確認する必要があります。

## なぜ移植したのか

2024年末の作業記録では、Chisel版のLoad/Storeがシミュレーションでは通る一方、FPGA実機で期待通り動かない問題を調査していました。メモリ周辺を調べる過程で、別の記述とツール構成を試す判断をし、Coreを中心にVerylへ移しました。

ここから「Chiselにバグがあった」と断定することはできません。原因としては、回路記述、生成されたRTL、FPGAのメモリ推論、初期化、タイミング制約など複数の可能性があります。記録上、原因を一般化できるところまで切り分けられてはいません。

私たちにとっての移植は、問題を調べながら設計を読み直し、今後の開発環境を選び直す工程でもありました。

## 移植するときに守ったもの

言語を変えてもCPUの外から見える仕様は維持します。

- 命令のbit配置
- opcodeと命令の意味
- レジスタの役割
- UART、SPI、GPIOなどの入出力
- クロックごとの状態変化

同じ命令列に対して同じ結果になるか、命令ごとのテストで確かめます。加算だけ動いても、符号付き右シフト、分岐、Load/Storeのような境界条件で差が出ることがあります。

実際の移植後も、2025年1月から2月にかけて算術・論理演算、分岐、メモリ操作、GPIOなどのテストが追加されました。移植直後を完成とせず、機能ごとに確認したことが大切です。

## Chiselを学んだことは無駄にならない

Chiselで学んだ次の考え方は、そのままVerylでも使います。

- 信号には決まったbit幅がある
- 組み合わせ回路と順序回路を区別する
- レジスタはクロックで更新される
- CPUはFetch、Decode、Executeなどのデータパスで考えられる
- テストで入力と期待値を決め、波形も確認する

言語は回路を表す道具です。回路そのものを理解していれば、別のHDLを読むときも「この信号は何を選び、このレジスタはいつ更新されるか」という視点を持てます。

## ここから先

完成版のVeryl実装には、CPUコアだけでなくUART、SPI、GPIO、映像出力、メモリ、キャッシュなどが加わりました。次の記事群では、まずChiselで各回路の考え方を学び、節目でVeryl版の同じモジュールと見比べます。

---

### 参考資料

- [Veryl公式サイト](https://veryl-lang.org/)
- [Veryl公式ドキュメント](https://doc.veryl-lang.org/book/)
- [Chisel公式ドキュメント](https://www.chisel-lang.org/docs)
- [森羅万象CPUリポジトリ](https://github.com/shinrabansyo/cpu)

