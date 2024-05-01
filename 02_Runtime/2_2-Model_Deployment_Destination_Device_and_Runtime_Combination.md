**[■ 目次](https://github.com/CyberAgentAILab/model-acceleration-tutorial/tree/main?tab=readme-ov-file#table-of-contents)**　**[◀ 前へ](https://github.com/CyberAgentAILab/model-acceleration-tutorial/blob/main/02_Runtime/2_1-Runtime_Options.md)**　**[次へ ▶]()**

## 2-2. モデルのデプロイ先デバイスとランタイムの組み合わせ
下記は `onnxruntime` のチュートリアルページにまとめられたサポート済みバックエンドの表を引用しています。表の列方向に推論に使用する機器 あるいは 適用ドメイン がまとまっています。若干列方向のまとめかたの粒度にばらつきがあり、直接の関連性が無いグルーピングになってはいますが、概ね知りたい切り口で分類されています。ひとくちに CPU / GPU といっても、Intel製、AMD製、ARM、などの様々なメーカーやアーキテクチャが混在している点にご注意ください。また、実際にはPCではなくスマートフォンやクラウド環境での利用を想定しているものまで混在しています。デバイスごとに最適化がなされているとはいえ組み合わせの多さに辟易します。

引用: https://onnxruntime.ai/docs/execution-providers/

![image](https://github.com/CyberAgentAILab/model-acceleration-tutorial/assets/33194443/79f85ed4-7ef8-492d-b215-d686991b6da7)

エンジニアリングの観点で言えば、デバイスごとに適切なフレームワーク あるいは ランタイムを介してモデルを実行する必要があるため、PyTorch / TensorFlow / JAX 単体ではどうにもならないことが多いです。また、それぞれのランタイムには独特のクセや、未実装のオペレータ、対応可能なテンソルの次元数の限界などがあります。

PyTorch や TensorFlow などの大規模フレームワークには一部が取り込まれていることがありますが、全てが横断的にフォローされているわけではないため、個別に最適化やランタイムへ適合させるためのコンバージョンが必要になります。これがモデルを実運用に移行する際の最も大きなボトルネックです。新しいアルゴリズムが組み込まれたモデルであればあるほど既存の標準的なランタイム上でストレートに動作させることは至難の業です。

ただ、幸いなことにあまり悲観しすぎる必要はありません。上図は `onnxruntime` でサポートされているバックエンドの表だと最初に記載しました。したがって、 `onnxruntime` を使用すれば、様々なバックエンドを `onnxruntime` のAPI経由で簡単に呼び出せます。推論時のおまじないのようなプログラムコードの作法が少なく、また、Python, C++、Rust などの主要言語から操作可能である点から、ランタイムのフロントエンドとしての役割がかなり優秀です。つまり、学習時に PyTorch を使用していようが、TensorFlow を使用していようが、JAX を使用していようが、ONNXフォーマットへコンバージョンさえ出来ればあまりバックエンドのことを注意深く意識しなくてもそのまま各種デバイス上で動作させることができます。 OpenXLA のような仕組みが整うまでの間はとても強力な仕組みであることは疑いがありません。　なお、Mac で使用される CoreML に関してはラインナップに記載がありませんが、しっかりサポートされています。

![image](https://github.com/CyberAgentAILab/model-acceleration-tutorial/assets/33194443/0ec165eb-b18e-4889-a563-d68b9debc11e)

**[■ 目次](https://github.com/CyberAgentAILab/model-acceleration-tutorial/tree/main?tab=readme-ov-file#table-of-contents)**　**[◀ 前へ](https://github.com/CyberAgentAILab/model-acceleration-tutorial/blob/main/02_Runtime/2_1-Runtime_Options.md)**　**[次へ ▶]()**
