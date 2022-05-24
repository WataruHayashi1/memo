# プレゼンテーションコンポーネントとコンテナコンポーネント

[Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)という記事を翻訳したものです。

> **Warning**
> 
> 翻訳が非常に怪しいので、信用しすぎないように

## 記事内容

Reactアプリを作っているときに見つけた非常に使いやすいシンプルなパターンがある。
[あなたも少しでもReactを書いたことがあるなら](https://reactjs.org/blog/2015/03/19/building-the-facebook-news-feed-with-relay.html)、すでに見つけているであろう。
この記事は余計な点を加えずに、[それについて説明するものである](https://medium.com/@learnreact/container-components-c0e67432e005)。

**プレゼンテーション**コンポーネントとは

- 見た目にこだわる。
- プレゼンテーションとコンテナの両方のコンポーネントが含まれることがあり、通常、いくつかのDOMマークアップとスタイルを持っている。
- よくthis.props.childrenを介した格納を可能にする。<!-- 非常に怪しい訳 -->
- Fluxやstoresのようなアプリの他の部分には依存しない。
- データの読み込みや変換の方法を指定しない。
- propsからデータやコールバックを排他的に受け取る。
- まれに独自のstateを持つことがある(データというよりUI state)
- stateやlifestyle hooks、パフォーマンスの最適化の結果が必要としない限り、[関数コンポーネント](https://reactjs.org/blog/2015/10/07/react-v0.14.html#stateless-functional-components)で記述される。
- 例 : Page、Sidebar、Story、UserInfo、Listなど

**コンテナ**コンポーネントとは

- 動作、仕組みにこだわる。
- プレゼンてーションとコンテナの両方のコンポーネントがh組まれることがあるが、通常、いくつかのラップdivを除いた自身のDOMマークアップを持たず、いかなるスタイルも持たない。
- プレゼンテーションや他のコンテナコンポーネントにデータや振る舞いを提供する。
- Fluxを呼び出し、コールバックとしてプレゼンテーションコンポーネントとして提供する。
- データソースとして機能するため、おおむねステートフルである。
- 通常、手動で書かれるより、React Reduxからのconnect()、RelayからのcreateContainer()、FluxUtilsからのContainer.create()のような[階層が上のコンポーネント](https://medium.com/@dan_abramov/mixins-are-dead-long-live-higher-order-components-94a0d2f9e750)を用いて作られる。
- 例 : UserPage、FollowersSidebar、StoryContainer、FollowedUserListなど

この2つは、別フォルダーに分けて違いをはっきりさせる。

**このアプローチの利点** 

- 懸念点の分離をよりよくする。この方法でコンポーネントを書くことにより、アプリケーションとUIを理解しやすいものにする。
- 再利用性の向上。まったく異なるstateで同じプレゼンテーションコンポーネントを作ることができ、異なるコンテナコンポーネントに組み込むことができる。さらにそれを再利用することができる。
- プレゼンテーションコンポーネントはアプリに必要なパレットである。コンポーネントをシングルページに適用し、デザイナーがアプリのロジックに触れることなく微調整するようにできる。そのページでスクリーンショットの再帰テストを実行できる。
- Sidebar、Page、ContextMenuのようなレイアウトコンポーネントを仕様できないようにし、それぞれのコンテナコンポーネントのレイアウトと重複したマークアップの代わりにthis.props.childrenを使うようにする。

思い出してほしいのは、コンポーネントはDOMを出力してはいけないということである。UIに関係する構成境界を提供するのみ必要である。