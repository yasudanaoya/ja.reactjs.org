---
id: codebase-overview
title: コードベースの概要
layout: contributing
permalink: docs/codebase-overview.html
prev: how-to-contribute.html
next: implementation-notes.html
redirect_from:
  - "contributing/codebase-overview.html"
---

このセクションでは React コードベースの構成や規則そして実装についての概要を説明します。

あなたが [React にコントリビュート](/docs/how-to-contribute.html)したい場合に、このガイドがあなたがより快適に変更を行えるように手助けとなる事を願っています。

これらの規約のいずれかをあなたの React アプリケーションで推奨しているというわけでは必ずしもありません。規約の多くは歴史的な理由で存在しており、時間とともに変化する可能性があります。

### 最上位フォルダ {#top-level-folders}

[React リポジトリ](https://github.com/facebook/react)をクローンした後、プロジェクトのルートディレクトリに複数のフォルダがあることに気がつくでしょう：

* [`packages`](https://github.com/facebook/react/tree/main/packages) には React リポジトリの全てのパッケージのメタデータ（`package.json` など）や、ソースコード（`src` サブディレクトリ）が含まれています。**あなたの変更がコードに関するものなら、各パッケージの `src` サブディレクトリが作業時間のほとんどを過ごす場所となります**。
* [`fixtures`](https://github.com/facebook/react/tree/main/fixtures) にはコントリビューター向けの React の小さなテスト用アプリケーションが含まれています。
* `build` は React のビルド出力です。リポジトリには存在しませんが、最初に [React をビルドした](/docs/how-to-contribute.html#development-workflow)後に clone した React ディレクトリに現れます。

ドキュメントは [React 本体とは異なるリポジトリ](https://github.com/reactjs/reactjs.org)にホスティングされています。

他にも最上位にあるフォルダがありますが、ほとんどはツールが使用するためのもので、コントリビュートする場合にそれらに直面することはほぼありません。

### テストコードをソースコードのそばに格納する {#colocated-tests}

ユニットテスト用の最上位ディレクトリはありません。代わりに、テスト対象のファイルのすぐ隣の `__tests__` というディレクトリに置いています。

例えば、[`setInnerHTML.js`](https://github.com/facebook/react/blob/87724bd87506325fcaf2648c70fc1f43411a87be/src/renderers/dom/client/utils/setInnerHTML.js) のテストは、すぐ隣の [`__tests__/setInnerHTML-test.js`](https://github.com/facebook/react/blob/87724bd87506325fcaf2648c70fc1f43411a87be/src/renderers/dom/client/utils/__tests__/setInnerHTML-test.js) にあります。

### 警告と不変数 (Invariant) {#warnings-and-invariants}

React のコードベースは警告の表示に `console.error` を使用しています：

```js
if (__DEV__) {
  console.error('Something is wrong.');
}
```

警告は開発時にのみ有効になります。本番時には警告は完全に取り除かれます。コードの一部の実行を禁止する必要がある場合は、代わりに `invariant` モジュールを使用してください：

```js
var invariant = require('invariant');

invariant(
  2 + 2 === 4,
  'You shall not pass!'
);
```

**invariant は `invariant` 内の条件が `false` のときスローされます。**

"Invariant" とは "この条件は常に true である" ということの単なる言い換えです。アサーションを作成していると考えることができます。

開発バージョンと本番バージョンで同じ動作となるようにしておく事は重要ですので、`invariant` は両方の環境でスローされます。本番バージョンではエラーメッセージは自動的にエラーコードに変換され、バイト数に悪影響が出ることを避けます。

### 開発バージョンと本番バージョン {#development-and-production}

コードベース内で `__DEV__` 擬似グローバル変数を使用して、開発用のコードブロックを保護することができます。

コンパイル時にインライン化され、CommonJS のビルド時には `process.env.NODE_ENV !== 'production'` というチェックに変換されます。

スタンドアロンなビルドにおいては、minify されないビルドでは `true` となり、minify されたビルドではそれを囲む `if` ブロックごと完全に除去されます。

```js
if (__DEV__) {
  // This code will only run in development.
}
```

### Flow {#flow}

最近私たちはコードベースに [Flow](https://flow.org/) によるチェックを導入し始めました。ライセンスヘッダで `@flow` アノテーションがマークされているファイルは型チェックをされています。

[既存のコードに Flow アノテーションを追加する](https://github.com/facebook/react/pull/7600/files)プルリクエストを受けつけています。Flow アノテーションは以下のようになります：

```js
ReactRef.detachRefs = function(
  instance: ReactInstance,
  element: ReactElement | string | number | null | false,
): void {
  // ...
}
```

可能であれば、新しいコードは Flow アノテーションを使うべきです。
`yarn flow` を実行することでローカル環境で Flow を使ってコードをチェックできます。

### 複数のパッケージ {#multiple-packages}

React は[単一リポジトリ (monorepo)](http://danluu.com/monorepo/) です。そのリポジトリには複数の別々のパッケージが含まれているので、それらの変更はまとめて調整でき、issue も 1 箇所にまとまっています。

### React コア (core) {#react-core}

React の「コア (core)」は[最上位の React API](/docs/react-api.html#react) を全て含んでいます。例えば：

* `React.createElement()`
* `React.Component`
* `React.Children`

**React core にはコンポーネントの定義に必要な API のみが含まれています。**[リコンシリエーション](/docs/reconciliation.html)のアルゴリズムやあらゆるプラットフォーム固有のコードは含まれません。コアは React DOM や React Native コンポーネントに使われています。

React core のコードはソースツリーの [`packages/react`](https://github.com/facebook/react/tree/main/packages/react) に格納されています。npm では [`react`](https://www.npmjs.com/package/react) パッケージとして利用可能です。対応するスタンドアロンなブラウザ向けのビルドは `react.js` と呼ばれ、`React` というグローバル変数をエクスポートします。

### レンダラ {#renderers}

React は元々 DOM のために作成されましたが、後になって [React Native](https://reactnative.dev/) によりネイティブなプラットフォームもサポートするようになりました。これにより React の内部に "レンダラ (renderer)" の概念が導入されました。

**レンダラは React ツリーを、基盤となるプラットフォーム固有の呼び出しへと変換する方法を管理します。**

レンダラは [`packages/`](https://github.com/facebook/react/tree/main/packages/) にも格納されています：

* [React DOM Renderer](https://github.com/facebook/react/tree/main/packages/react-dom) は React コンポーネントを DOM に変換します。[トップレベルの `ReactDOM` API](/docs/react-dom.html) を実装し、npm では [`react-dom`](https://www.npmjs.com/package/react-dom) パッケージとして利用可能です。`react-dom.js` と呼ばれるスタンドアロンなブラウザ向けバンドルとしても使用でき、`ReactDOM` というグローバル変数をエクスポートします。
* [React Native Renderer](https://github.com/facebook/react/tree/main/packages/react-native-renderer) は React コンポーネントをネイティブのビューに変換します。React Native の内部で使われています。
* [React Test Renderer](https://github.com/facebook/react/tree/main/packages/react-test-renderer) は React コンポーネントを JSON ツリーに変換します。[Jest](https://facebook.github.io/jest) の[スナップショットテスト機能](https://facebook.github.io/jest/blog/2016/07/27/jest-14.html)で使用され、npm では [react-test-renderer](https://www.npmjs.com/package/react-test-renderer) パッケージとして利用可能です。

その他に公式にサポートしているレンダラは [`react-art`](https://github.com/facebook/react/tree/main/packages/react-art) だけです。個別の [GitHub リポジトリ](https://github.com/reactjs/react-art)にありましたが、現在はメインのソースツリーに移動しました。

>**補足：**
>
>技術的に、[`react-native-renderer`](https://github.com/facebook/react/tree/main/packages/react-native-renderer) は React に React Native の実装とやり取りの方法を教える非常に薄い層です。ネイティブのビューを管理する実際のプラットフォーム固有のコードは、コンポーネントと共に [React Native リポジトリ](https://github.com/facebook/react-native) にあります。

### リコンサイラ (reconciler) {#reconcilers}

React DOM と React Native のように大幅に異なるレンダラでも多くのロジックを共有する必要があります。特に、ツリーの[リコンシリエーション](/docs/reconciliation.html)のアルゴリズムは可能なかぎり同じものにして、宣言型レンダリング、独自コンポーネント、state、ライフサイクルメソッドおよび ref がプラットフォーム間で一貫した動作となるようにするべきです。

異なるレンダラ間でコードの一部を共有することでこの課題を解決します。React のこの箇所を "リコンサイラ（reconciler, 差分検出処理）" と呼んでいます。`setState()` のような更新がスケジュールされると、差分検出処理ではツリー内のコンポーネントで `render()` を呼び出して、コンポーネントをマウント、更新、もしくはアンマウントします。

リコンサイラは、パブリック API がないため、個別にパッケージ化されていません。代わりに、それらは React DOM や React Native などのレンダラーによってのみ使用されます。

### スタック (stack) リコンサイラ {#stack-reconciler}

"stack" リコンサイラは React 15 およびそれ以前で実装されています。それ以降は使用されていませんが、[次のセクション](/docs/implementation-notes.html)では詳細に説明されています。

### Fiber リコンサイラ {#fiber-reconciler}

"fiber" リコンサイラは stack リコンサイラが内在的に抱える問題を解決し、いくつかの長年の問題を修正することを目的とした新しい取り組みです。React 16 以降はデフォルトのリコンサイラとなっています。

主な目的は以下の通りです：

* 中断可能な作業を小分けに分割する機能
* 進行中の作業に優先順位を付けたり、再配置や再利用をする機能
* React でレイアウトをサポートするための親子間を行き来しながらレンダーする機能
* render() から複数の要素を返す機能
* error boundary のサポートの向上

React Fiber のアーキテクチャに関して[ここ](https://github.com/acdlite/react-fiber-architecture)や[ここ](https://blog.ag-grid.com/inside-fiber-an-in-depth-overview-of-the-new-reconciliation-algorithm-in-react)で読むことができます。React 16 と共にリリースされていますが、非同期機能についてはデフォルトではまだ有効化されていません。

ソースコードは [`packages/react-reconciler`](https://github.com/facebook/react/tree/main/packages/react-reconciler) に格納されています。

### イベントシステム {#event-system}

React はブラウザ間でのイベント挙動の差異を吸収するため、ネイティブイベントの上にレイヤーを実装しています。ソースコードは [`packages/react-dom/src/events`](https://github.com/facebook/react/tree/main/packages/react-dom/src/events) にあります。

### この次は？ {#what-next}

React 16 以前のリコンサイラについてより詳細に学ぶには[次のセクション](/docs/implementation-notes.html)を読んでください。新しいリコンサイラの内部についてはまだドキュメント化していません。
