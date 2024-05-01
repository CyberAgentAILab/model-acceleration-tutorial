**[■ 目次](https://github.com/CyberAgentAILab/model-acceleration-tutorial/tree/main?tab=readme-ov-file#table-of-contents)**　**[◀ 前へ](https://github.com/CyberAgentAILab/model-acceleration-tutorial/blob/main/02_Runtime/2_1-Runtime_Options.md)**　**[次へ ▶]()**

## 2-2. モデルのデプロイ先デバイスとランタイムの組み合わせ
下記は `onnxruntime` のチュートリアルページにまとめられた表を引用しています。表の列方向に推論に使用する機器 あるいは 適用ドメイン がまとまっています。若干列方向のまとめかたの粒度にばらつきがあり、直接の関連性が無いグルーピングになってはいますが、概ね知りたい切り口で分類されています。ひとくちに CPU / GPU といっても、Intel製、AMD製、ARM、などの様々なメーカーやアーキテクチャが混在している点にご注意ください。また、実際にはPCではなくスマートフォンやクラウド環境での利用を想定しているものまで混在しています。デバイスごとに最適化がなされているとはいえ組み合わせの多さに辟易します。

引用: https://onnxruntime.ai/docs/execution-providers/

![image](https://github.com/CyberAgentAILab/model-acceleration-tutorial/assets/33194443/79f85ed4-7ef8-492d-b215-d686991b6da7)

エンジニアリングの観点で言えば、デバイスごとに適切なフレームワーク あるいは ランタイムを介してモデルを実行する必要があるため、PyTorch / TensorFlow / JAX 単体ではどうにもならないことが多いです。

**[■ 目次](https://github.com/CyberAgentAILab/model-acceleration-tutorial/tree/main?tab=readme-ov-file#table-of-contents)**　**[◀ 前へ](https://github.com/CyberAgentAILab/model-acceleration-tutorial/blob/main/02_Runtime/2_1-Runtime_Options.md)**　**[次へ ▶]()**
