---
title: "【monorepo編】後輩ちゃんと学ぶ自作フレームワーク！！ binフィールドとprocess.cwd()で『規約』をつくる"
emoji: "📦"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["monorepo", "pnpm", "vite", "react", "自作フレームワーク"]
published: true
---

:::message
この記事の目的
普段はNext.jsなどのフレームワークを使って開発しているエンジニアが、内部を理解するためにフレームワークを自作してみる記事です。
執筆時点で筆者が公開しているフレームワークはこちらにあります
https://github.com/AN-Sippo/sippa

**全5回シリーズの第4回です。**

- [第1回：基礎編 — Vite×React Router](https://zenn.dev/sippooo/articles/frontend-framework-1)
- [第2回：SSR編](https://zenn.dev/sippooo/articles/frontend-framework-2)
- [第3回：React Router編 — Hydration failedの正体とloaderの仕組み](https://zenn.dev/sippooo/articles/frontend-framework-3)
- 第4回：monorepo編 — binフィールドとprocess.cwd()で『規約』をつくる（この記事）
- [第5回：本番ビルド編 — ビルドのエントリポイントは2つある](https://zenn.dev/sippooo/articles/frontend-framework-5)

:::

## 登場人物たち

| 見た目 　                            | 名前   | プロフィール                                            |
| ------------------------------------ | ------ | ------------------------------------------------------- |
| ![](/images/general/Sippo.png =100x) | シッポ | 筆者。Webフロントエンドが少しだけ分かる。               |
| ![](/images/general/anzu.png =100x)  | アンズ | Sippoの後輩。技術に詳しい。とりあえずなんでも知っている |

**アンズ**

> では、今回も自作フレームワークの続きをやっていきますよっ！
> せんぱい、前回でどこまでやったか、覚えていますか？

**シッポ**

> 前回までで、ViteとReact-Routerを使って、SSRとルーティング・loaderの実行までが動作するようになったよね。

**アンズ**

> そうですね！
> より詳しく言うと、ViteとReact-Routerを使って、SSRとルーティング・loaderが、**ローカル開発環境**で動作する**アプリケーション**が完成した、
> という状態になっていますよね！

**シッポ**

> そうだね。
> いまできてるのは動作確認用のWebアプリであって、まだフレームワークの形にはなってない。
> このアプリ以外が、今回実装したSSRやルーティングの機能を使うことはまだできないよね。

**アンズ**

> その通りです！
> なので、今回の記事ではこれを目標にします！

:::message
本記事の目標

1. SSR・ルーティングの機能を、他のアプリが使える状態に持っていく

:::

**アンズ**

> さっそく「SSR・ルーティングの機能を、他のアプリが使える状態に持っていく」をやっていきたいんですがこれ、どう進めたらいいか、見通し立ちますか？

**シッポ**

> う、うーん？　
> まず、タイトルにある通りmonorepo構成にするでしょ？　その後は……
> 全然わからないです。

**アンズ**

> monorepo構成にするのは正解だと思います！　わざわざnpmに公開しなくても手元だけでフレームワークの動作確認ができますからね！
> そのあとは……うーん、そうですよねえ。
> フレームワークとかライブラリって今はきれいにnpmとかがラップしてくれてるので、普通に生きてたら直接触る機会ないですもんね。

**シッポ**

> お恥ずかしながら……

**アンズ**

> だ、だいじょうぶです！
> 今回でばっちり理解しちゃいましょう！

## 実装方針をたてる

**アンズ**

> まずなんですが、今動作してるアプリケーションがありますよね？
> これはSSR/ルーティング/loaderの全てが動作していて、完全に要件を満たしていますね？

**シッポ**

> うん？　そうだね。
> これまでそういう風に作ってきたもんね。

**アンズ**

> そしたら、仮にmonorepo構成を以下のようにしたとしましょう！

```js
packages
├── myframework // 今のプロジェクトはこっちになる予定です
└── sample // 動作させたいアプリ。フレームワークの「ユーザー」にあたる

```

> このとき、`sample`側で今の動作確認アプリと同じものが動作すれば、いいですよね？

**シッポ**

> そうなるね。
> 今のアプリを動作させて、かつ、アプリの仕様変更に`sample`側の変更だけで対応できたら良いってことだよね？

**アンズ**

> あっ！　そうですそうです！
> 「アプリの仕様変更に`sample`の変更だけで対応できたら良い」
> これ、めちゃくちゃ良い表現だと思います！

> そうしたら、「アプリの仕様変更に`sample`の変更だけで対応する」ためにsampleアプリ側に移動させないといけないファイル/フォルダはどれでしょうか！

**シッポ**

> まず、アプリそのものである`/pages`は`sample`側に置きたいよね。
> それから、アプリに別のルートを増やしたいかもしれないから、`route.ts`もそうだね。
> `index.html`は……どっちなんだろう？

**アンズ**

> いいですね！　`index.html`で迷ってるところまで含めて完璧です！
> この子、機能としては間違いなくアプリ側からいじれる必要があるんですが、役割的にはフレームワーク側が近いですし、アプリ側からいじれなくなっても大まかな変更には問題ないことが多いんですよね。

**シッポ**

> たしかに、`index.html`に`<meta>`,`<link>`とか追記したくなることはありそうだもんね。
> でも、そこをいじれなくても全くアプリが作れないわけじゃないもんね。

**アンズ**

> そういうことです！
> 最近では、隠蔽しつつもいじるためのインターフェースをフレームワークとして提供する。というアプローチをしているケースが多い印象がありますね。
> ですが今回は、あくまで全体を理解することが目的なので、`index.html`を隠蔽しつつも、いじるためのインターフェースは用意しない判断でやっていこうと思います！

**シッポ**

> おぉ……。大胆。

**アンズ**

> と、少し話が逸れましたが、`/pages`・`route.ts`の２つを`sample`側に置くことができたら、今回の目標が達成できそうですよね？

:::message
本記事の目標(再掲)

1. SSR・ルーティングの機能を、他のアプリが使える状態に持っていく

:::

**シッポ**

> そうだね。いいと思う。

**アンズ**

> ありがとうございますっ！
> そうしましたら、目標を達成するための手順を整理しますね！

:::message
SSR・ルーティングの機能を他のアプリが使える状態にする手順

- [ ] 1. monorepo構成にする
- [ ] 2. `sample`からフレームワークを呼び出せるようにする
- [ ] 3. `/pages`・`route.ts`を`sample`パッケージに移動する

:::

> それではこれを道しるべに、やっていきましょー！

## monorepo構成にする

まず、`packages/`配下に`myframework`・`sample`の２パッケージをつくって、これまで作っていたファイルたちを全て、`packages/myframework`以下に移動させます。

```zsh
mkdir -p packages/myframework #ここにこれまで作っていたファイルを丸ごと移動
mkdir -p packages/sample
```

移動できたら、`packages/myframework/package.json`の`name`を`myframework`に変えておいてください！
第1回で`pnpm init`したときの名前のままになっているはずなんですが、あとで`sample`から`"myframework": "workspace:*"`という形で参照するので、ここが一致していないと`pnpm install`が失敗してしまいます。

```json:packages/myframework/package.json
{
  "name": "myframework",
  "version": "1.0.0",
  "main": "index.js",
  "type": "module",
  "scripts": {
    "dev": "tsx src/server.ts"
  },
  "dependencies": {...},
  "devDependencies": {...}
}

```

そしたらmonorepoのルートに２つのファイルを作成してください！

```json:package.json

{
  "name": "educational_monorepo_root",
  "private": true
}

```

```yaml:pnpm-workspace.yaml

packages:
  - "packages/*"

```

最後に、一度pnpmのロックファイルを消して、入れ直します。
pnpmのmonorepo構成では、ロックファイルをルートに１つだけ持つことになっているんです！

```shell:プロジェクトルートで実行
rm packages/myframework/pnpm-lock.yaml
pnpm install
```

これで、monorepo構成にする準備が整いました！
ここはテーマからズレるので詳しく解説しませんが、もし気になったらあとで調べてみてくださいっ。「monorepo」あるいは「モノレポ」で調べればすぐヒットするはずです！

今回はファイル操作が複雑だったので、treeコマンドの結果も載せておきます！
途中で、「なんか変だぞ？」ってなったら、せんぱい自身のコードと見比べてみてください！

```zsh

tree -I "node_modules"
.
├── package.json #今回新しく作ったほうです！
├── packages
│   ├── myframework #ここに今までのファイルを全て移動させました！
│   │   ├── index.html
│   │   ├── package.json
│   │   ├── src
│   │   │   ├── entry-client.tsx
│   │   │   ├── entry-server.tsx
│   │   │   ├── pages
│   │   │   │   ├── Dog.tsx
│   │   │   │   ├── Index.tsx
│   │   │   │   └── counter.tsx
│   │   │   ├── route.ts
│   │   │   └── server.ts
│   │   └── tsconfig.json
│   └── sample
├── pnpm-lock.yaml
└── pnpm-workspace.yaml
```

**アンズ**

> と、なんかすごい一気に解説しちゃいましたけど、だいじょぶですか？

**シッポ**

> た、たぶん……
> とりあえず、monorepo構成では`packages`配下にパッケージを並べて、ルートの`pnpm-workspace.yaml`で指定してあげればいいんだね？

**アンズ**

> pnpmを使ってる限りはですが、その通りです！
> たぶんここは使っていくうちに慣れていく分野なので、今完璧じゃなくてもいいですよっ！

**シッポ**

> そっか。了解。
> じゃあ、ここはざっくりと抜けて、次に進んじゃうね。

**アンズ**

> それが良いと思います！
> 次は、今作った`sample`パッケージから`myframework`を呼び出すところをやっていきます！

## `sample`からフレームワークを呼び出せるようにする

**アンズ**

> さて。
> これまでの作業は全部１つのパッケージ、今で言う`myframework`配下で全て行ってきましたね？
> ここからは、複数パッケージをまたいでの作業となります！

**シッポ**

> 複数パッケージ。`myframework`と`sample`だね

**アンズ**

> そうです！
> 特に今回は、`sample`から`myframework`の機能を呼び出せるようにするのが目的ですよ！
> そんな時に使われるのが、`package.json`の`exports`フィールド・`bin`フィールドです！

```json:package.json(まだ追記しなくていいやつです！)
  "exports": {
    ".": "./dist/index.js"
  },
  "bin": {
    "myframework": "./dist/cli.js"
  },
```

> ここのフィールドにパスを指定すれば、外からそのファイルを参照できるようになります！
> 細かく知りたかったらぜひ調べてみてほしいんですが、大まかに、

|         |                                                                      |
| ------- | -------------------------------------------------------------------- |
| exports | 外から`import hoge from myframework`のようにできる(今回は使いません) |
| bin     | `pnpm myframework`のようにcliから実行できる                          |

> こんな整理が出来ていれば今回は十分です！

**シッポ**

> あー、なるほど。npmのパッケージはそうなってたんだ。
> `exports`と`bin`フィールドね。
> あれ、そういえばそこに指定してるファイルは`.js`なんだね。そうか、外部に公開する以上、JavaScriptから呼ばれる可能性もあるから、`.js`にするしかないんだ。

**アンズ**

> よく気が付きました！　そう。`.js`なんですよ。
> 理由も、せんぱいの理解で概ね合っています！　実は「`.ts`のままでもいい」ケースもあるにはあるんですが、迷ったら毎回きちんと`.js`にビルドしてあげても、あんまり困らないと思います。

**シッポ**

> そうなんだ。
> とりあえず例外はあるにしろ、exports,binフィールドには`.js`ファイルを指定するのが無難ってことだね。

**アンズ**

> そういうことです！
> というわけでここから、ビルドして`.js`ファイルを作って、それを`package.json`に指定して、それを`sample`から参照してみましょう！

### binフィールドを疎通させる

まず、`.js`へコンパイルするコマンドを作ってあげます……と言いたいところなんですが、まずは、`bin`フィールドに指定するためのcli専用ファイルを作っておきましょう！

```ts:myframework/src/cli.ts
#!/usr/bin/env node

// server.ts側は下のように、createServerをexportするよう変えておいてください！
import createServer from "./server.js";

const cmd = process.argv[2];

const dev = () => {
  createServer();
};

switch (cmd) {
  case "dev":
    dev();
    break;
  default:
    console.error("Invalid command");
    process.exit(1);
}
```

```ts:myframework/src/server.ts
(中略)
// createServer(); 呼び出しを削除して、exportします
export default createServer;
```

こうしておくと、`pnpm myframework dev`というコマンドを実行したときに`createServer`が走って、今までと同じサーバーが起動します！

ここで、コンパイルに使うesbuildを入れておきます！

```zsh
pnpm add -D esbuild
```

そしたら、これで必要なファイルが準備できましたので、コンパイルのコマンドを追加してあげます！
ついでに、もう使わない`dev`コマンドは消しておきます。

```json:myframework/package.json
  "scripts": {
    //こっちは消してください！ "dev": "tsx src/server.ts",
    "compile": "esbuild \"src/**/*.ts\" \"src/**/*.tsx\" --outdir=dist --packages=external"
  },
```

実行してみると………

```zsh
pnpm run compile
```

こんな感じで、`myframework/dist`が生成されているはずです！
![](/images/frontend-framework-4/dist.png)

そうしたら、先程追加した`cli.ts`のビルド後である`cli.js`をbinフィールドで公開してあげたら、外から使えるようになりそうですよね？

```json:myframework/package.json
{
  (略)
  "bin": {
    "myframework": "./dist/cli.js"
  },
  "dependencies": {...},
  "devDependencies": {...}
}
```

これを、sampleパッケージから使ってみましょう！
monorepo構成でパッケージを参照するには、[workspaceプロトコル](https://pnpm.io/ja/workspaces#%E3%83%AF%E3%83%BC%E3%82%AF%E3%82%B9%E3%83%9A%E3%83%BC%E3%82%B9%E3%83%97%E3%83%AD%E3%83%88%E3%82%B3%E3%83%AB-workspace)を使います！

```json:sample/package.json
{
  "name": "sample",
  "private": true,
  "dependencies": {
    "myframework": "workspace:*"
  }
}

```

```shell:/sampleで実行
pnpm install
```

これで試してみると………

```shell:/sampleで実行
pnpm myframework dev
```

```shell:実行結果(URLアクセス時)
>>> pnpm myframework dev

Server is running at http://localhost:5173
Error: ENOENT: no such file or directory, open '/.../packages/sample/index.html'
```

なにやらエラーが出ていますね。
ですが、サーバーは起動しています……！

エラーは次の章で解説しますので、一旦これで、monorepo構成で外のパッケージと疎通することができましたね！

**アンズ**

> ……どうでしょう。せんぱい。
> ここまでついてこられていますか？

**シッポ**

> う、うーん。たぶん大丈夫なはず。
> workspaceプロトコルとか、cli.tsの中身とかは雰囲気でしか理解できてないけど。
> workspaceプロトコルは、とにかくmonorepoの別パッケージを参照するための「なにか」。
> cli.tsは引数でswitchして色んな処理を書けるような設計にしてて、現状では`dev`だけが定義されてる状態ってことであってる？

**アンズ**

> えっ、ばっちりですよ！
> その辺りは難しいので、「とりあえず動けばおっけー！」くらいの気持ちで大丈夫です！
> ……ですが、念のため、軽くまとめておきましょうか！

:::message
sampleパッケージから`pnpm myframework dev`を疎通させるまでの手順

1. cli.tsを追加して、引数がdevなときに、createServerを呼び出す
2. これらをesbuildでビルドして、`dist/cli.js`を作成する
3. myframeworkパッケージの`package.json`に`myframework: dist/cli.js`と指定する
4. sampleパッケージからworkspaceプロトコルを用いて、myframeworkパッケージを依存関係に追加する

:::

> あ！　それと、monorepo構成にできたので、ここまでチェックしておきますね！

:::message
SSR・ルーティングの機能を他のアプリが使える状態にする手順(再掲)

- [x] 1. monorepo構成にする
- [ ] 2. `sample`からフレームワークを呼び出せるようにする
- [ ] 3. `/pages`・`route.ts`を`sample`パッケージに移動する

:::

### パスを整理する

#### server.ts

**アンズ**

> では、気を取り直して先程疎通させた`pnpm myframework dev`コマンドのエラーを直していきましょう！
> と、その前に覚えてほしい呪文があります！
> 「`import.meta.dirname`はそのファイルのパス。`process.cwd()`は実行されてるパス。」です！

**シッポ**

> `import.meta.dirname`はそのファイルのパス。`process.cwd()`は実行されてるパス。
> ……これはなんの呪文？

**アンズ**

> ふふっ、唱えてくださってありがとうございます♪
> ちゃんと解説しますねっ！

| ケース                                | `import.meta.dirname` | `process.cwd()` |
| ------------------------------------- | --------------------- | --------------- |
| `app/hoge`から`app/src/main.js`を実行 | `app/src`             | `app/hoge`      |

> こうして例を見てみるとスッキリ理解できませんか？

**シッポ**

> あ、なるほど、そういう話か。最初はアンズちゃんがおかしくなったのかと思ったけど、今なら呪文の意味も理解できるよ。
> 「`import.meta.dirname`はそのファイルのパス。`process.cwd()`は実行されてるパス。」だね

**アンズ**

> わたしがおかしくなったと思ってたんですか！？
> こ、こほんっ。今は理解していただけてるならそれでいいですっ。

**シッポ**

> うん。今は理解できてるよ。

**アンズ**

> では、あらためてエラーを確認しますと……

```shell:実行結果(URLアクセス時)
>>> pnpm myframework dev

Server is running at http://localhost:5173
Error: ENOENT: no such file or directory, open '/.../packages/sample/index.html'
```

> 「index.htmlがないよ〜」ってエラーですよね？
> なので、index.htmlを読みに行っているコードを確認しにいくと……

```ts:myframework/src/server.ts(一部抜粋)
let template = fs.readFileSync(
    path.resolve(process.cwd(), "index.html"),
    "utf-8",
);
```

> これを見た上で、もう一度さっきの呪文を唱えて見てもらえますか？

**シッポ**

> 呪文？
> 「`import.meta.dirname`はそのファイルのパス。`process.cwd()`は実行されてるパス。」だったよね。
> あっ、そうか。

**アンズ**

> おっ。気が付きましたか！
> そうです！
> ここでは、`process.cwd()`を使ってますが、現在のmonorepo構成でこのファイルを実行指定しているパスはどこでしたっけ？

**シッポ**

> いま、実行しているパスは`packages/sample`。
> だから、`process.cwd()`は`packages/sample`と同じ意味になる。だけど、現在`packages/sample/index.html`は存在しないから、さっきのエラーになってるんだ！

**アンズ**

> はいっ！　もう完璧ですね！
> ではどう変えたらいいでしょうか！

**シッポ**

> `index.html`のパスは`packages/myframework/index.html`。これを示すようにしたい。
> 現在の状況だと、呪文のパスは以下の表みたいになる。
> つまり、`process.cwd()`じゃなくて、`import.meta.dirname`を使うようにすればいい？

| `import.meta.dirname`      | `process.cwd()`   |
| -------------------------- | ----------------- |
| `packages/myframework/src` | `packages/sample` |

**アンズ**

> いいですね！
> ここまで理解できてたらばっちりです！　早速修正に取り掛かっていきましょう！
> index.htmlの他にも修正しないといけない箇所があるのでついでに直してしまいますが、今のせんぱいなら解説はいらないと思います！

| 対象         | before                                      | after                                                  |
| ------------ | ------------------------------------------- | ------------------------------------------------------ |
| index.html   | `path.resolve(process.cwd(), "index.html")` | `path.resolve(import.meta.dirname, "../index.html")`   |
| entry-server | `"src/entry-server.tsx"`                    | `path.resolve(import.meta.dirname, "entry-server.js")` |

:::details 現時点でのserver.ts全文

```ts:myframework/src/server.ts
// server.ts
import fs from "node:fs";
import path from "node:path";
import express from "express";
import { createServer as createViteServer } from "vite";
import type { renderFunc } from "./entry-server.js";
import { createRequest, sendResponse } from "@remix-run/node-fetch-server";

async function createServer() {
  const app = express();

  // Viteをexpressサーバーのミドルウェアとして使います！
  // このミドルウェアは例えば以下のようなことをしてくれています
  // /publicの静的ファイル配信
  // .tsx->jsなどのトランスパイル
  const vite = await createViteServer({
    server: { middlewareMode: true },
    appType: "custom",
  });
  app.use(vite.middlewares);

  app.use(async (req, res, next) => {
    const url = req.originalUrl;

    try {
      // (1)expressサーバーがあらゆるリクエストに対して一律で`index.html`をエントリポイントとする
      let template = fs.readFileSync(
        path.resolve(import.meta.dirname, "../index.html"),
        "utf-8",
      );

      // ViteのHMRなどにつかうclientモジュールを突っ込んでくれています。ここは「おまじない」くらいの理解で大丈夫です！
      template = await vite.transformIndexHtml(url, template);

      // サーバーサイドのエントリポイントの読み込みです！
      //ssrLoadModule経由で呼び出すことでHMRが提供されたり、ブラウザ用コードと同じようにESMで書いたもの
      // Node環境で動くように変換してくれたりします。
      // .tsxも自動でトランスパイルしてくれます！
      const { render } = (await vite.ssrLoadModule(
        path.resolve(import.meta.dirname, "entry-server.js"),
      )) as {
        render: renderFunc;
      };

      // (2)`index.html`を`entry-server.tsx`をつかって、Reactコンポーネントが反映されたSSR済みのhtmlに変換する
      const request = createRequest(req, res);
      const routerResult = await render(request);
      if (routerResult instanceof Response) {
        // リダイレクト等であれば、そのまま返す
        sendResponse(res, routerResult);
        return;
      }
      const html = template.replace(`<!--ssr-outlet-->`, () => routerResult);

      // レスポンスします
      res.status(200).set({ "Content-Type": "text/html" }).end(html);
    } catch (e) {
      if (e instanceof Error) {
        vite.ssrFixStacktrace(e);
        next(e);
      }
    }
  });

  app.listen(5173, () => {
    console.log("Server is running at http://localhost:5173");
  });
}

export default createServer;


```

:::

#### index.html

**アンズ**

> これで無事に完成……と行きたいんですが、実は一箇所、厄介なパスがありまして、
> それが、`index.html`から参照している`entry-client.tsx`です。

```html:index.html
<!doctype html>
<html lang="ja">
  <head>
    <meta charset="UTF-8" />
  </head>
  <body>
    <div id="app"><!--ssr-outlet--></div>
    <script type="module" src="/src/entry-client.tsx"></script>
  </body>
</html>

```

**シッポ**

> この`src/entry-client.tsx`だね
> 厄介っていうのはどういうこと？

**アンズ**

> まずなんですが、この`index.html`に書いた`<script>`のjsファイルはどこにあることが想定されていますかね？

**シッポ**

> えっとこれは……、Viteミドルウェアが静的ファイルとして返すやつだから、実行されてるパスを基準に探しに行くはず。
> `packages/sample/src/entry-client.tsx`が想定されるパスかな？

**アンズ**

> 正解です！ つまり、`index.html`のパスではなく、プロセスのパスを基準に決まっているんですよね！

**シッポ**

> 呪文でいうと、Viteは`process.cwd()`を基準に探しに行くってことだね。

**アンズ**

> その通りです！
> なので、「`process.cwd()`ではなく、`import.meta.dirname`を使いましょう！」って言いたいところなんですが、htmlにそんな機能はないですよね。

**シッポ**

> あれ？　たしかに……。

**アンズ**

> これが、「厄介」と言った理由です！
> なにしろ、パスを記述しているのがhtmlファイルで、そのパスの解釈はViteが内部で済ませてしまうので、これまでのやり方で介入できないんですよ〜。

**シッポ**

> うわ〜。本当だ。
> html側で、「このパスはこういう風に解釈して」なんて伝える機能はないもんね。

**アンズ**

> そうなんです。なので、htmlで指定するパスではなく、Viteが行っているパスの解釈のほうをいじりましょう！
> それが、Vite-pluginsです！

**シッポ**

> あ、プラグイン。
> 聞いたことあったけど、こういう時に使うのか。

**アンズ**

> たぶん見たほうが早いので、さっそく実装していきます！

```ts:myframework/src/vite/plugins.ts(新規ファイル)
import path from "node:path";
import type { Plugin } from "vite";

const ENTRY_CLIENT_SOURCE = "@myframework/entry-client";
const ENTRY_CLIENT_ID = path.resolve(import.meta.dirname, "../entry-client.js");

export const entryClient: Plugin = {
  name: "myframework:entry-client",

  resolveId: (source: string) => {
    if (source == ENTRY_CLIENT_SOURCE) {
      return ENTRY_CLIENT_ID;
    }
  },
};

```

ここは「`@myframework/entry-client`っていうモジュールがあったら、`path.resolve(import.meta.dirname, "../entry-client.js")`のことだよ！」という指示になります。

そしてこれを、Viteインスタンスに渡してあげます！

```ts:myframework/src/server.ts
// ...(略)
import { entryClient } from "./vite/plugins.js";

// Viteインスタンスを作る段階でプラグインを指定してあげればOKです！
  const vite = await createViteServer({
    server: { middlewareMode: true },
    appType: "custom",
    plugins: [entryClient],
  });
```

最後に、先程の`resolveID`に合わせて`index.html`もそれに合わせて変えてあげます。
さっき、`const ENTRY_CLIENT_SOURCE = "@myframework/entry-client";`とした部分と対応している点に注目ですよ！

```html:myframework/index.html
<!doctype html>
<html lang="ja">
  <head>
    <meta charset="UTF-8" />
  </head>
  <body>
    <div id="app"><!--ssr-outlet--></div>
    <script type="module" src="@myframework/entry-client"></script>
  </body>
</html>

```

#### パスの整理の動作確認

**アンズ**

> ここまでできたら早速、実行し直してみましょう！
> フレームワーク側の再ビルドを忘れないようにしてくださいね！

```zsh:packages/myframeworkで実行
pnpm run compile
```

```zsh:packages/sampleで実行
pnpm myframework dev
```

> どうでしょうか！　動いてますかね？

**シッポ**

> えっと、フレームワーク側をビルドしなおして……。あ！　ちゃんと動くようになった！
> `/`・`/dog`・`/counter`の全部動いてるよ！

**アンズ**

> やりましたね！
> これで外から使えるフレームワークになりました！

:::message
SSR・ルーティングの機能を他のアプリが使える状態にする手順(再掲)

- [x] 1. monorepo構成にする
- [x] 2. `sample`からフレームワークを呼び出せるようにする
- [ ] 3. `/pages`・`route.ts`を`sample`パッケージに移動する

:::

## pages・route.tsをsampleパッケージに移動する

**アンズ**

> では、本記事最後の仕上げです
> `/pages`と`route.ts`を`sample`パッケージに移動していきましょう！

**シッポ**

> 最初に話した、「アプリの仕様変更にsampleの変更だけで対応できたら良い」を達成するためのやつだね。

**アンズ**

> そうです！
> これ、ここまでにやった知識が生きるところなんですけど、せんぱい、分かりますか？

**シッポ**

> えっと、「`import.meta.dirname`はそのファイルのパス。`process.cwd()`は実行されてるパス。」を使えばいいんだよね。
> だから移動した上で、`import route from './route.js'`じゃなくて、`process.cwd()`からimportすればいい……んだけど、import文に`process.cwd()`は書けない。
> あ、ここもVite Pluginsを書くの？

**アンズ**

> `process.cwd()`を使う。大正解です！
> もちろんViteプラグインを書いてもいいんですが、実は単純にパスを置き換えるだけなら、もっと簡単な方法があります。
> それが、Vite configの`resolve.alias`です！
> ちょっと見ててくださいね……

まず、`/pages`と`route.ts`を移動させちゃいます！　移動後のtreeコマンドを貼っておきますね！

```shell:packages/sampleで実行
tree -I "node_modules"
.
├── package.json
└── src
    ├── pages
    │   ├── Dog.tsx
    │   ├── Index.tsx
    │   └── counter.tsx
    └── route.ts
```

そして、`sample`パッケージ側にもreactなどのライブラリを追加します。

```json:packages/sample/package.json
{
  "name": "sample",
  "private": true,
  "dependencies": {
    "myframework": "workspace:*",
    "react": "^19.2.8",
    "react-dom": "^19.2.8",
    "react-router": "^8.2.0"
  },
  "devDependencies": {
    "@types/react": "^19.2.17",
    "@types/react-dom": "^19.2.3",
    "@types/react-router": "^5.1.20"
  }
}

```

追記したら、インストールを忘れずに！

```shell:packages/sampleで実行
pnpm install
```

で、ここから注目してほしいんですが、
こんな風に、Viteミドルウェアを作る段階でオプションを渡してあげると、Vite（や裏で動いているrollup）の挙動を少しだけ制御することができるんです！

```ts:myframework/src/server.ts
const vite = await createViteServer({
  server: { middlewareMode: true },
  appType: "custom",
  plugins: [entryClient],
  resolve: {
    alias: {
      "@myframework/route": path.resolve(process.cwd(), "src/route.ts"),
    },
  },
});
```

こうすると、`@myframework/route`をViteが`path.resolve(...)`として解釈してくれるようになるんです！
なので、この挙動に合わせてimport側も整えてあげましょう！

```tsx:myframework/src/entry-client.tsx
import route from "@myframework/route";
```

```tsx:myframework/src/entry-server.tsx
import route from "@myframework/route";
```

こうしてあげます！
これで、あとは先程の`resolve.alias`に従って、Viteが解釈してくれるようになりました！

……なんですが、このままだと、以下のエラーがでていますよね？

```shell
Cannot find module '@myframework/route' or its corresponding type declarations.
```

これは、TypeScriptはViteの`resolve.alias`を解釈しないので、「`@myframework/route`なんてファイルないよ！」と怒っている状態ですね。
なので、型定義ファイルを追加してあげます！

```ts:myframework/src/myframework.d.ts
declare module "@myframework/route" {
  import type { RouteObject } from "react-router";
  const routes: RouteObject[];
  export default routes;
}
```

これで、TypeScriptも静かになったはずです！

**アンズ**

> さて、どうでしょうか？
> ここまでできたら、もう一度実行してみてください！（フレームワークのコンパイルも忘れずに！）

**シッポ**

> きた！　うごいたよ！　……なるほど。フレームワークってこうなってるんだ。
> 一旦普通に機能を作ってから、いじらせたい設定ファイルとかだけを`process.cwd()`を使ったりして外に出す。

**アンズ**

> そうです！　そしてこの`src/route.ts`が、このフレームワークの『規約』になりました
> Next.js で`/pages`にファイルを置くと勝手にルーティングされる。
> myframeworkで`src/route.ts`にルート定義を書くとルーティングされる。
> 同じ性質のものを今作ったことになるんですよ！！

**シッポ**

> たしかに……
> いま、フレームワークの『規約』を作ったことになるんだ。

**アンズ**

> そうなんですよ。
> ここまで実装すると大分、フレームワークの魔法が解けてきたんじゃないですか？

**シッポ**

> うん。大分わかってきたかも。
> フレームワークの本質は`process.cwd()`だね。

**アンズ**

> それはさすがに言い過ぎだと思いますけど笑
> でも、フレームワークを支えている重要な技術であることは間違いないと思います！
> あ、そうだ。　手順書のチェックもつけておきますね。　これにて完了です！

:::message
SSR・ルーティングの機能を他のアプリが使える状態にする手順(再掲)

- [x] 1. monorepo構成にする
- [x] 2. `sample`からフレームワークを呼び出せるようにする
- [x] 3. `/pages`・`route.ts`を`sample`パッケージに移動する

:::

## 終わりに

**アンズ**

> さて、これでもう大分、フレームワークが形になってきましたね。
> せんぱいも、かなり理解が深まったんじゃないでしょうか。
> こういう風にブラックボックスの中を見れるのは、やっぱりスクラッチ実装の醍醐味ですよね！

**シッポ**

> うん。もう魔法ではなくなってきたよ。
> 専用のcliとか、起動方法(今回は`myframework dev`コマンド)を用意しておいて、アプリ側でいじれる項目だけをなんとかして外に出す。
> 多分言語とかレイヤ・ライブラリが違っても、フレームワークってこんな風にできてるんだろうなって想像ができるようになったよ。

**アンズ**

> もうここで完成にしてもいいんですが、せっかくなので最後までやりきります！
> 次回は、`myframework dev`コマンドの他に、本番環境用のコマンドを作っていきますよ！
