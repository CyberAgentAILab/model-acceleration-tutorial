**[■ 目次](https://github.com/CyberAgentAILab/model-acceleration-tutorial/tree/main?tab=readme-ov-file#table-of-contents)**　**[◀ 前へ](https://github.com/CyberAgentAILab/model-acceleration-tutorial/blob/main/01_Introduction/1_2-What_to_explain_and_what_not_to_explain.md)**　**[次へ ▶](https://github.com/CyberAgentAILab/model-acceleration-tutorial/blob/main/02_Runtime/2_2-Model_Deployment_Destination_Device_and_Runtime_Combination.md)**

## 2-1. ランタイムの選択肢
PyTorch や TensorFlow などで設計したモデルを学習し、プロダクトやフィールド実験の現場へ学習に使用したフレームワークと学習済みのモデルをそのまま持ち込むことは全ての状況で同じことが言えるとは限りませんが以下の観点で得策ではないときがあります。

1. 学習を実施可能な高機能なフレームワークのインストーラ あるいは バイナリ（実行ファイル）のサイズは数GB以上あるものがほとんどで、環境構築時に大容量の通信や長時間の待ち時間を伴うことが多い
2. 環境をクローンしたり、ストレージ容量が少ない環境下で動作させたい要件が出た場合、ランタイムのサイズの巨大さがとても邪魔になる
3. 設計したモデル構造をソースコードごとまとめてデプロイする必要があることが多く、リソース管理上の無駄が多い
4. 特定のハードウェア (以降 HW) 向けのランタイムの最適化が不十分なことがある
5. 一方向、あるいは、数パターンのフレームワーク間コンバージョンにしか対応していないときがある

では、上で挙げた点についてひとつづつ具体例を挙げていきます。

### 2-1-1. インストーラやバイナリのサイズ
フレームワークを実際にインストールしてどれぐらいのリソースを消費するかを検証してみます。ここでは主要フレームワークの下記３種類を取り上げます。

|Framework|Version|Installer|Installer Size|
|:-:|:-|:-|-:|
|PyTorch|2.3.0|[ダウンロード](https://download.pytorch.org/whl/cu121/torch-2.3.0%2Bcu121-cp310-cp310-linux_x86_64.whl)|781.0 MB|
|TensorFlow|2.16.1|[ダウンロード](https://files.pythonhosted.org/packages/a8/dc/e5797d6bf966cd3baa5a6ae32bec31472934c9021ca1505dc7bf5c8fc902/tensorflow-2.16.1-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl)|589.8 MB|
|ONNX (onnxruntime-gpu)|1.17.3|[ダウンロード](https://files.pythonhosted.org/packages/24/18/86d24a10e624ad1b94294230b4efc2bcd243b83b169cc5d4e3f5ace5de25/onnxruntime_gpu-1.17.1-cp310-cp310-manylinux_2_28_x86_64.whl)|192.1 MB|

いずれも 2024年04月25日時点 での最新バージョンです。PyTorchとTensorFlowは２世代以上古いバージョンのインストーラは 2GB を大幅に超えていましたが、最新のインストーラはどちらも大幅に軽量化されているようです。これはあくまで本体部分のインストーラのサイズであり、依存パッケージ部分を含まないサイズであることにご注意ください。では、実際にクリーンな環境へインストールしてどれほどストレージを消費するかを実際に検証してみます。

### 2-1-2. ストレージサイズ

### 2-1-3. モデルとソースコードの管理

### 2-1-4. ハードウェアへの最適化

### 2-1-5. フレームワーク間コンバージョン


**[■ 目次](https://github.com/CyberAgentAILab/model-acceleration-tutorial/tree/main?tab=readme-ov-file#table-of-contents)**　**[◀ 前へ](https://github.com/CyberAgentAILab/model-acceleration-tutorial/blob/main/01_Introduction/1_2-What_to_explain_and_what_not_to_explain.md)**　**[次へ ▶](https://github.com/CyberAgentAILab/model-acceleration-tutorial/blob/main/02_Runtime/2_2-Model_Deployment_Destination_Device_and_Runtime_Combination.md)**
