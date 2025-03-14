**[■ 目次](https://github.com/CyberAgentAILab/model-acceleration-tutorial/tree/main?tab=readme-ov-file#table-of-contents)**　**[◀ 前へ](https://github.com/CyberAgentAILab/model-acceleration-tutorial/blob/main/01_Introduction/1_2-What_to_explain_and_what_not_to_explain.md)**　**[次へ ▶](https://github.com/CyberAgentAILab/model-acceleration-tutorial/blob/main/02_Runtime/2_2-Model_Deployment_Destination_Device_and_Runtime_Combination.md)**

## 2-1. ランタイムの選択肢 (15 min)
PyTorch や TensorFlow などで設計したモデルを学習し、プロダクトやフィールド実験の現場へ学習に使用したフレームワークと学習済みのモデルをそのまま持ち込むことは全ての状況で同じことが言えるとは限りませんが以下の観点で得策ではないときがあります。

1. 学習を実施可能な高機能なフレームワークのインストーラ あるいは バイナリ（実行ファイル）のサイズは数GB以上あるものがほとんどで、環境構築時に大容量の通信や長時間の待ち時間を伴うことが多い
2. 環境をクローンしたり、ストレージ容量が少ない環境下で動作させたい要件が出た場合、ランタイムのサイズの巨大さがとても邪魔になる
3. 特定のハードウェア (以降 HW) 向けのランタイムの最適化が不十分なことがある
4. 一方向、あるいは、数パターンのフレームワーク間コンバージョンにしか対応していないときがある

では、上で挙げた点についてひとつづつ具体例を挙げていきます。

### 2-1-1. インストーラやバイナリのサイズ
フレームワークを実際にインストールしてどれぐらいのリソースを消費するかを検証してみます。ここでは主要フレームワークの下記３種類を取り上げます。いずれも GPU (CUDA) を使用できるパッケージを想定し、できる限り評価の基準を統一しています。ご存知の方もいらっしゃるとは思いますが念の為補足しておきますと、最近は ONNX も学習用の機構を持っており、Hugging Face などのバックエンドとして取り込まれています。

|Framework|Version|Installer|Installer Size|
|:-:|:-|:-|-:|
|PyTorch|2.3.0|[ダウンロード](https://download.pytorch.org/whl/cu121/torch-2.3.0%2Bcu121-cp310-cp310-linux_x86_64.whl)|781.0 MB|
|TensorFlow|2.16.1|[ダウンロード](https://files.pythonhosted.org/packages/a8/dc/e5797d6bf966cd3baa5a6ae32bec31472934c9021ca1505dc7bf5c8fc902/tensorflow-2.16.1-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl)|589.8 MB|
|ONNX (onnxruntime-gpu)|1.17.3|[ダウンロード](https://files.pythonhosted.org/packages/24/18/86d24a10e624ad1b94294230b4efc2bcd243b83b169cc5d4e3f5ace5de25/onnxruntime_gpu-1.17.1-cp310-cp310-manylinux_2_28_x86_64.whl)|192.1 MB|

いずれも 2024年04月25日時点 での最新バージョンです。PyTorchとTensorFlowは２世代以上古いバージョンのインストーラは 2GB を大幅に超えていましたが、最新のインストーラはどちらも大幅に軽量化されているようです。これはあくまで本体部分のインストーラのサイズであり、依存パッケージ部分を含まないサイズであることにご注意ください。では、実際にクリーンな環境へインストールしてどれほどストレージを消費するかを実際に検証してみます。あまり手間をかけず環境を汚さずに簡易的に計測したいため、Docker イメージをビルドしてイメージサイズを比較します。なお通常はフレームワークの本体のみをインストールして利用することは少なく、また、パッケージの依存関係に応じて自動的に周辺パッケージを追加インストールして利用することのほうが多いと思いますので、最もシンプルな公式のチュートリアルどおりの導入手順にしたがって環境構築する前提とします。

研究開発のフェーズにおいて、フレームワークそのものをセルフビルドしてリソースを切り詰めることは現実的ではありません。したがって、設計時点では扱いやすいフレームワークを選択いただき、運用のフェーズに移り変わるタイミングでできる限りスムーズに移行するために、扱いやすいフレームワーク上であらかじめ意識しておいていただきたいことを後続でご説明します。

```bash
docker pull ubuntu:22.04
```

#### 1. PyTorch
[Dockerfile.pytorch](./Dockerfile.pytorch)

```Dockerfile
FROM ubuntu:22.04
ENV DEBIAN_FRONTEND=noninteractive
SHELL ["/bin/bash", "-c"]
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
では、フレームワーク本体以外の依存関係部分でどれほど多くのパッケージが追加インストールされたかを見てみます。
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
フレームワーク本体以外の部分だけで `4.5 GB` もの依存パッケージが勝手にインストールされることが分かりました。なお、 `--no-deps` オプションを指定して `pip install` すれば本体のみをインストールすることが可能なためストレージ消費をもっと低く抑えることは可能ですが、依存するパッケージをインストールしないため、フレームワークが実質まともに動作しません。

#### 2. TensorFlow
[Dockerfile.tensorflow](./Dockerfile.tensorflow)

```Dockerfile
FROM ubuntu:22.04
ENV DEBIAN_FRONTEND=noninteractive
SHELL ["/bin/bash", "-c"]
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        python3-pip \
    && pip install \
        tensorflow \
    && apt clean \
    && rm -rf /var/lib/apt/lists/* \
    && rm /etc/apt/apt.conf.d/docker-clean \
    && pip cache purge
```
```bash
docker build -t tensorflow -f Dockerfile.tensorflow .
docker images
```
同じく、Docker イメージのサイズが Wheel ファイルのサイズを大幅に超過しました。
```bash
ubuntu 22.04 7af9ba4f0a47 2 weeks ago 77.9MB
tensorflow latest 787922c5739a 18 seconds ago 2.09GB
```
依存関係を見てみます。パッケージの数自体はかなり多いですが、ストレージの消費量は PyTorch の半分ほどでした。
```bash
docker run tensorflow pip list

Package                      Version
---------------------------- --------
absl-py                      2.1.0
astunparse                   1.6.3
certifi                      2024.2.2
charset-normalizer           3.3.2
flatbuffers                  24.3.25
gast                         0.5.4
google-pasta                 0.2.0
grpcio                       1.62.2
h5py                         3.11.0
idna                         3.7
keras                        3.3.2
libclang                     18.1.1
Markdown                     3.6
markdown-it-py               3.0.0
MarkupSafe                   2.1.5
mdurl                        0.1.2
ml-dtypes                    0.3.2
namex                        0.0.8
numpy                        1.26.4
opt-einsum                   3.3.0
optree                       0.11.0
packaging                    24.0
pip                          22.0.2
protobuf                     4.25.3
Pygments                     2.17.2
requests                     2.31.0
rich                         13.7.1
setuptools                   59.6.0
six                          1.16.0
tensorboard                  2.16.2
tensorboard-data-server      0.7.2
tensorflow                   2.16.1
tensorflow-io-gcs-filesystem 0.36.0
termcolor                    2.4.0
typing_extensions            4.11.0
urllib3                      2.2.1
Werkzeug                     3.0.2
wheel                        0.37.1
wrapt                        1.16.0
```

#### 3. onnxruntime-gpu
[Dockerfile.onnx](./Dockerfile.onnx)

```Dockerfile
FROM ubuntu:22.04
ENV DEBIAN_FRONTEND=noninteractive
SHELL ["/bin/bash", "-c"]
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        python3-pip \
    && pip install \
        onnxruntime-gpu \
    && apt clean \
    && rm -rf /var/lib/apt/lists/* \
    && rm /etc/apt/apt.conf.d/docker-clean \
    && pip cache purge
```
```bash
docker build -t onnx -f Dockerfile.onnx .
docker images
```
同じく、Docker イメージのサイズが Wheel ファイルのサイズを大幅に超過しました。
```bash
ubuntu 22.04 7af9ba4f0a47 2 weeks ago 77.9MB
onnx latest 5985dc8c78bc About a minute ago 816MB
```
依存関係を見てみます。パッケージの数自体はかなり少なく、ストレージの消費量は 1GB を下回りました。
```bash
docker run onnx pip list

Package         Version
--------------- -------
coloredlogs     15.0.1
flatbuffers     24.3.25
humanfriendly   10.0
mpmath          1.3.0
numpy           1.26.4
onnxruntime-gpu 1.17.1
packaging       24.0
pip             22.0.2
protobuf        5.26.1
setuptools      59.6.0
sympy           1.12
wheel           0.37.1
```

以上の結果から、導入の容易さ、という観点ではどのフレームワークでも大差ありませんが、扱いやすさやモデル設計のしやすさ、トレーニング後のチューニングのし易さという観点ではストレージ消費量、すなわち依存パッケージの量や種類の豊富さとトレードオフの関係にあるように見えます。ただしもう一度補足しますが、これはあくまでチュートリアルどおりに公式のインストール手順に従った場合の結果であり、自力で必要な機能のみをビルドする場合はこの限りではありません。もちろん PyTorch であってもCUDAが不要であればセルフビルドを行うことで `1 GB` を下回る軽量なインストーラを生成することも可能です。

遠回りになりましたがここで述べたいのは、チュートリアルどおりのインストールシーケンスどおりに環境構築を行うことは、不必要にパッケージダウンロード時のネットワーク帯域を圧迫することや、みなさんの待ち時間が伸びること、CIを使用したテストの時間が伸びること、それに伴って外部ベンダーへの課金額が増えること、場合によってはネットワークのレートリミットに達して作業を継続できないなど、様々な問題を抱えることがあります。この話は単純に「リソースを節約するために軽量なランタイムを選択しましょう。」という話をしたいわけではなく、後続の章でご説明する運用を見越した様々な最適化の話に繋がります。

### 2-1-2. ストレージサイズや恣意的なレートリミット
大手のクラウドベンダはそのコストの高さもさることながら、とても厳しいレートリミットを設定していることがあります。特にネットワーク帯域周りではリミットに到達することが多いと感じます。また、クラウドストレージやネットワークの従量課金はとても高コストですので、できる限りリソースをクラウドに配置したくありません。開発や運用の効率性の観点からクラウドが最適な選択のひとつになることが多いことは疑いませんが、組織単位でコストを積み上げると驚くようなコストになっていることがよくあります。レートリミットの観点で具体例を挙げると、GitHub Conatiner Registory (GHCR) はバックエンドに Azure を使用していますが、有償・無償に関わらずCIでGHCRと連携したジョブを組んだときに、 サイズが 10 GB 前後の Docker イメージを Pull すると、あきらかに帯域を制限されて 数十KB/秒 ほどの下り速度しかでなくなり、ジョブが半日終わらない、というようなことが容易に起こります。なお、気にしなければならないのはなにもクラウドに関することだけではなく、Box PC やスマートフォンなど、最終デプロイ先のデバイスのリソース制約に応じて、まず最初に矢面に立つのがランタイム（バイナリ）のサイズです。設計したモデルは作り込み次第では柔軟に入れ替えることが可能ですが、ランタイムはよほどのことが無い限り入れ替えることにはならないので、仮に特定のフレームワークを選択したとしても、できる限りフレームワーク間の転用がしやすくなることを念頭に置きつつ、最終的なリソース消費が少なくなるようにも考慮しておくことは重要です。

### 2-1-3. ハードウェアへの最適化
HWとひとくちに言っても様々有ります。
1. CPU (x64, ARM32, ARM64, RISC-V, etc...)
2. GPU (NVIDIA RTXxxxx, AMD Radeon, Intel Arc, etc...)
3. AI Accelerator (TPU, Myriad, Hailo-8, MN-Core, etc...)

昔から議論されて尽くされていますので、ここではコンピュータ・サイエンスの細かな部分には触れませんが、私が目にするアクセラレータはNHWC形式のテンソルを期待しているものが今のところは多いです。気になる方はこちらのブログ記事がわかりやすくまとまっているようですのでそちらをご覧ください。

[NHWC vs NCHW : A memory access perspective on GPUs](https://medium.com/@deepika_writes/nhwc-vs-nchw-a-memory-access-perspective-on-gpus-4e79bd3b1b54)

![image](https://github.com/CyberAgentAILab/model-acceleration-tutorial/assets/33194443/e246304b-f2a8-4df0-82a3-fe6e703f034e)

また、下記は記事内で引用されているNVIDIAさんの公開資料です。様々な観点でHWによる処理パフォーマンスが分析されています。

[Convolutional Layers User's Guide](https://docs.nvidia.com/deeplearning/performance/dl-performance-convolutional/index.html#imp-gemm-dim)

`PyTorch` で採用されている `NCHW` 形式より、リサーチ分野ではほとんど目にすることが無くなった `TensorFlow` で採用されている `NHWC` 形式の処理パフォーマンスが優れていることを示す図を１箇所だけ引用しておきます。

![image](https://github.com/CyberAgentAILab/model-acceleration-tutorial/assets/33194443/b0686eac-ac6b-40e0-9d67-737bb7a33ec6)

記事や資料を読んで頂くと分かるとおり、モデル設計時や学習時のパフォーマンスはともかくとして、推論時のパフォーマンスにおいては `NHWC` が少し有利のように見えます。ただ、最近は PyTorch においてもエッジデバイス向けにランタイムの最適化を始めているようですので状況の注視が必要です。

- ExecuTorch

    ExecuTorch is an end-to-end solution for enabling on-device inference capabilities across mobile and edge devices including wearables, embedded devices and microcontrollers. It is part of the PyTorch Edge ecosystem and enables efficient deployment of PyTorch models to edge devices.

    https://github.com/pytorch/executorch

2024年04月時点で上記は絶賛開発中のものですが、あと数年後にはもしかしたら状況が大きく変わるかもしれませんが、現時点において `NHWC` 形式を前提としたアクセラレータが比較的多いという前提に立った場合、 `NHWC` を前提とした ランタイム あるいは フレームワーク も無視することができません。


### 2-1-4. フレームワーク間コンバージョン
前節までの内容をまとめると、 `NCHW` 形式を前提として扱いやすい `PyTorch` でモデルを設計し、 `NHWC` 形式でデバイスにデプロイする、という方法が理想的なフローのように見えます。モデル設計や学習のし易さ、論文実装の膨大さの観点から `PyTorch` はとても良い選択肢です。 `TensorFlow` に `PyTorch` のようなしなやかさが備わっていれば、 学習から end-to-end でデバイスに最適化できる `TensorFlow` は不動の地位を築いていたかもしれません。残念ながら現時点では、研究開発でモデルを設計する多くのリサーチャーとモデルを実際にデプロイするエンジニアの間には大きな壁が存在しています。

今のところ、データセンターのように潤沢な計算リソースと大容量の電源を確保できる環境は限られており、汎用的に多くの環境へデプロイするには `PyTorch` 単体では困難を伴うことがあります。最初にご説明したランタイムのサイズの観点やHWへの最適化の観点など、さまざまな要因が存在します。したがって、特定のフレームワークならびにランタイムに依存し過ぎないように、設計したモデルをフレームワーク間でコンバージョンすることもひとつの選択肢に挙がりますが、スムーズにコンバージョンできる状況はあまり多くありません。その理由は、実はモデルの設計時点でコンバージョンの難易度を無意識に上げてしまっていることが要因のひとつとして挙げられます。特定のHWとフレームワークが厳密に紐付いてしまい、開発要件や環境によって使用するフレームワークが異なる状況が多いためとても手間が多いです。

なお、こういった状況を打開する取り組みがあるためココで少しだけ触れておきます。 `StableHLO` というプロジェクトで、これは様々なMLフレームワーク（TensorFlow、JAX、PyTorchなど）とMLコンパイラ（XLAやIREEなど）の相互運用性を高めることで、ML開発を簡素化し、加速することを目的としています。 また、`StableHLO` や `XLA (Accelerated Linear Algebra)` を取り込んだ `OpenXLA` というものが誕生しており、ありとあらゆる有力デベロッパーが互いに協力して、モデルのデプロイに関する障壁を取り払おうとしてくれています。2023年時点での参画企業は Alibaba, Amazon Web Services, AMD, Anyscale, Apple, Arm, Cerebras, Google, Graphcore, Hugging Face, Intel, Meta, NVIDIA, SiFive などです。

- OpenXLA - Google Open Source Blog

    https://opensource.googleblog.com/2023/03/openxla-is-ready-to-accelerate-and-simplify-ml-development.html

- StableHLO - チュートリアル

    English： https://openxla.org/stablehlo

    Japanese： https://openxla.org/stablehlo?hl=ja

- StableHLO - GitHub

    https://github.com/openxla/stablehlo

- XLA (Accelerated Linear Algebra) - チュートリアル

    English: https://openxla.org/xla

    Japanese: https://www.tensorflow.org/xla?hl=ja

`PyTorch` や `TensorFlow` でこの仕組みを活用するためのコミットが少しづつ追加されていることを観測しています。以下に `JAX` と `PyTorch` と `TensorFlow` での簡易的なサンプルが記載されているURLをご紹介します。

- JAX (Hugging Face, TensorFlow):
  - [Tutorial: Exporting StableHLO from JAX](https://openxla.org/stablehlo/tutorials/jax-export)
- PyTorch:
  - [Tutorial: Exporting StableHLO from PyTorch](https://openxla.org/stablehlo/tutorials/pytorch-export)
- TensorFlow:
  - [Tutorial: Embedding StableHLO in SavedModel](https://openxla.org/stablehlo/tutorials/savedmodel-embed)
  - [XLA: Optimizing Compiler for Machine Learning](https://openxla.org/xla/tf2xla)

モデルの共通的な記述仕様へ統一されていき、モデルの開発から最終デプロイまでのワークフローがほぼ同じになることが期待されます。今のところはチュートリアルがとても簡易的で、デバイスへのデプロイまでの統一的なフローが実現されているようにはあまり見えない状況ではありますが、参画デベロッパーの全てがこの仕組みを取り入れることで、将来的には学習用フレームワークや最終デプロイ先のデバイスの種類をあまり意識する必要が無くなるかもしれません。

この仕組みが早く世界中に浸透することを願ってやみませんが、現時点では参考例や製品で標準的に使用されている事例などの情報が極めて少ないため、最終デプロイ先のデバイス向けにモデルをコンバージョンする前提で、どれぐらいのデバイスの種類があるか、それぞれの特性に応じたデプロイ方法の組み合わせ、現時点で取りうる対策について次の節以降で触れます。

**[■ 目次](https://github.com/CyberAgentAILab/model-acceleration-tutorial/tree/main?tab=readme-ov-file#table-of-contents)**　**[◀ 前へ](https://github.com/CyberAgentAILab/model-acceleration-tutorial/blob/main/01_Introduction/1_2-What_to_explain_and_what_not_to_explain.md)**　**[次へ ▶](https://github.com/CyberAgentAILab/model-acceleration-tutorial/blob/main/02_Runtime/2_2-Model_Deployment_Destination_Device_and_Runtime_Combination.md)**
