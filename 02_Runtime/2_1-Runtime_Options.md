**[■ 目次](https://github.com/CyberAgentAILab/model-acceleration-tutorial/tree/main?tab=readme-ov-file#table-of-contents)**　**[◀ 前へ](https://github.com/CyberAgentAILab/model-acceleration-tutorial/blob/main/01_Introduction/1_2-What_to_explain_and_what_not_to_explain.md)**　**[次へ ▶]()**

## 2-1. ランタイムの選択肢
PyTorch や TensorFlow などで設計したモデルを学習し、プロダクトやフィールド実験の現場へ学習に使用したフレームワークと学習済みのモデルをそのまま持ち込むことは全ての状況で同じことが言えるとは限りませんが以下の観点で得策ではないときがあります。

1. 学習を実施可能な高機能なフレームワークのインストーラ あるいは バイナリ（実行ファイル）のサイズは数GB以上あるものがほとんどで、環境構築時に大容量の通信や長時間の待ち時間を伴うことが多い
2. 環境をクローンしたり、ストレージ容量が少ない環境下で動作させたい要件が出た場合、ランタイムのサイズの巨大さがとても邪魔になる
3. 設計したモデル構造をソースコードごとまとめてデプロイする必要があることが多く、リソース管理上の無駄が多い
4. 特定のハードウェア (以降 HW) 向けのランタイムの最適化が不十分なことがある
5. 一方向、あるいは、数パターンのフレームワーク間コンバージョンにしか対応していないときがある



引用: https://onnxruntime.ai/docs/execution-providers/

**[■ 目次](https://github.com/CyberAgentAILab/model-acceleration-tutorial/tree/main?tab=readme-ov-file#table-of-contents)**　**[◀ 前へ](https://github.com/CyberAgentAILab/model-acceleration-tutorial/blob/main/01_Introduction/1_2-What_to_explain_and_what_not_to_explain.md)**　**[次へ ▶]()**
