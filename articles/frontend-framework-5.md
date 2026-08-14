---
title: "【本番ビルド編】後輩ちゃんと学ぶ自作フレームワーク！！ ビルドのエントリポイントは2つある"
emoji: "🚀"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vite", "react", "ssr", "express", "自作フレームワーク"]
published: true
---

:::message
この記事の目的
普段はNext.jsなどのフレームワークを使って開発しているエンジニアが、内部を理解するためにフレームワークを自作してみる記事です。
執筆時点で筆者が公開しているフレームワークはこちらにあります
https://github.com/AN-Sippo/sippa

**全5回シリーズの最終回（第5回）です。**

- [第1回：基礎編 — Vite×React Router](https://zenn.dev/sippooo/articles/frontend-framework-1)
- [第2回：SSR編](https://zenn.dev/sippooo/articles/frontend-framework-2)
- [第3回：React Router編 — Hydration failedの正体とloaderの仕組み](https://zenn.dev/sippooo/articles/frontend-framework-3)
- [第4回：monorepo編 — binフィールドとprocess.cwd()で『規約』をつくる](https://zenn.dev/sippooo/articles/frontend-framework-4)
- 第5回：本番ビルド編 — ビルドのエントリポイントは2つある（この記事）

:::

## 登場人物たち

| 見た目 　                            | 名前   | プロフィール                                            |
| ------------------------------------ | ------ | ------------------------------------------------------- |
| ![](/images/general/Sippo.png =100x) | シッポ | 筆者。Webフロントエンドが少しだけ分かる。               |
| ![](/images/general/anzu.png =100x)  | アンズ | Sippoの後輩。技術に詳しい。とりあえずなんでも知っている |

## 本番ビルドって学ぶ必要ある？(この記事の目標)

**アンズ**

> 前回までの記事で、 ViteとReact-Routerを組み合わせたフレームワークを作って、monorepo構成で動作させるところまでできました。
> 今回は本当に最後の仕上げ。
> 「本番ビルド」をできるようにしていきます！

**シッポ**

> ……

**アンズ**

> どうしました。せんぱい？
> なんか反応微妙じゃないです？

**シッポ**

> うん、なんというか。前回まででmonorepoにして、フレームワークを動作させるところまでできたわけじゃん。
> もうそれでほとんど完成というか。フレームワークを理解する目的に照らしたとき、本番ビルドってなんだか「本質じゃない」ような気がして。

**アンズ**

> なるほど。言いたいことはわかりますよ。
> もうフレームワークは動作しているわけですし、なにか機能を足すわけでもないですからね。

**シッポ**

> そうなんだよ。
> だから、あんまりやる気がでないんだ。

**アンズ**

> せんぱい、Webアプリを作っていて、一番めんどくさい瞬間っていつですか？

**シッポ**

> 突然だね。……そうだなあ。
> 環境構築・理解不能なバグ・CSSをガチャガチャ・デプロイとか。……あ！

**アンズ**

> ふふっ。気が付かれましたか。
> そうです。日頃の開発で知識が手薄になりがちなデプロイです。
> もちろんクラウドの知識も必要ですが、今回の本番ビルドの知識も必要になります。

**シッポ**

> 業務ならCI/CDが組まれているし、個人開発なら手作業だけど、それでもせいぜい数回程度。
> 深く学ばなくても、その場しのぎでなんとかなってしまうから、いっつもなんとなくでやり過ごしちゃうんだよね。

**アンズ**

> ……そろそろ理解したくないですか？

**シッポ**

> ……したいです。

**アンズ**

> うんうん。というわけで、今回は本番ビルド編です！

**シッポ**

> 負けた気がする。

## 本番ビルドってなに？

**アンズ**

> というわけなんですけども

**シッポ**

> なんですけども？

**アンズ**

> まず目指すべきゴールをはっきりさせておきましょう。
> そもそも、フロントエンドにおける本番ビルドってどういう状態か、わかりますか？

**シッポ**

> え？　うーん。
> `pnpm myframework build`を叩くと、`out.js`みたいなファイルが1つだけできて、本番サーバーにこのjsファイルを1つ置くだけで動く！
> みたいな？

**アンズ**

> そういうイメージですよね。
> 実はフロントエンドでは、ブラウザに配信する関係もあって、そこまで1つのファイルにまとめる方法は稀だったりします！

**シッポ**

> たしかに普段ビルドしたときも、なんかよく分からない構成で大量のファイルが生成されていた気が……。

**アンズ**

> そうですね笑　正解を言ってしまうと、
> 本番関係のサーバーで、`pnpm myframework build`→`pnpm myframework start`を叩いて、それで動作すればいいんです！

**シッポ**

> あ、え？　そうなの？
> でも、Next.jsだともうちょっとミニマムに持ち運べる構成になってなかった？

**アンズ**

> Next.jsのstandaloneモードですかね？
> 最近は、先程せんぱいが言ったような形に近いものも出てきているので、そっちを触ったのかもしれませんね。
> 「そういう形を目指すべきだよね」という流れがあるのも事実なんですが、まだ現状では本番サーバーで`npm`を叩くのが王道なんです！

**シッポ**

> 意外とそんなものなんだ。

**アンズ**

> はい！　なので、本番のDockerイメージをビルドするたびに`npm i`させている構成とか、普通に見かけますよ！

## buildコマンドをつくる

**アンズ**

> と、遅くなりました。いよいよコードを書く実装に入っていきますよっ。
> 先程、「本番関係のサーバーで、`pnpm myframework build`→`pnpm myframework start`を叩いて、それで動作すればいい」と言いました。
> それを既存のコードに追加すると……

```ts:packages/myframework/src/cli.ts
#!/usr/bin/env node
// importは一旦省略します

const dev = () => {
  createServer();
};

const start = () => {
    // TODO: ビルド結果を参照して本番サーバーを起動する
};

const build = async () => {
    // TODO: ビルドする
};

switch (cmd) {
  case "dev":
    dev();
    break;
  case "start":
    start();
    break;
  case "build":
    build();
    break;
  default:
    console.error("Invalid command");
    process.exit(1);
}

```

> こうなりますよね？
> 最終的にこの２つのコマンドが動作すればいいわけです。

**シッポ**

> ここまではそうだね。
> start,buildコマンドのガワをつくっただけだもんね。

**アンズ**

> そうですね。そしたらここからbuild関数の中身を実装していきます。
> ところで、これまでは開発サーバーで動かしていたわけですけど、TypeScriptからJavaScriptへのビルドなどは必要だったはずですよね？
> これは誰がやってくれていましたか？

**シッポ**

> それは、`server.ts`に書いた`viteDevServer`とか、`vite.ssrLoadModule`が中でやってくれていたんだよね？

**アンズ**

> そうです！　よく覚えていますね。Viteがやってくれていました。
> 同じように、本番ビルド用の関数もViteが用意してくれているんです！

```ts:packages/myframework/src/cli.ts
#!/usr/bin/env node
import { build as buildVite } from "vite";

// 省略

const build = async () => {
  await buildVite(...);
};


```

> こんなイメージですね。
> で、使い方なんですけど、ビルドするときってエントリポイントを指定しないといけないんですよ。

**シッポ**

> エントリポイント？

**アンズ**

> エントリポイントというのは、「ここから辿れるファイルを全部ビルドしてね」という、ビルドの起点となるファイルや関数のことですよ！
> たとえば、今作っているmyframeworkなら、`index.html`をエントリポイントにしてビルドすれば、`<script>`から`entry-client.js`が辿れて、そこから`route.ts`、そこからアプリケーションコードのように辿れることができます！

**シッポ**

> あ、index.htmlがエントリポイントなのか。
> `entry-client`・`entry-server`の２つがあるから、この２つがエントリポイントなのかと思ったよ。
> 「エントリー」って書いてあるじゃん？

**アンズ**

> なるほど……！　実は、せんぱいの言っていることも半分は正しいです！
> entry-clientもたしかにエントリポイントっぽいんですが、一旦`index.html`を見直してみましょうか。

```html:packages/myframework/index.html
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

> ここで`<script>`に書いている`@myframework`のパス。これ、Viteの機能を使ったエイリアスですよね？
> なので、index.htmlもビルドしてあげないといけない。だから、`entry-client`ではなく、その１つ上流の`index.html`をエントリポイントに指定するんです！

**シッポ**

> あー、なるほど。
> `index.html`でも、Viteの機能を使っている……というか、「`index.html`で使える機能を提供しているから、そこをエントリポイントにしてね」っていうのがViteの思想なんだ

**アンズ**

> いい理解だと思います！

**シッポ**

> あれ？　でもこれって足りてる？

**アンズ**

> 足りてるっていうと、なんのことでしょう？

**シッポ**

> 「ここを起点に辿れるものをビルドしてね」がエントリポイントなんだよね？
> そしたら、`index.html`だけだと、`entry-server`から辿る方の経路が使われていないように思える

**アンズ**

> ……！　そうです！　その通りです！
> なので実は、このフレームワークでは**ビルドのエントリポイントが２つある**んですよ！
> さっきのコードに追加すると、こういうことです！

```ts:packages/myframework/src/cli.ts
#!/usr/bin/env node
import { build as buildVite } from "vite";

// 省略

const build = async () => {

  // server
  await buildVite(
    // TODO: entry-serverをエントリポイントにしてビルドする
  );

  // client
  await buildVite(
    // TODO: index.htmlをエントリポイントにしてビルドする
  );
};

```

**シッポ**

> そうだね。`index.html`を基準にしたクライアント用のビルドと、`entry-server`を基準にしたサーバー用のビルドがある。

**アンズ**

> そうです！　ではさっそくビルド関数の中身を書いていきたいんですが、
> viteのbuild関数において、エントリポイントのデフォルト値は「アプリケーションのルートにあるindex.html」です！

**シッポ**

> 「`index.html`をエントリポイントにするのがViteの思想」だから、デフォルト値になってるんだね。

**アンズ**

> 予想ですが、それで合ってると思います！
> それで、SSR用に`.js`ファイルをエントリポイントにしたいときは、`build.ssr`に指定してあげる必要があります。
> つまり、

```ts:packages/myframework/src/cli.ts
#!/usr/bin/env node
import { build as buildVite } from "vite";

// 省略


const build = async () => {
  // 実行時、このファイルは「myframework/dist/cli.js」なので、`root = packages/myframework`と等価です！
  const root: string = path.resolve(import.meta.dirname, "..");

  // server
  await buildVite({
    root: root, // 「アプリケーションのルート」を指定する。これで、myframework/index.htmlがエントリポイントになります！
    build: {
      ssr: path.resolve(import.meta.dirname, "./entry-server.js"), // SSRのエントリポイント
    },
  });

  // client
  await buildVite({
    root: root,
  });
};

```

> こんな感じになりますよね！

**シッポ**

> うん。わかる。
> サーバー側の方では、SSR用に`.js`をエントリポイントにするから`build.ssr`を指定して、
> クライアントの方では、`packages/myframework/index.html`がエントリポイントだから、デフォルト値のまま。

**アンズ**

> はいっ！　そうです！
> ここまでが、build時の要点になります。
> 他にもいくつかオプションを指定しないといけないんですが、それに関しては説明を載せておくので、興味があったら読んでみてください！

**シッポ**

> 流し読みくらいでいいかな？

```ts:packages/myframework/src/cli.ts(全文)
#!/usr/bin/env node
import createServer from "./server.js";
import { build as buildVite } from "vite";
import path from "node:path";
import { entryClient } from "./vite/plugins.js";
const cmd = process.argv[2];

const dev = () => {
  createServer();
};

const start = () => {
    // TODO: ビルド結果を参照して本番サーバーを起動する
};

const build = async () => {
  console.log("本番ビルドします");
  const root: string = path.resolve(import.meta.dirname, "..");

  // server
  await buildVite({
    root: root,
    base: "/",
    plugins: [entryClient],
    resolve: {
      alias: {
        "@myframework/route": path.resolve(process.cwd(), "src/route.ts"),
      },
    },
    build: {
      outDir: path.resolve(process.cwd(), "dist/server"),
      emptyOutDir: true,
      ssr: path.resolve(import.meta.dirname, "./entry-server.js"),
    },
  });

  // client
  await buildVite({
    root: root,
    base: "/",
    plugins: [entryClient],
    resolve: {
      alias: {
        "@myframework/route": path.resolve(process.cwd(), "src/route.ts"),
      },
    },
    build: {
      outDir: path.resolve(process.cwd(), "dist/client"),
      emptyOutDir: true,
    },
  });
};

switch (cmd) {
  case "dev":
    dev();
    break;
  case "start":
    start();
    break;
  case "build":
    build();
    break;
  default:
    console.error("Invalid command");
    process.exit(1);
}

```

:::details その他のオプションについて

| オプション          | 機能                                                                                             |
| ------------------- | ------------------------------------------------------------------------------------------------ |
| `base`              | このアプリがどのパスに展開されるか。「`/users`配下のみこのアプリが担当する」のようなケースで使う |
| `plugins`           | `server.ts`で使ったものと同じ。今回は、自作したVite pluginsを入れるために使っている              |
| `resolve.alias`     | `server.ts`で使ったものと同じ。ファイルパスを置換するために使っている                            |
| `build.outDir`      | ビルド結果を出力するフォルダを指定する                                                           |
| `build.emptyOutDir` | `build.outDir`が`root`の外にあるときは、ここを`true`にする必要がある                             |

:::

**アンズ**

> こんな感じになります！

**シッポ**

> ちょっと思ったより行数が多くてびっくりしたけど、うん。
> よく見るとさっきの要点から大きくは変わってないね。
> `entry-server`由来のビルドは`dist/server`に・`index.html`由来のビルドが`dist/client`に生成されるんだね。

**アンズ**

> ですね！　
> `dist/server`は主にサーバー側において、`server.js`から読まれて使われます。
> 一方、`dist/client`はブラウザからのアクセスに対してレスポンスとして返されるファイルとなることが多いですよ！

## startコマンドをつくる

**シッポ**

> 今のコードみて思いだしたけど、そういえば`start`コマンドはまだ実装してないんだったね。

**アンズ**

> あっ！　そうですね！
> そうなんですが、`start`は要するに`dev`と同じで、サーバーを起動するだけなので、1行でいいです！

```ts:packages/myframework/src/cli.ts
const start = () => {
  createServer(true);
};
```

> `dev`と同じく`createServer`を呼ぶだけにしています！
> ただ、本番サーバーであることを伝えるために、フラグを`true`にして渡しています。

**シッポ**

> `createServer`側で「本番か？」のフラグを受けるようになっていないから、いまのままだとTypeScriptが怒ってるんだけど、
> これは今から追加するやつかな？

**アンズ**

> そうですそうです！
> すみません。言い忘れていました💦
> 一旦TypeScript君は怒らせたままで大丈夫です！

**シッポ**

> TypeScriptは怒らせたままね。
> 了解です！

:::details cli.ts(全文)

```ts:packages/myframework/src/cli.ts
#!/usr/bin/env node
import createServer from "./server.js";
import { build as buildVite } from "vite";
import path from "node:path";
import { entryClient } from "./vite/plugins.js";
const cmd = process.argv[2];

const dev = () => {
  createServer();
};

const start = () => {
  createServer(true);
};

const build = async () => {
  console.log("本番ビルドします");
  const root: string = path.resolve(import.meta.dirname, "..");

  // server
  await buildVite({
    root: root,
    base: "/",
    plugins: [entryClient],
    resolve: {
      alias: {
        "@myframework/route": path.resolve(process.cwd(), "src/route.ts"),
      },
    },
    build: {
      outDir: path.resolve(process.cwd(), "dist/server"),
      emptyOutDir: true,
      ssr: path.resolve(import.meta.dirname, "./entry-server.js"),
    },
  });

  // client
  await buildVite({
    root: root,
    base: "/",
    plugins: [entryClient],
    resolve: {
      alias: {
        "@myframework/route": path.resolve(process.cwd(), "src/route.ts"),
      },
    },
    build: {
      outDir: path.resolve(process.cwd(), "dist/client"),
      emptyOutDir: true,
    },
  });
};

switch (cmd) {
  case "dev":
    dev();
    break;
  case "start":
    start();
    break;
  case "build":
    build();
    break;
  default:
    console.error("Invalid command");
    process.exit(1);
}

```

:::

## server.tsを本番サーバーに対応させる

### 方針を考える

**アンズ**

> それで、ここから`createServer`側も合わせて修正していきます！

**シッポ**

> 「本番サーバーである」フラグを受けるようにするんだね

**アンズ**

> そうですね！
> わりと中身がまるっと違うので、個人的に関数ごと分けてもいいのかなって思ったんですけど、
> Viteのドキュメントが同じ関数の`if-else`で処理していたので、それに合わせてみました。

**シッポ**

> まずはドキュメントに忠実にね。

**アンズ**

> なので、createServerの定義はこんな感じになります！

```ts:packages/myframework/src/server.ts
async function createServer(isProduction?: boolean) {...}
```

**シッポ**

> そうだね。引数でフラグを受けつつ、引数なしでの呼び出しを壊さないようにオプショナルにしてる。

**アンズ**

> それで、本番サーバー用にフラグを作るくらいなので、本番サーバーと開発サーバーではやってほしいことが違うわけなんですけど、
> 実際、どこが違うのか、分かりますか？

**シッポ**

> うーん……？

**アンズ**

> そうですねえ。じゃあ、聞き方を変えてみましょうか。
> 開発サーバーで使っていた機能のうち、**本番サーバーで要らない機能はどれでしょうか？**

**シッポ**

> 要らない機能……。
> あ、HMRとか！　本番でコードを修正することはないから要らないよね。

**アンズ**

> その通りです！　HMRも要らないです！
> 他にも、Viteが自動でやってくれている`tsx`や`ts`のトランスパイルも要らないですよね！
> 本番なら事前にビルドしているので、すでにブラウザが解釈できるjsファイルになっているはずです。

**シッポ**

> あ、そうか。たしかにトランスパイルもいらないね。

**アンズ**

> はい！　それで、現状の開発サーバーでそういう機能を使っている部分をまとめると以下の表になります！

| 開発サーバー         | 機能                                    | 本番で不要になる機能 | 本番サーバー     |
| -------------------- | --------------------------------------- | -------------------- | ---------------- |
| `viteDevServer`      | 静的ファイル配信・HMR・トランスパイル   | HMR・トランスパイル  | `express.static` |
| `transformIndexHtml` | HMR                                     | HMR                  | 削除             |
| `ssrLoadModule`      | モジュール読み込み・トランスパイル・HMR | HMR・トランスパイル  | `import`         |

**シッポ**

> ほう。なるほどねえ。HMR・トランスパイルを担当していた部分を片っ端から置き換えて行く感じなんだね。

**アンズ**

> そうなります！

### viteDevServerの置き換え

**シッポ**

> ところで……。`express.static`ってなに？

**アンズ**

> あ、説明したことないですもんね。本筋とはあんまり関係ないので、深くは理解しなくて良いんですが、
> これは、expressサーバーで動くファイル配信するためのやつです！
> リクエストされたものと、同じファイル名のものがあるかを探して、見つかったらそれを返してくれます！

**シッポ**

> 一致するファイルを返すから「静的ファイル配信」なんだ。
> でもこれって、React-Routerで作った部分とは違うの？

**アンズ**

> React-Routerを使ってこれまで作っていたのが、`/Counter`・`/Dog`みたいなルートを扱うものだとすると、
> `counter-438fsadf.js`・`dog-83dfs78.css`みたいな具体的なファイルを扱うのがexpress.staticです！
> 今回は構成の関係であんまり活躍しないんですが、実際にアプリを作るとすごくお世話になると思いますよ！

**シッポ**

> そっか。なるほどね。
> じゃあ、その静的ファイルとして配信するのは、どのファイルになるの？

**アンズ**

> それが、先程ビルドした`dist/client`なんです！

**シッポ**

> あっ。そうなるのか。
> `index.html`から`entry-client`→`route.ts`から辿れるファイル、つまりアプリケーションのほぼ全てのファイルが`dist/client`に含まれていて、
> それをブラウザからのリクエストに応じて返す。

**アンズ**

> そうなりますね！
> なのでSSRが必要ないなら、`dist/client`だけでも十分なんですよっ。

**シッポ**

> なるほどなあ。`dist/client`を静的ファイルとして配信すれば良いんだ。
> さっきの３箇所を変更したら完成になる？

**アンズ**

> そうですね！　一旦そうしていいと思います！
> じゃあ、せんぱい最後の仕上げ、やってもらっていいですか？

**シッポ**

> わかった！
> まずは1つ目。`viteDevServer`を`express.static`と出し分けるようにすればいいから……

```ts:packages/myframework/src/server.ts(一部抜粋。最後に全文載せます)
  let vite: ViteDevServer | undefined;
  if (isProduction) {
    app.use(
      "/",
      express.static(path.resolve(productionDistPath, "client"), {
        index: false, // 「/」へのアクセスに対して自動で`index.html`を返すか？のオプション。自前のハンドラで処理したいのでfalseとする。
      }),
    );
  } else {
    vite = await createViteServer({
      server: { middlewareMode: true },
      appType: "custom",
      plugins: [entryClient],
      resolve: {
        alias: {
          "@myframework/route": path.resolve(process.cwd(), "src/route.ts"),
        },
      },
    });
    app.use(vite.middlewares);
  }
```

**シッポ**

> こうなるよね。

**アンズ**

> ……そうですね！　いいと思います！

### `transformIndexHtml`と`ssrLoadModule`の置き換え

**アンズ**

> で、次が`transformIndexHtml`と`ssrLoadModule`です！　この２つはまとめてやっちゃうのが良さそうかもしれません。

**シッポ**

> ……あ、ここは、
>
> 1. fs.readFile()でindex.htmlを取ってくる
> 2. `transformIndexHtml`を呼ぶ
> 3. `ssrLoadModule`で`render`関数を取ってきて、呼ぶ
>
> この３つの処理がまとまってるんだね。

**アンズ**

> そうなっていますね！
> ここも色々と整理はできそうですが一旦、「index.htmlを取ってきて、必要なら`transformIndexHtml`を呼んでから返す。`render`をimportして返す。」
> 部分を関数としてまとめちゃうのが良いかもしれません。

```ts
type RenderTemplate = {
  template: string;
  render: renderFunc;
};
```

> こんな型を返す関数を、Develop用とProduction用でそれぞれ用意するイメージです！

**シッポ**

> さっきの1-3の一連の処理のうち、共通する部分をなるべく外に出して、違う部分だけ関数にまとめる意識だね

**アンズ**

> その通りです！　では最後にこの2つの関数を実装していきます。
> まず私が、これまでのコードを移植する形で、Develop用のコードを書いて見せますね！

**シッポ**

> 了解です！

```ts:packages/myframework/src/server.ts
type RenderTemplate = {
  template: string;
  render: renderFunc;
};

const importDevelop = async ({
  vite,
  req,
}: {
  vite: ViteDevServer; // Viteの機能が必要だったので一旦引数として受けることにした
  req: Request;
}): Promise<RenderTemplate> => {
  let template = fs.readFileSync(
    path.resolve(import.meta.dirname, "../index.html"),
    "utf-8",
  );
  template = await vite.transformIndexHtml(req.url, template);
  const render = (
    await vite.ssrLoadModule(
      path.resolve(import.meta.dirname, "entry-server.js"),
    )
  ).render;

  return { template, render };
};
```

**シッポ**

> ふむふむ。
> 色々と引数は受けて、中では色々してるけど、最終的にhtmlの文字列と`render`を返せばいい、と。

**アンズ**

> そうです！
> Production用だと引数は要らなくなる気がするので、返り値さえ合っていれば大丈夫です！
> じゃあせんぱい、Production用のほうを書いてもらっていいですか？

**シッポ**

> わかった！
> ……ってあれ？　ここで読み込むhtmlファイルは、ビルド後のやつ？

**アンズ**

> よく気が付きました！
> エイリアスといった「Viteの機能」をhtmlファイルでも使っているので、そういった機能の適用が終わっているもの。つまり、ビルド後のhtmlファイルを持ってくる必要があります！

**シッポ**

> そっか！　だから、`packages/sample/dist/client`を参照するようにして……

```ts:packages/myframework/src/server.ts
const productionDistPath: string = path.resolve(process.cwd(), "dist");

const importProduction = async (): Promise<RenderTemplate> => {

  // renderはただのimportにする
  const render = (
    await import(path.resolve(productionDistPath, "server/entry-server.js"))
  ).render;

  const template = fs.readFileSync(
    path.resolve(productionDistPath, "client/index.html"),
    "utf-8",
  );
  return { render, template };
};

```

**シッポ**

> これでどうだろう？

**アンズ**

> ふむ……。はいっ！　いいと思います。ばっちりですね！
> 一旦ここまでで、コードの全文を載せて整理しておきましょうか！

:::details server.ts(全文)

```ts:packages/myframework/src/server.ts
// server.ts
import fs from "node:fs";
import path from "node:path";
import express from "express";
import { createServer as createViteServer, ViteDevServer } from "vite";
import { renderFunc } from "./entry-server.js";
import { createRequest, sendResponse } from "@remix-run/node-fetch-server";
import { entryClient } from "./vite/plugins.js";

const productionDistPath: string = path.resolve(process.cwd(), "dist");
const PORT = process.env.PORT ?? 5173;

type RenderTemplate = {
  template: string;
  render: renderFunc;
};

const importDevelop = async ({
  vite,
  req,
}: {
  vite: ViteDevServer;
  req: Request;
}): Promise<RenderTemplate> => {
  let template = fs.readFileSync(
    path.resolve(import.meta.dirname, "../index.html"),
    "utf-8",
  );
  template = await vite.transformIndexHtml(req.url, template);
  const render = (
    await vite.ssrLoadModule(
      path.resolve(import.meta.dirname, "entry-server.js"),
    )
  ).render;

  return { template, render };
};

const importProduction = async (): Promise<RenderTemplate> => {
  const render = (
    await import(path.resolve(productionDistPath, "server/entry-server.js"))
  ).render;
  const template = fs.readFileSync(
    path.resolve(productionDistPath, "client/index.html"),
    "utf-8",
  );
  return { render, template };
};

async function createServer(isProduction?: boolean) {
  const app = express();
  let vite: ViteDevServer | undefined;
  if (isProduction) {
    app.use(
      "/",
      express.static(path.resolve(productionDistPath, "client"), {
        index: false,
      }),
    );
  } else {
    vite = await createViteServer({
      server: { middlewareMode: true },
      appType: "custom",
      plugins: [entryClient],
      resolve: {
        alias: {
          "@myframework/route": path.resolve(process.cwd(), "src/route.ts"),
        },
      },
    });
    app.use(vite.middlewares);
  }

  app.use(async (req, res, next) => {
    try {
      let template: string;
      let render: renderFunc;
      const request = createRequest(req, res);

      if (isProduction) {
        ({ template, render } = await importProduction());
      } else {
        ({ template, render } = await importDevelop({
          vite: vite!,
          req: request,
        }));
      }

      const routerResult = await render(request);

      // リダイレクト等であれば、そのまま返す
      if (routerResult instanceof Response) {
        sendResponse(res, routerResult);
        return;
      }

      const html = template.replace(`<!--ssr-outlet-->`, () => routerResult);

      // レスポンスします
      res.status(200).set({ "Content-Type": "text/html" }).end(html);
    } catch (e) {
      if (e instanceof Error) {
        if (vite) {
          vite.ssrFixStacktrace(e);
        }
        next(e);
      }
    }
  });

  app.listen(PORT, () => {
    console.log(`Server is running at http://localhost:${PORT}`);
  });
}

export default createServer;

```

:::

## 動作確認

**アンズ**

> と、これで完全に動作するようになったはずです！
> 試してみてもらえますか？

```zsh:packages/myframeworkで実行
pnpm run compile
```

```zsh:packages/sampleで実行
pnpm myframework build
pnpm myframework start
```

> ビルドしたら、sampleパッケージにはこんな感じの成果物が作成されているはずです！

![distフォルダのスクショ](/images/frontend-framework-5/dist.png)

**シッポ**

> えっと、`sample`パッケージに移動して、ビルドしてから……。
> おっ！　すごい！　これまでと同じように動作してるよ！

**アンズ**

> やりました！
> 世にある多くのフレームワークも、これと全く同じ手順で本番サーバーを立てることができるんです！

**シッポ**

> Next.jsのstandaloneビルドとかはあれど、こうやって起動するのがまだ王道なんだったね。

**アンズ**

> はいっ！　ここまで自力で走りきれたせんぱいなら、フレームワークを使って開発する時も、理解スピードが段違いになると思います！

## 自作フレームワークを終えて

**アンズ**

> どうでしょう？　ここまで5回にわたってフレームワークをスクラッチで作ってきましたけど。

**シッポ**

> やっぱり一番大きいのは、ブラックボックスを潰せたことかな。
> `/pages`にファイルを置くとルーティングされるとか、npmのパッケージとして動作させるとか、やってみて初めて実感が湧いたよ。

**アンズ**

> こういう「調べれば使えるけど、深くは理解してない」領域を理解できるのは、スクラッチ実装の醍醐味ですよね！

**シッポ**

> 最初は、「フレームワークを作る」なんて想像もつかなかったけど、やれるものなんだね。

**アンズ**

> ViteとかReact Routerのサポートを受けられるのも大きいですよね！
> Next.jsとかだとここの中身も自前で持ってるので、コード量もすごいことになってそうだなあって思ってます笑

**シッポ**

> そっか。スクラッチ実装といっても、express, React Router, Viteとかまだまだ色んなライブラリに依存してるもんね。

**アンズ**

> そうなんですよね。
> これをさらに掘って、ルーターライブラリから自作するのもアリではあるんですが、今回はあくまでも「全体像を把握する」ことを目的としているので、遠慮なく使わせてもらいましたよっ！

**シッポ**

> うん。最初からルーターまで自作したらもう頭追いつかないかも笑

**アンズ**

> さて、これで本当に自作フレームワークの完成になります！
> せんぱい、本当にここまでお疲れ様でした〜！
