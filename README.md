# kaggle-H&M
H&M コンペのリポジトリ

- directory tree
```
Kaggle-H&M
├── README.md
├── data         <---- gitで管理するデータ
├── data_ignore  <---- .gitignoreに記述されているディレクトリ(モデルとか、特徴量とか、データセットとか)
├── nb           <---- jupyter lab で作業したノートブック
├── nb_download  <---- ダウンロードした公開されているkagglenb
└── src          <---- .ipynb 以外のコード

```

## Pipeline
- 実行例
  ```bash
  python3 pipeline.py --globals.balanced=1 --globals.comment=test
  ```

- 結果の表示例
  ```bash
  python3 show_result.py -d 0
  ```

## Info
- [issue board](https://github.com/fkubota/kaggle-Cornell-Birdcall-Identification/projects/1)   <---- これ大事だよ
- [google slide](https://docs.google.com/presentation/d/1ZcCSnXj2QoOmuIkcA-txJOuAlkLv4rSlS7_zDj90q6c/edit#slide=id.p)
- [flow chart](https://app.diagrams.net/#G1699QH9hrlRznMikAEAE2-3WTjreYcWck)
- [google drive](https://drive.google.com/drive/u/1/folders/1UDVIKTN1O7hTL9JYzt7ui3mNy_b6RHCV)
- ref:
  - [metricについて](https://www.kaggle.com/shonenkov/competition-metrics)
- docker run 時にいれるオプション
  - `--shm-size=5G`

## Timeline

## Dataset
|Name|Detail|Ref|
|---|---|---|
|images|商品の画像についてのデータ|データ|
|article.csv|商品関連のデータ|-|
|customers.csv|顧客情報のデータ|-|
|transactions_train.csv|顧客の商品購入履歴に関するデータ|-|
|sample_submission.csv|提出用のデータ。顧客毎に12のアイテムを記載して提出する|-|

## データ分析ノート
|テーブル|変数名|内容|データタイプ|特徴|変数アイディア|
|----|----|----|----|----|----|
|rating|録音の質を表す(A,B,C,D,Eの5段階)|


## Features
|Name|shape (feat only)|size(MB)|Detail|
|---|---|---|---|
|nb004_librosa_mfcc.csv|(21,375, 11)|2.0|librosaのmfcc(2~12)。audiofile1つにつき1ベクトル。srを揃えてないので周波数空間の大きさに差が有り問題がありそう。srを16kHzとかにそろえたほうがいいと思う。|
|nb007_librosa_mfcc02.csv|(4,779,859, 11)|436.1|nb004の特徴量の拡張。audiofile内のn_feat/m_audio/1_bird。nb004の特徴量よりかなりデータ数が多い。|
|nb008_librosa_basic|(4,779,859, 12)|482.7|['rms', 'centroid', 'sc_1', 'sc_2', 'sc_3', 'sc_4', 'sc_5', 'sc_6', 'sb', 'sf', 'sr', 'zcr']。nb004と同じくsrを揃えていない問題がある。|
|nb010_librosa_rms.csv|(4779859, 3)|144|event部分だけ抽出する際のthresholdとして使う。|

## Paper
|No.|Status|Name|Detail|Date|Url|
|---|---|---|---|---|---|


## Log
### 20220219
- スタート
- 分析のアプローチ案
　-ユニークなarticleのうち一部の商品しか買われていないのではないか？
　　-まずは期間中飼われている商品の可視化
　　-上記仮説が正しければ、商品を絞り込んだ上で、ユーザー✖️商品（絞り込んだもの）のリストを作成しその中で予測を行う
  
### 20220306
- 前年の直前の7日間と直後の七日間の購入商品の分布はほぼ同じ
　-　今年度においても直前の7日間の人気と直後の7日間の購入される商品の分布は同じなのではないか
　-　articleの数をカウントして、TopNの商品をレコメンドの母数にするのがありなのでは？

### 20220307
- 20年9月16日から22日のデータを抽出してC_periodと呼ぶ
  - 期間中のTOP Nのarticle_IDの商品をレコメンド対象と設定し、その中で推薦を行う 
    - Nを決めるためにパレート図を作成しTOP何のarticle_IDで8割を占めているかを確認する（next） 
    - 

#eck と呼ばれてる

- nb004
  - 初めての特徴量を作成した。
  - librosaのmfcc(2~12)。
  - m_feat/m_audiofile/1_ebird
  - 1つのwavからはフレームごとにmfccを得られるがそれを平均化した。

- nb005
  - スペクトログラムを画像で保存するためのコードを書いた。
  - 画像を27000枚ほど作らなければいけなかったのでdpiを小さくした。
  - メモリリークが微妙に防げない...

### 20200802
- nb006
  - nb004 で作成した特徴量を使ってrfcモデルを作成する。
  - モデルを5つ、infoを1つ保存した。
    - info
      - featsets (今回は、nb004_librosa_mfcc.csv)
      - feat_names (↑のfeatsetsから何かを除いたりすることもあると思うので)
    - models (モデルがfold分格納されている)
      - size: 37x5MB

- nb007
  - nb004で作成した特徴量の拡張版
  - n_feat/m_wav/1_bird にした。
  - window_sizeとstrideは0.5, 0.25 sec
  - nb004の特徴量よりデータ数がかなり多い。
    - shape: (4,779,859, 11)
    - time: 2:51:06

### 20200803
- kagglenb_02_nocall_only
  - nocall だけでsubmitしてみた
  - result
    - cv: none
    - sub: 0.544

- このSample Submission File(公式) 見る感じ、複数の鳥が鳴いていればそれを予測するということか。
  - ![duration](./data/info/images/readme/011.png)

- 今後使いそうなNNの初手
  - https://twitter.com/mlaass1/status/1290131798735781890/retweets/with_comments

- nb008
  - librosaの基本的な特徴量を実装。
  - n_feat/m_wav/1_bird
  - w_size=0.5, w_stride=0.25 sec
  - feats
    - ['librosa_rms', 'librosa_centroid', 'librosa_sc_1', 'librosa_sc_2', 'librosa_sc_3', 'librosa_sc_4', 'librosa_sc_5', 'librosa_sc_6', 'librosa_sb', 'librosa_sf', 'librosa_sr', 'librosa_zcr']


 ### 20200804
  - nb006
    - nb004で作成した特徴量を使ってrfcモデルを作成する。
    - feat: nb004_librosa_mfcc
    - model: rfc
    - cv: 5-fold

  - kagglenb_04_sub
    - nb006のモデルを1つだけ使ってsubしてみた
    - result
      - pub: 0.544 (<---閾値大きすぎて全部nocallだったっぽい)




- memo
  - [ディスカッション](https://www.kaggle.com/c/birdsong-recognition/discussion/158908): 音データのdata augmentationについて書かれている。
  - [アライさん](https://www.kaggle.com/c/birdsong-recognition/discussion/170821#951101)はSpecAugmentとmixup効かなかったんだって。
  - ↑のディスカッションで[SpecMix](http://dcase.community/documents/challenge2019/technical_reports/DCASE2019_Bouteillon_27_t2.pdf)は効くという話もあった。

  - [birdnet](https://www.kaggle.com/c/birdsong-recognition/discussion/169538#960900)使えばラベル付できるの？

### 20200815(Sun)
SpectrogramEventRmsDatasetV4 を作成。1sec で動かせるようにする。
1secモデルを作成する(debugモードで、50epochすぐ回るようにする(train_pathを減らせばいい)。early stopping を実装。)
kernelで動作確認する
nocall データセット作成する

- kagglenb08
  - nb019で作ったモデルを提出(しなかった)
  - どうせスコアよくないのわかってたからやめた。

- nb020
  - nb019を改良
    - earlystoppingを実装
    - best stateでモデルを保存
    - resnet18 --> resnet50
    - result:
      <img src='./data/info/images/readme/018.png' width='300'>
       

- nb021
  - SpectrogramEventRmsDatasetV4 を作成。1sec で動かせるようにする。
  - まずはdf_event(nb012) が 1sec でもしっかり動作してるか確認する。
  - 境界問題をスリムにした。
  - 完成！ 音聞いたけど、いい感じだった！

- nb022
  - nb020の改良
  - SpectrogramEventRmsDatasetV4(nb021)を使用する
  - 1秒の推論を行なうモデル
  - resnet50を使用
  - result
      <img src='./data/info/images/readme/019.png' width='300'>

- memo
  - やっぱ[カエル先生](https://www.kaggle.com/c/birdsong-recognition/discussion/171247#954409)はtrainデータに5secごとのラベルを振ったみたいだな。
  - アライさんの[SEDノートブック](https://www.kaggle.com/hidehisaarai1213/introduction-to-sound-event-detection/data?scriptVersionId=40372870): ノイズは除いていない見たい。


- kagglenb09
  - nb020で作ったモデル(resnet50)を提出。
  - threshold = 0.8
  - result
    - score: 0.560
    - pubLB: 506/805    <----- クソみたいなスコアだけど、とりあえずベスト

- kagglenb10
  - アライさんの[SEDノートブック](https://www.kaggle.com/hidehisaarai1213/introduction-to-sound-event-detection/data?scriptVersionId=40372870)をフォーク。
  - DatasetをPANNsDatasetMod に差し替えてみた。
  - event_rms を使用している。
  - result
    - score: 0.544 (すべてnocallになってた)
    - ディスカッションに[理由](https://www.kaggle.com/hidehisaarai1213/introduction-to-sound-event-detection/data#963392)あった。すべ
    てnocallだったのは、モデルがない---> samplesubが提出される。ということが原因だったっぽい。
    - 自分でモデルちゃんと作らないとね...反省。


### 20200817(Mon)
- kagglenb11
  - kagglenb10のPANNsDatasetMod を PANNsDataset に差し替え。
  - kagglenb10の学習がうまくいってなかったので、もとのままではどうなのか確認する目的。
  - kagglenb10はうまくいってなかったわけじゃなかった(logのスコア比較)


- kagglenb12
  - kagglenb09(score: 0.56)のthresholdを0.8-->0.6にする。
  - result
    - score: 0.557  <---下がっとるやん


### 20200818(Tue)
- nb023
  - hard label を作成する
  - nb017で作成したevnt_rms データフレームを使用する。
  - train wav それぞれに5secのラベルを付与する
  - wavファイルの数(birdごと)
    - 最大100(100が多い)
    - 最小9
  - result
    - 失敗に終わった
    - 以下3つの例で説明する
      <img src='./data/info/images/readme/20.png' width='1000'>  

      --> 失敗例
      --> 灰色の部分の音を聞くと鳴き声が含まれている
      <br>

      <img src='./data/info/images/readme/21.png' width='1000'>  

      --> 成功例
      --> これは、非常にうまく言っている例
      <br>

      <img src='./data/info/images/readme/22.png' width='1000'>

      --> 失敗例
      --> 1つ前の例と非常に類似していてうまくいっているようにも見える。  
      --> しかし実際には灰色領域にも小さいが鳥の鳴き声が聞こえている。

    - event_rms データフレームのまとめ。

      |actual \ predict|call|nocall|
      |---|---|---|
      |call|high|**<font color='red'>high</font>**|
      |nocall|low|low|

      表でまとめてみた。  
      event_rmsデータフレームがcall(event)と判断したものが、実際にcallである場合は非常に多かった。(これまでの使い方に問題はなかった)  
      今回の試みは、nocall部分を抽出することであったがこれに失敗した。  
      event_rmsデータフレームが、nocallだと判断した部分には、多くのcallが含まれていた。
      - event_rmsデータフレームは使えない？
        - --> No！ 
        - これまでと同じような使いかたなら問題ない。
      - 簡単に言うと
        - **call だと抽出された部分は信用してもいい(未検知はあるが、過検知は少ない。 high precision)**
        - **nocall だと抽出された部分を信用してはいけない(過検知がひどい。 low recall)**

 ### 20200819
- 音響イベントと音響シーンの違い曖昧だったから、この[PDF](chrome-extension://nacjakoppgmdcpemlfnfegmlhipddanj/https://www.jstage.jst.go.jp/article/jasj/74/4/74_198/_pdf)読んでよかった。

- nb024
  - [アライさんのSEDの入門ノートブック](https://www.kaggle.com/hidehisaarai1213/introduction-to-sound-event-detection)をローカルで動かしてみる。
  - pretrained model は[こちら](https://zenodo.org/record/3987831#.Xzu9GXVfjS8)から
  - BCELoss でnanが出る。エラーの原因が正直わからない。
    - nanが出たら、early_stopping が走らないようにした

- nb025
  - nb024の改良
  - PANNsDataset-->PANNsEventRmsDatasetにした。
  - result
    - 共通のF1スコア設けてないからいいのか悪いのかわからん！   <---- バカ！！！！！！
      <img src='./data/info/images/readme/23.png' width='300'>
    


### 20200820
- kagglenb13
  - nb024 のやつ提出しようとしたけど、どうせスコア悪いからやめた。

- kagglenb14
  - nb025 のモデルデータで学習した
  - score: 0.481  <---- スコアかなり悪いな... なにがよくなかったんだろう...

- nb026
  - resnestを実装
  - [tawaraさんのノートブック](https://www.kaggle.com/ttahara/training-birdsong-baseline-resnest50-fast)を真似してみただけ。
      <img src='./data/info/images/readme/25.png' width='300'>

- nb027
  - nb026の改良
  - resnest
  - SpectralDataset を SpectrogramEventRmsDatasetV3に変更
  - result
      <img src='./data/info/images/readme/24.png' width='300'>

    

- memo
  - SEDのSOTAな手法をまとめてくれてる[discussion](https://www.kaggle.com/c/birdsong-recognition/discussion/175027)
  - DeepLearning & audio の[youtube解説](https://www.youtube.com/playlist?list=PLhA3b2k8R3t2Ng1WW_7MiXeh1pfQJQi_P)
  - [カエル先生のディスカッション](https://www.kaggle.com/c/birdsong-recognition/discussion/174187)。距離の影響について話ししている。
  - アライさんのfeaturemap aggregationについての[ディスカッション](https://www.kaggle.com/c/birdsong-recognition/discussion/167611)

### 20200821
- kagglenb14
  - nb027のモデルを提出
  - resnest
  - SpectrogramEventRmsDatasetV3
  - threshold = 0.8
  - result
    - score: 0.554

- kagglenb15
  - nb026のモデルを提出
  - resnest
  - SpectrogramDataset
  - threshold = 0.6
  - result
    - score: 0.556
    - おかしい。tawaraさんのノートブックを真似しただけのはずだが...
      - 違いと言えば、batch sizeぐらいだと思う...

### 20200822
issue#93をやる

- nb028
  - nb026のbatch_size を変えて、tawaraさんと同じ状況にしてみるだけ。
  - [tawaraさんのノートブック](https://www.kaggle.com/ttahara/training-birdsong-baseline-resnest50-fast)を真似してみただけ。
  - result
      <img src='./data/info/images/readme/26.png' width='300'>
    - 学習はうまくいっているように見える
      
    - birdcall-checkが結構違う...
  - prediction のコードが間違っているのかを確認するために以下の手順を踏んだ
    - tawara さんが学習したモデルをダウンロード
    - nb028のprediction loop にダウンロードしたモデルを使ってみる
    - result
      ---> 　[tawaraさんの公開ノートブック](https://www.kaggle.com/ttahara/inference-birdsong-baseline-resnest50-fast/data?select=best_model.pth)の結果と同じものが出力された。  
      --->  ということは、学習のコードに問題があることになる...
  



- memo
  - test_sample から nocall のデータを抽出した[discussion](https://www.kaggle.com/c/birdsong-recognition/discussion/176368)を見つけた。あとで確認する。[issue](https://github.com/fkubota/kaggle-Cornell-Birdcall-Identification/projects/1#card-44109140)化済み。

- nb029
  - stratifiedKfoldのパラメータがまったく同じなのにも関わらず、出てくる値がまったく違うということが起こっていた。
  - scikit-learnのバージョンの違いが原因のようだ....
    - scikit-learn==0.23.1にすると、一致した...
    - train と valid に使われるデータ数に差があったのが原因だったの？
  - result
    - まだtawaraさんの結果とはちょっと違う
    - でもこれたぶん、pytorchのバージョンとかのせいだと思うから、あまり気にしないでおこう。  
  <img src='./data/info/images/readme/28.png' width='300'>


- [issue#93](https://github.com/fkubota/kaggle-Cornell-Birdcall-Identification/issues/93)  
  - 解決！！
  - [mindmap](https://drive.mindmup.com/map/1DgCOa0mNgTx8Ji1EvrlWPD5mExDj4Upq)
    <img src='./data/info/images/readme/27.png' width='2000'>


### 20200823
- nb030
  - nb029を改良
  - SpectrogramDataset を SpectrogramEventRmsDatasetに変更した
  - result   
    <img src='./data/info/images/readme/29.png' width='300'>




## 鳥コンペ反省会
|著者|Ref|
|---|---|
|trtd|[url](https://docs.google.com/presentation/d/1E7fcFxzmHFAypB3ToyxhjW8IjKKRehzDjffIFD5TUW0/edit#slide=id.p)|
|ymicky06|[url](https://docs.google.com/presentation/d/1QAcl5dMW_d-J3B8AEidv737ELGq8yo9g2wOanMGL83M/edit#slide=id.p)|
|fkubota|[url](https://speakerdeck.com/fkubota/niao-konpedecan-bai-sitahua-tokonpefalsequ-rizu-mifang)|
|enu_kuro|[url](https://zenn.dev/enu_kuro/articles/d8ff124a232c576756c4)|
|nino_pira|[url](https://docs.google.com/presentation/d/e/2PACX-1vS70DWBHs8Vurd_CoqwUSWN4V2BLUJjSh2QEwd9ehWe_F3z78iRwHawpV6bAXbUuRrHZVTeEcwIl4XK/pub?start=false&loop=false&delayms=3000&slide=id.p)|
|arai|[url](https://speakerdeck.com/koukyo1994/niao-konpefan-sheng-hui-zi-liao)|
