# 4章 Webプレゼンテーション 

## 概要

MVCでコントローラには2つの使い方がある

1. 入力コントローラ
	- ユーザーの入力を解釈する
	- 適切なモデルに処理を委譲する
	- モデルが処理した結果を適切なビューに処理を委譲する
1. アプリケーションコントローラ
	- アプリケーションのフローを制御し、どの順番でどの画面を表示するか決定する
	- アプリケーションコントローラの実現方法は2パターンある
		- プレゼンテーションレイヤの一部となり、密接に連携する
		- プレゼンテーションレイヤとドメインロジックを仲介するレイヤ
	- 次の場合にはアプリケーションコントローラが有用である
		- システムに画面の順序とその間の遷移に関する多くのロジックがある場合
		- ドメイン上でページとアクションがシンプルにマッピングされている場合

### アプリケーションコントローラ

[アプリケーションコントローラ](https://bliki-ja.github.io/pofeaa/ApplicationController/)

>画面ナビゲーションやアプリケーション フローを扱う集中ポイント

| アプリケーションコントローラ |
|:----:|
|<img src="https://www.martinfowler.com/eaaCatalog/appControllerSketch.gif" width=500>|

アプリケーションには、大量の画面ロジックが含まれるものがある。こういったロジックは、様々なポイントで使われる。

- ある時間にある画面を出すといったもの
	- ウィザード形式と呼ばれる
	- 決まった画面順にユーザーが進んでいく
- ある条件下でのみ表示されるもの
- 以前の入力に応じて異なる画面が表示されるもの

ある程度は、Model View Controllerの複数のインプットコントローラでこういった判断を行うことができる。しかし、アプリケーションが複雑になると、扱う画面の異なるいくつかのコントローラが同一条件下で何を行えばよいか知る必要があり、コードが重複してしまう。

こういった重複は、すべてのフローロジックをApplication Controllerに置くことで解決できる。入力コントローラはApplication Controllerにモデルに対して実行する適切なコマンド、アプリケーションのコンテキストに合った適切なビューを尋ねる。


### 感想・気づき

- この時代はクラウドが出始めた時期だけど、この思想がクラウドに活かされている感じ
- MVCの実現方法に課題があったからMVP/MVVMが出てきたんだなとおもった

### 疑問

- コントローラを実装するときに入力コントローラ、アプリケーションコントローラを使い分けているのかな?
- There are two main forms of structuring a program in a Web server: as a script or as a server page.
	- スクリプトページとサーバーページの違いは、サーバーページがHTML構造からプログラムで記載されたロジックを呼び出すのに対して、スクリプトページはHTMLから作成するということ？

サーバページにおけるプログラム構造化には2つの手法がある

1. スクリプトとしての構造化
	- プログラムで実現されている
	- HTTP callを処理する関数やメソッドからなるプログラム
		- 例：CGIスクリプトやJavaサーブレットなど
	- HTTPリクエスト(文字列)からデータを取得する
	- HTTPレスポンスを出力する
2. サーバページとしての構造化
	- レスポンスが少ない場合に有効
	- 入力に基づいた決定が必要な場合には向かない

実際に使用する際にはこの2つを組み合わせる。つまり、リクエストの解釈はスクリプト型で実現し、レスポンスの書式化はサーバページ型を採用する。

## ビューに関するパターン

### シングルステージビュー

シングルステージビューは、アプリケーションの画面ごとに1つのビューコンポーネントを持つ。ビューは、ドメイン指向のデータを受け取り、そのデータをHTMLに変換する。

つまり、データとビューが1対1につながる。

### ツーステップビュー

[ツーステップビュー](https://bliki-ja.github.io/pofeaa/TwoStepView/)

>ドメインのデータを2ステップに分けてHTMLに入れるパターン。最初に論理的プレゼンテーションを作り、そしてHTMLへとレンダリングする

| ツーステップビュー |
|:----:|
|<img src="https://www.martinfowler.com/eaaCatalog/twoStageViewSketch.gif" width=500>|

シングルステージビューでは、共通処理の構成が、各処理に複製される。そのため、共通処理の変更すると、関連した処理が全て変更になる。

そのため、ツーステップビューでは2つのステージに分けて処理を実現する

1. モデルのデータを論理的な処理に変換:各処理個別の変換
1. 実際に実行する処理に変換:前処理共通の変換

2つ目の処理を変更することで、システムの共通処理を全て変更することが出来る

----

Webアプリケーションの画面が多くなってくると、サイトの画面デザインを統一したくなる。よくTemplate ViewやTransform Viewパターンが用いられるが、複数のページや変換モジュールをまたいでプレゼンテーションを決定するので扱いが難しくなる。これではサイト全体を変更するために、いくつものファイルを変更しなくてはならない。

Two Step Viewはこの変換を2ステップに分ける。

- モデルのデータを論理的プレゼンテーションに変換する
	- このときフォーマットは特に定めない
- その論理的プレゼンテーションを実際に必要となるフォーマットへと変換する

この方法だと、2ステップ目をいじればサイト全体の見た目を変更できる。2ステップ目を複数用意すれば、複数のルックアンドフィールを用意することも可能になる。

### シングルステージビューとツーステップビューの選択

1度に複数の表示に対応する場合や、頻繁に表示を変える場合にはツーステップビューを選択するメリットは大きい。ただ、変更容易性を持たせるためいにはそれなりの準備が必要なため、実装の難易度は高くなる。

### 感想・気づき

- ここでの選択肢は4象限なのに3つに見える書き方をしたのはなぜだろう?
- テンプレートビューやツーステップビューを無理や適用すると辛くなるイメージ
- この4象限でそれぞれ実装されているサイトを探してみて、具体的に語ってみたい
- ツーステップビューが有効な場面は限られてきそう
    - 論理画面をうまく作らないと一括変更などのメリットを得るのが難しそうだと感じた

||Transform View|Template View|
|----|----|----|
|Single Stage View|||
|Two Step View|||

### memo

フロントエンドにおける自分の最近の指向は、ツーステップビュー寄りかもです
一旦プレーンなjsオブジェクトで画面表示情報を組み立てて、ReactとかVueは、そのオブジェクトへの操作とイベントハンドリングや表示のマッピングで終わらせよう、的な

- 以前に↓みたいな感じで発表してます
- https://speakerdeck.com/tooppoo/web-front-end-design-to-avoid-being-a-fat-component


表示内容と、その表示方法を分離して、互いに密結合させないアプローチ

特に、表示内容とかその内容選択のルールは、毎回スクラップ & ビルドするようなものでも無いと思うので

>実際の表示方法は時代とともに変化するが、表示内容はそこまで変わっていない

### 疑問

## 入力コントローラに関するパターン

### 感想・気づき

- 自分の知っているコントローラの責務だった

### 疑問

## その他の要点

### 感想・気づき

### 疑問

----

## 放課後

###### tags: `PoEAA`