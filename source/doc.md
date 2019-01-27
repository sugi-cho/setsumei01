# 映像演出システム

Author: Hironori Sugino / WideWireWorks

---

## 概要

- 第69回NHK紅白歌合戦のLED映像演出に使用したシステム
- 今後、展示やライブ、様々なコンテンツの核になるよう拡張していく

---

![01](01.gif)

機材構成

--

![02](02.jpg)

--

## mac mini

- LTCを受け、タイムコードに合わせてシーン切り替えのOSCを映像表示PC（複数台）に送る
- シーン待機時、緊急時の手動オペレーション

--

## Win PC

- 本番機、予備機、録画再生機
- 映像(借景・前景)の再生/切り替え
- リアルタイム連動映像のレンダリング
- 最終アウトプットの合成、送出

---

## ソフトウェア構成

- [TouchDesigner](https://www.derivative.ca/)
- [Unity](https://unity3d.com/)
- [Max](https://cycling74.com/)

--

### mac

- TouchDesigner
  - LTCを受信し、その値をMaxに送る
- Max
  - TouchDesignerから受け取ったLTCの値を元に、シーン切り替えのOSCをWinPCに送る

--

### Win

- TouchDesigner
  - 借景動画、前景動画の再生
  - 各曲ごとに再生する動画を切り替える
  - Unityに借景動画を送る
  - Unityのリアルタイムレンダリングと前景動画の合成
- Unity
  - 照明、電飾卓から信号(ArtNet)を受け、リアルタイム照明連動美術空間のレンダリング
  - 借景動画とリアルタイムレンダリングの合成
  - TouchDesignerにレンダリングした映像を送る

--

### 今後のシンプル化

- 予備機を考えなければ、TouchDesignerとUnityのみでLTC受けから出力までPC1台構成で運用可能
- 保守の観点から、WinPCのみにして、TouchDesignerとUnityのみで構成するべき

---

## Unity

--

![unity](02.png)

--

- 美術セット、電飾、照明を配置しておく
- 電飾、照明は各灯体のDMXのUniverse、Channelを設定する
- 各曲ごとにステージの配置、ライティングの設定を行う

--

### 機能

- UDPの受信
- 照明（ムービングライト）
- 電飾（LEDライン）
- シーンの切り替え（ステージ、ライティング、演出）
- カメラからディスプレイへのマッピング

--

### UDP信号の受信

- ArtNet
  - 照明信号、電飾信号
- OSC
  - シーン切り替え情報

--

### ArtNetによる照明と電飾の同期

--

![capture](04.gif)

--

### 照明同期

- Tilt,Pan,Color等のパラメータの同期
- 光線の表現
  - VolumetricLightBeam
  - Goboの再現

--

![gobo](03.jpg)

Gobo

--

### 電飾同期

- 仮想空間上にLED電飾を配置
  - [DoubleLine、Bar LED](http://www.komaden.co.jp/products/)
  - 間接光電飾
  - 電球(カブトムシ/aiko)
  - ネオン(USA/DA PUMP)

--

### ディスプレイマッピング

- カメラ(視点)から見た絵を各ディスプレイにマッピングするシステム
- カメラの移動、ディスプレイの移動に対応
- カメラからディスプレイを見た時のパースペクティブを成立させる

--

![camera](03.gif)

パースペクティブの同期

--

![projection](04.jpg)

プロジェクション行列の計算

---

## Unity Assets

今回のプロジェクトに使ったアセット

--

- [Hx Volumetric Lighting](http://hitboxteam.com/HxVolumetricLighting/)
  - ボリュームライティングの表現に使用
  - Cookieを設定することで、Goboの再現も可能
  - カメラのAABB Planesによるカリングの処理を修正
  - ボリュームライトの減衰の処理をしているShaderを修正
  - RayCastとColliderを使用した光線の突き抜け防止機能の追加

--

- [ArtNet.Unity](https://github.com/sugi-cho/ArtNet.Unity)
  - [ArtNet](https://en.wikipedia.org/wiki/Artnet)(DMX via UDP)
  - [ArtNet.Net](https://github.com/MikeCodesDotNET/ArtNet.Net)(.Net(C#)用ライブラリ)
- [UdpData-Unity](https://github.com/sugi-cho/UdpData-Unity)
  - OSCの受信に使用
  - [OSC](https://ja.wikipedia.org/wiki/OpenSound_Control)(Open Sound Control)
  - UDPのレコード機能の実装(テストデータの作成用)

--

- [KlackSpout](https://github.com/keijiro/KlakSpout/tree/master/Assets/Klak/Spout)
  - [Keijiro Takahashi](https://github.com/keijiro)
  - [Spout](http://spout.zeal.co/)を使用した複数ソフトウェア間でのTextureの共有

--

- PerspectiveSyncSystem(仮)
  - [Kodai Takao](https://github.com/kodai100)による実装
  - [参照、解説 (第9章 Multi Plane Perspective Projection)](https://indievisuallab.stores.jp/items/59edf11ac8f22c0152002588)
  - [Unityの別APIを使用した実装](https://github.com/sugi-cho/VrWindowRenderer)

---

## 使用しなかったアセット

導入を検討したが使用しなかったアセット

--

- [VolumetricLightBeam](http://saladgamer.com/vlb-doc/)
  - ポストエフェクトではなく、コーンオブジェクトを使用した照明の光線の再現
  - GPU Instancingを使用したレンダリングにより描写が非常に高速
  - オブジェクトを使用した実装なので光源に大きさがある光線の表現も可能
  - LightCookieに対応していない
    - 開発版のCookie対応したものを試させてもらったが、まだ、かなり重いようだった

---

## 今後の方針

- 展示でもライブでも応用できるベースシステムにする（汎用化）
- 照明、電飾の配置ワークフローの効率化を検討
- 様々な入力に対応する
- 安定する機材を検証、選定する

--

### 照明、電飾表現について

- HDRPで実装しなおしてみる
  - フルカラーテクスチャをGoboとした光線表現が使用可能
  - 処理負荷を計測して、実戦投入可能か判断
  - 照明、電飾の表現をよりリッチにできる？
- 汎用照明灯体を開発する
  - 現在は実際にある照明灯体を再現して使用しているが、仮想空間の演出においては、理想的な灯体が1つあれば、十分なはずで、チャンネル数、ユニバース数も抑えられる
- ArtNet送信による実照明の演出連動

--

### ステージの切り替え機能

- ほぼほぼ紅白専用で作られているので汎用化が必要
  - 部分部分の応用は可能そう
- 現状、ステージの切り替えは`gameObject.SetActive`で行っている
  - パッと一瞬で切り替えるしかできない
  - ステージセットの登場の仕方、退場の仕方をアニメーション等カスタムできるようにしたい