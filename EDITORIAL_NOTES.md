# 査読メモ（公開対象外）

このファイルは記事の査読用メモです。公開用記事へ含める前に削除または除外してください。

## 今回の編集範囲

- 既存の導入4記事を初学者向けに全面改稿
- シリーズ案内とロードマップを追加
- ChiselからVerylへの移植記事を追加
- 対応するCPU-1～CPU-3のコードを段階教材として再構成
- 2023年秋から2026年2月までの開発を、CPU・周辺回路・システムの16記事として追加
- 2026年2月後半のホームページ制作、ISA v2以降は今回の記事から除外
- 調達が必要な図や写真は `画像差し込み候補` と仮ファイル名を本文へ記載

## 参照した一次資料

- `shinrabansyo/cpu` のコミット履歴と各時点のソース
- `shinrabansyo/spec` の `spec_v1.adoc`、`spec_v2.adoc`
- `footprint/footprint.txt` の定期MTG記録
- Chisel公式ドキュメント
- Veryl公式ドキュメント
- SD AssociationのSimplified Specifications
- UEFI SpecificationのFAT関連章
- ArmのAMBA AXI公式解説

仕様リポジトリは、確認のため `ShinrabansyoArticles/spec-reference` にcloneした。記事用・コード用のリポジトリには変更していない。

## 修正した技術上の問題

1. 旧CPU-2はI形式の `rs1` を `instr(17, 13)` の5 bitとして読み、`imm=instr(47, 16)` と2 bit重複していた。仕様ではI形式の `rs1` は `instr(15, 13)` の3 bit。
2. 旧サンプルは `r0`へ3を書き込んでいた。現仕様の呼び出し規約では `r0`は常に0のゼロレジスタ。新CPU-3は `r4`、`r5`、`r6`を使う。
3. 旧記事には「加算器だけの回路」をCPUと呼ぶ表現があった。加算器はCPUの構成要素として説明し直した。
4. 「Chiselを実行するとハードウェアができる」という表現を、回路表現・RTL生成・シミュレーション・FPGA実装の段階に分けた。
5. Tang Nano 9KのUSB 5 Vを根拠に論理しきい値を2.5 Vとする記述を削除した。USB電源電圧とFPGA I/O規格の論理しきい値は同一ではない。
6. x86の命令長を「1 bitから15 bit」とする記述を削除した。一般にx86命令長は1～15 byteであり、bitではない。
7. Veryl移植の理由は、記録で確定できる範囲に限定した。Chisel自体の不具合とは断定していない。
8. UARTは非同期通信として記述し、idle/start/data/stop bitと受信信号の同期を分けて説明した。
9. FAT16は読み出し、SFN/LFN、絶対パス、複数clusterの確認までを完成範囲とした。書き込み、FAT32、GPT bootは構想と区別した。
10. 2026年2月のcache設定は、line 32 byte、I-cache 4 KiB・1 way、D-cache 4 KiB・2 wayとして `develop` branchから確認した。
11. pipelineは `Fetch`、`Decode`、`Execute`、`MemAccess`、`Writeback` の5段、forwarding、load-use stall、branch flushをsourceで確認した。
12. `axi_*` というmodule名だけから規格準拠を断定せず、記事では「AXI風のsubset」とした。
13. `malloc` は作業記録上の動作確認だけを完成事項とした。allocator内部方式はsourceを確認できていないため一般概念と区別した。

## 公開前に人が確認する点

- Chisel 3.6.1を教材として固定し続けるか、最新版へ別途移行するか
- Windows、macOS、Linuxそれぞれの環境構築手順をどこまで掲載するか
- 記事URLを維持するため既存ディレクトリ名を残すか
- 画像を新しい本文に合わせて撮り直すか。既存画像の一部は本文から参照しなくなった
- 「森羅万象」「Shinrabansho」のローマ字表記をリポジトリ間で統一するか
- `画像差し込み候補` の図を自作するか、実機写真を用意するか。第三者画像はライセンスと人物の写り込みを確認する
- compilerとmallocの記事へ、対象source repositoryとtest結果を追記できるか
- カンファレンスの正式名称、開催日、発表・受賞内容を各公式記録と照合する

## 仕様書側で見つけた確認候補

- `spec_v1.adoc` の用語定義でUARTを「同期式シリアル通信」と説明しているが、UARTはasynchronous（非同期）を名称に含む。記事では「非同期シリアル通信」とした。
- 仕様書には開発途中の予約領域や未実装項目がある。記事では実装済みと予約を区別する。
- v1とv2で命令形式が異なるため、今後の記事では対象バージョンを冒頭に明記する。
- `isb` は森羅万象CPU固有の命令であり、他architectureで同名の命令が持つ意味と混同しない。
- SELFは独自形式でELF互換ではない。magicだけでは破損や悪意ある入力を検出できない。

## ロードマップの根拠

- 2023年8月～10月: 加算器、Fetch/Decode、add/addi、フィボナッチ
- 2023年10月～2024年2月: 分岐、Load/Store、UART、in/out、同期メモリ、デモ
- 2024年3月～11月: SPI、GPIO、アセンブラ、JAL、呼び出し規約、タイマー
- 2024年12月～2025年4月: Veryl移植、命令テスト、UART/SPI、映像出力
- 2025年5月～7月: SDカード、命令メモリ書き込み、ブートローダー
- 2025年秋～2026年2月: I-cache、D-cache、AXI風インターコネクト、パイプライン
- 2026年2月: D-cache、AXI風interconnect、5段pipeline、trace比較、malloc、library準備

「AXI風」の正確な準拠範囲はコードレビューが必要なので、公開記事で規格準拠を断定しないこと。

## 今回追加した記事の完成範囲

- 02-01～02-04: 条件分岐、load/store、assembler、ABI
- 03-01～03-04: UART、IOBus/GPIO/timer、SPI、FPGA実機debug
- 04-01～04-08: video、SD、SELF boot、FAT16、compiler、cache、pipeline/interconnect、malloc

記事内の `画像差し込み候補` は、仮の `images/...` pathだけを置いている。画像file自体は作成していない。
