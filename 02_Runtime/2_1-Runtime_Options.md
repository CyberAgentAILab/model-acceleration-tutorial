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
フレームワークを実際にインストールしてどれぐらいのリソースを消費するかを検証してみます。ここでは主要フレームワークの下記３種類を取り上げます。いずれも GPU (CUDA) を使用できるパッケージを想定し、できる限り評価の基準を統一しています。ご存知の方もいらっしゃるとは思いますが念の為補足しておきますと、最近は ONNX も学習用の機構を持っており、Hugging Face などのバックエンドとして取り込まれています。

|Framework|Version|Installer|Installer Size|
|:-:|:-|:-|-:|
|PyTorch|2.3.0|[ダウンロード](https://download.pytorch.org/whl/cu121/torch-2.3.0%2Bcu121-cp310-cp310-linux_x86_64.whl)|781.0 MB|
|TensorFlow|2.16.1|[ダウンロード](https://files.pythonhosted.org/packages/a8/dc/e5797d6bf966cd3baa5a6ae32bec31472934c9021ca1505dc7bf5c8fc902/tensorflow-2.16.1-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl)|589.8 MB|
|ONNX (onnxruntime-gpu)|1.17.3|[ダウンロード](https://files.pythonhosted.org/packages/24/18/86d24a10e624ad1b94294230b4efc2bcd243b83b169cc5d4e3f5ace5de25/onnxruntime_gpu-1.17.1-cp310-cp310-manylinux_2_28_x86_64.whl)|192.1 MB|

いずれも 2024年04月25日時点 での最新バージョンです。PyTorchとTensorFlowは２世代以上古いバージョンのインストーラは 2GB を大幅に超えていましたが、最新のインストーラはどちらも大幅に軽量化されているようです。これはあくまで本体部分のインストーラのサイズであり、依存パッケージ部分を含まないサイズであることにご注意ください。では、実際にクリーンな環境へインストールしてどれほどストレージを消費するかを実際に検証してみます。あまり手間をかけず環境を汚さずに簡易的に計測したいため、Docker イメージをビルドしてイメージサイズを比較します。なお通常はフレームワークの本体のみをインストールして利用することは少なく、また、パッケージの依存関係に応じて自動的に周辺パッケージを追加インストールして利用することのほうが多いと思いますので、最もシンプルな公式のチュートリアルどおりの導入手順にしたがって環境構築する前提とします。

```bash
docker pull ubuntu:22.04
```

#### 1. PyTorch
```Dockerfile
FROM ubuntu:22.04
ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        python3-pip \
    && pip install \
        torch \
        torchvision \
        torchaudio \
    && apt clean \
    && rm -rf /var/lib/apt/lists/* \
    && rm /etc/apt/apt.conf.d/docker-clean \
    && pip cache purge
```
```bash
docker build -t torch -f Dockerfile.pytorch .
docker images
```
Docker イメージのサイズが Wheel ファイルのサイズを大幅に超過しました。
```bash
ubuntu 22.04 7af9ba4f0a47 2 weeks ago 77.9MB
torch latest a8683c508332 48 seconds ago 5.33GB
```
では、フレームワーク本体以外の部分がどれぐらい多く依存関係としてインストールされたかを見てみます。
```bash
docker run torch pip list

Package                  Version
------------------------ ----------
filelock                 3.13.4
fsspec                   2024.3.1
Jinja2                   3.1.3
MarkupSafe               2.1.5
mpmath                   1.3.0
networkx                 3.3
numpy                    1.26.4
nvidia-cublas-cu12       12.1.3.1
nvidia-cuda-cupti-cu12   12.1.105
nvidia-cuda-nvrtc-cu12   12.1.105
nvidia-cuda-runtime-cu12 12.1.105
nvidia-cudnn-cu12        8.9.2.26
nvidia-cufft-cu12        11.0.2.54
nvidia-curand-cu12       10.3.2.106
nvidia-cusolver-cu12     11.4.5.107
nvidia-cusparse-cu12     12.1.0.106
nvidia-nccl-cu12         2.20.5
nvidia-nvjitlink-cu12    12.4.127
nvidia-nvtx-cu12         12.1.105
pillow                   10.3.0
pip                      22.0.2
setuptools               59.6.0
sympy                    1.12
torch                    2.3.0
torchaudio               2.3.0
torchvision              0.18.0
triton                   2.3.0
typing_extensions        4.11.0
wheel                    0.37.1
```
フレームワーク本体以外の部分だけで 4.5 GB もの依存パッケージが勝手にインストールされることが分かりました。なお、 `--no-deps` オプションを指定して `pip install` すれば本体のみをインストールすることが可能なためストレージ消費をもっと低く抑えることは可能ですが、依存するパッケージをインストールしないため、フレームワークが実質まともに動作しません。

#### 2. TensorFlow

#### 3. onnxruntime-gpu

### 2-1-2. ストレージサイズ

### 2-1-3. モデルとソースコードの管理

### 2-1-4. ハードウェアへの最適化

### 2-1-5. フレームワーク間コンバージョン


**[■ 目次](https://github.com/CyberAgentAILab/model-acceleration-tutorial/tree/main?tab=readme-ov-file#table-of-contents)**　**[◀ 前へ](https://github.com/CyberAgentAILab/model-acceleration-tutorial/blob/main/01_Introduction/1_2-What_to_explain_and_what_not_to_explain.md)**　**[次へ ▶](https://github.com/CyberAgentAILab/model-acceleration-tutorial/blob/main/02_Runtime/2_2-Model_Deployment_Destination_Device_and_Runtime_Combination.md)**
