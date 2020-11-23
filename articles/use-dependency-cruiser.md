---
title: "依存関係を検証してコードの秩序を保とう"
emoji: "🧐"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["javascript","react"]
published: true
---

今回は以下のような困りごとを解決するための一手段を紹介します。  
- atomic designを採用したため、下位層から上位層のcomponentをimportしてしまっているか検証したい
  - 例: atomsからmoleculesやorganismsをimport
- 特定のライブラリを使用していないかどうかを検証したい
- 循環参照をしてしまっていないか検証したい

## 依存関係を検証する
[dependency-cruiser](https://github.com/sverweij/dependency-cruiser)を使用して検証していきます。
尚、公式ドキュメントを読む際は[こちらのページ](https://github.com/sverweij/dependency-cruiser/blob/develop/doc/rules-tutorial.md)が比較的理解しやすかったです。

## 導入方法

導入は非常に簡単です。
```shell
npm install --save-dev dependency-cruiser
npm depcruise --init
```
以上のコマンドを使用するといくつかの質問の後に自動でルールが生成されるはずですが、今回は進行上の都合のためルールを下記のように削除して進めます。
（便利なルールが記載してあるため実際に導入する際は削除しないことをお勧めします。）

```javascript
module.exports = {
  forbidden: [],
};
```

## 特定のcomponentをimportしている場合にerrorを出力する
今回はatomic designを例にしてルールを追加していきます。
以下のコードは、atoms・molecules・organisms・templatesからpagesのcomponentをimportしている場合にerrorを出力させるルールです。

```javascript
module.exports = {
  forbidden: [
    {
      name: "atomic-design",
      from: {
        path: "components/(atoms|molecules|organisms|templates)/.+",
      },
      severity: "error",
      to: { path: "components/pages/.+" },
    },
  ],
};
```

何かを禁止したい際には、基本的にforbiddenの中にオブジェクトを記載していきます。
今回の場合はatomsなどからpagesをimportさせたくないため、fromに```"components/(atoms|molecules|organisms|templates)/.+"```、toに```"components/pages/.+"```を記載しています。
また、dependency-cruiserはデフォルトではwarnを出力しますが、今回はerrorを出力させたいのでseverityにerrorを指定しています。
このように、非常に簡単にルールを追加していくことが可能です。

## 特定のライブラリを直接使用している場合errorを出力する
ライブラリを使う際に、何らかの事情で自作wrap関数からのみimportさせたい場合があるかと思います。
今回はreact-useを例にルールを記載します。

```javascript
module.exports = {
  forbidden: [
    {
      name: "Do not use the react-use directly",
      from: {},
      severity: "error",
      to: {
        path: "react-use",
        dependencyTypes: ["npm"],
      },
    },
  ],
};
```
前述した方法と基本的には同じですが、今回はtoに[dependencyTypes](https://github.com/sverweij/dependency-cruiser/blob/develop/doc/rules-reference.md#dependencytypes)を指定しています。
"npm"と指定することで「package.jsonのdependenciesのreact-use」のみを対象とすることが可能です。
これにより、例えreact-useという同名のファイルが存在していたとしてもそれを誤検知しないようにするというメリットを享受できます。


## ライブラリの特定の関数を直接使用している場合にerrorを出力する

:::message alert
node_modules内部を対象に含めるため動作が重くなります。
:::
上記のルールをあるライブラリの特定の関数のみに適用させたい場合について記載します。

```javascript
module.exports = {
  forbidden: [
    {
      name: "Do not use the useToggle directly",
      from: { path: "components/.+" },
      severity: "error",
      to: {
        path: "^node_modules/react-use/lib/useToggle",
        reachable: true,
      },
    },
  ],
};
```
```dependencyTypes: ["npm"]```を指定した場合、node_modules/react-use/lib/index.jsまでしか検証されません。
今回は特定の関数のみを対象としたいため、dependencyTypesを指定せずに直接pathを記載します.
また、options.doNotFollowに以下のように記載されている場合はnode_modulesが検証対象外となるため削除してください。
```js
module.exports = {
  options: {
    doNotFollow: {
      path: "node_modules",
    },
  },
};
```


## 感想など
プロジェクトが大きくなればなるほど便利になるツールだと思われます。
