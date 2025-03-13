**[■ 目次](https://github.com/CyberAgentAILab/model-acceleration-tutorial/tree/main?tab=readme-ov-file#table-of-contents)**　**[◀ 前へ](https://github.com/CyberAgentAILab/model-acceleration-tutorial/blob/main/02_Runtime/2_2-Model_Deployment_Destination_Device_and_Runtime_Combination.md)**　**[次へ ▶](https://github.com/CyberAgentAILab/model-acceleration-tutorial/blob/main/03_Design/3_2_1-Preparing_the_environment_for_hands-on.md)**

## 3-1. CPU・GPU・TensorRT、他HW・FW推論を意識した設計 (1 min)
学習可能なフレームワークで設計したモデルをできる限り汎用的に多くのランタイムへ適合させることを考える場合、多くの細かな点を意識しておく必要が有ります。ココでは大まかなポイントを記載し、次の節以降で実際に手を動かしながら具体的にどのようなことを述べているのかを理解いただきながら進めていきます。

まず、大まかに意識しておいたほうがよいポイントを下記に列記します。次節のハンズオン内で解説しないポイントも一部含みます。

1. 必要以上に次元の展開と圧縮を使用しない
2. 連続するオペレーションの順序を入れ替えてできる限り不必要なオペレーションを減らす
3. 定数で表現可能な全てのオペレーションは事前に定数同士で演算しておいて定数そのもの、あるいは単純な四則演算の数を減らす
4. 乗算に置き換え可能な演算はできる限り乗算に置き換えておく
5. 言語仕様による数値計算上の端数の差異が生じる演算は極力割ける (小数部の処理、切り上げ、切り下げ、四捨五入、etc...)
6. 真偽値と数値の間のキャスト (1->True, True->1, 0->False, False->0)
7. ５次元あるいはそれ以上のオペレーションをできる限り減らす
8. 形状不定の次元をバッチサイズ以外の次元で複数持たない
9. できる限りプリミティブなオペレーションを組み合わせる

これらは、モデル展開先のフレームワークの内部実装バグの影響を受けやすいポイントであったり、HWの特性上処理できない構造になったり、モデルの構造上の冗長さをむやみに増やしたり、モデルの推論・演算パフォーマンスを大きく悪化させる事項であり、ちょっとしたことではあるもののモデルの汎用性を著しく低下させる要素です。

次の節以降は、上記をハンズオン形式で実際にコードを書いて体験しながら説明していきます。

**[■ 目次](https://github.com/CyberAgentAILab/model-acceleration-tutorial/tree/main?tab=readme-ov-file#table-of-contents)**　**[◀ 前へ](https://github.com/CyberAgentAILab/model-acceleration-tutorial/blob/main/02_Runtime/2_2-Model_Deployment_Destination_Device_and_Runtime_Combination.md)**　**[次へ ▶](https://github.com/CyberAgentAILab/model-acceleration-tutorial/blob/main/03_Design/3_2_1-Preparing_the_environment_for_hands-on.md)**
