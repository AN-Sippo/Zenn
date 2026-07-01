---
title: "【SSR編】後輩ちゃんと学ぶ自作フレームワーク！！(Vite×React-Router)"
emoji: "🐈️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react", "vite", "ssr", "自作フレームワーク", "フルスクラッチ"]
published: true
---

:::message
この記事の目的
普段はNext.jsなどのフレームワークを使って開発しているエンジニアが、内部を理解するためにフレームワークを自作してみる記事です。
執筆時点で筆者が公開しているフレームワークはこちらにあります
https://github.com/AN-Sippo/sippa

この記事は第二回です。前回記事は[こちら](https://zenn.dev/sippooo/articles/frontend-framework-1)

:::

## 登場人物たち

| 見た目 　                            | 名前   | プロフィール                                            |
| ------------------------------------ | ------ | ------------------------------------------------------- |
| ![](/images/general/Sippo.png =100x) | シッポ | 筆者。Webフロントエンドが少しだけ分かる。               |
| ![](/images/general/anzu.png =100x)  | アンズ | Sippoの後輩。技術に詳しい。とりあえずなんでも知っている |

🐱 **アンズ**

> さて、せんぱい。続きやりますよ！前回はVite×React×TypeScriptの構成でカウンターアプリまで動作確認できましたよね！
> 次、なにやるかわかりますか？

🐶 **シッポ**

> ええ？うーん
> あ、ルーティングだ！React Router入れるんでしょ？

🐱 **アンズ**

> 惜しいです！それもやりたいですが、1つ先です。
> 実は、React Routerの前にSSR(server side rendering)を先に設定するのがおすすめなんです！
> サーバー側で使うルーターとブラウザで使うルーター、React Routerではそれぞれ別のものを使う必要があるので、SSRの設定を済ませてから、ルーティング周りはまとめてやっちゃいましょう！

🐶 **シッポ**

> ……？　そういうものなんだ？

🐱 **アンズ**

> 今はピンとこなくても実装したらきっとわかりますよ！
> ちょうどいいタイミングなので、これからの手順を整理しますね！

:::message
自作フレームワークまでの道順

- [x] Viteの導入
- [x] TypeScript,Reactの設定
- [ ] SSR(server side rendering)
- [ ] React Router(router,loader)の導入
- [ ] Monorepoで管理する

:::

ここまで全部できたら、きっとせんぱいもフレームワークの内側についてもっと詳しくなっているはずですよっ！

## SSR時の処理フロー

🐱 **アンズ**

> なので、まずはSSRを実装していきましょう！
> ここからちょっとややこしくなってくるので、ゆっくり整理しながら進めていきます！
> まずせんぱい、SSRについて説明できますか？

🐶 **シッポ**

> サーバーサイドレンダリング。
> ブラウザからリクエストがきた時点で、サーバー側でReact→htmlに変換(レンダリング)して、それをレスポンスとして返す。
> そういう仕組みだよね？

🐱 **アンズ**

> そうですね。あってます！
> じゃあ、React+React Router、つまり今回の構成で以下のようなアクセスをされたとき、どんな処理フローになるかわかりますか？
> せんぱいのアプリが`http://example.com/`で動作していると考えてください

1. 検索エンジンなどから`http://example.com/`にアクセスしてくる
2. そして、`http://example.com/`から`react-router-dom/Link`コンポーネントを踏んで、`http://example.com/counter`にページ内遷移してくる

🐶 **シッポ**

> え？これなら、
>
> 1. `/`にマッチする`Index.tsx`をSSRして返却
> 2. `/counter`にマッチする`Counter.tsx`をSSRして返却
>    じゃないの？

🐱 **アンズ**

> ふふ。引っかかりましたねせんぱい！
> 誰もが最初はそう思うんですが、正しくはこうなんです！

:::message
SSR時の正しい処理フロー

1. 外部から遷移してきたとき
   - SSRして返却する
2. `react-router/Link`などで内部遷移してきたとき
   - 足りない`js`ファイルだけをリクエストして、CSR(client side rendering)する

:::

🐶 **シッポ**

> そうなの！？
> じゃあ、実はSSRって結構出番少ない……？

🐱 **アンズ**

> そうなんですよ。
> もちろんサーバーでの実装次第なので違うようにもできるんですが、こうするのが現在の主流になっています！

🐶 **シッポ**

> でもなんで？全部のサーバーでやってあげたほうが良さそうな気がするんだけど

🐱 **アンズ**

> おっ、いい質問ですね！
> それは、なるべく高速に遷移させるためです！
> 今回の例で言うと、`Index.tsx`と`Counter.tsx`の両方からすごく大きいコンポーネント`<Hoge/>`を使っていたとしましょう。
> こんなイメージですよっ！

```tsx:Index.tsx

export default function Index() {
  return <Hoge />;
}
```

```tsx:Counter.tsx

export default function Counter() {
  return <Hoge />;
}
```

🐶 **シッポ**

> ほうほう。`components/Hoge.tsx`みたいのがあるイメージだね

🐱 **アンズ**

> これ、JavaScriptとして`Hoge.js`をリクエストする分にはブラウザにキャッシュがあるので、２回リクエストしても通信は一回で済むじゃないですか。
> だけどこれをSSRしてから`html`としてレスポンスしてたらどうですか？

🐶 **シッポ**

> あっ。そうか。キャッシュが効かなくなるんだ。
> `Hoge.js`というリクエストじゃなく、`Index`と`Counter`への完全に無関係な２つのリクエストになるから。
> それで、めっちゃでかいhtmlが２つ降ってくる。

🐱 **アンズ**

> 正解です！　さすが理解が早くて助かります♪
> というわけで、先程の処理フローを頭に入れた状態で、先に進んでくださいね！

## SSRの実装方針

🐱 **アンズ**

> じゃあ早速実装……と行きたいところなんですが、先にやりたいことのイメージをつかみましょうっ！
> まずは復習からです！
> 現在のソースコードはこうなっていますよね？

```html:index.html
<!doctype html>
<html lang="ja">
  <head>
    <meta charset="UTF-8" />
  </head>
  <body>
    <div id="app"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

```tsx:main.tsx
import { createRoot } from "react-dom/client";
import { Counter } from "./pages/counter";

const root = createRoot(document.getElementById("app")!);
root.render(<Counter />);
```

🐱 **アンズ**

> せんぱい、問題です！今、Counterコンポーネントがブラウザの画面に描画されるまでの流れはどうなってますか？

🐶 **シッポ**

> ええっと……
>
> 1. Viteの開発サーバーがあらゆるリクエストに対して一律で`index.html`をレスポンスする
> 2. `index.html`には、`<script>`があるから、ブラウザは`/src/main.tsx`をリクエストする
> 3. Viteがesbuildをつかって、tsxをjsにトランスパイルして、`/src/main.tsx`のレスポンスとして返す
> 4. `/src/main.tsx`からのimportに対して同じように、リクエスト→トランスパイル→レスポンスの流れ
>    だっけ？

🐱 **アンズ**

> そうです！ばっちりじゃないですか！SSRさせるときはそのフローを少しだけ変えるようにします！
> 具体的には、

:::message
Vite×ReactでのSSR実装方針

1. expressサーバーがあらゆるリクエストに対して一律で`index.html`を**エントリポイントとする**
2. **`index.html`を、`entry-server.tsx`をつかって、Reactコンポーネントが反映されたSSR済みのhtmlに変換する**
3. `index.html`には、`<script>`があるから、ブラウザは **`/src/entry-client.tsx`**をリクエストする
4. Viteがesbuildをつかって、tsxをjsにトランスパイルして、**`/src/entry-client.tsx`**のレスポンスとして返す
5. **`/src/entry-client.tsx`**からのimportに対して同じように、リクエスト→トランスパイル→レスポンスの流れ
6. **`React.hydrateRoot`をつかって、レンダリング済みのhtmlを`/src/entry-client.tsx`でhydrateする**

:::

> こうなるようにします！

🐶 **シッポ**

> おお。少しだけ変えるといいつつ、２工程ふえた……

🐱 **アンズ**

> あれ、本当ですね？
> 頭の中ではシンプルなつもりだったんですけど、文字にするとなんか複雑になっちゃいました。
> で、でも！　１つずつやればそこまで難しくないですから！　本当ですから！

## SSRの実装

🐱 **アンズ**

> さて、ようやく実装ですね……！　ここまでついてこれてますか？

🐶 **シッポ**

> な、なんとか……
> さっきの「Vite×ReactでのSSR実装方針」はまだなんとなくイメージつくかも？くらいだよ

🐱 **アンズ**

> いいですね。それで十分ですよっ！
> ここからさっきの実装方針に従って進めていくので、たまに戻って確認しながらついてきてください！

🐱 **アンズ**

> それで、具体的な実装方法ですが、これもViteのドキュメントに記載があります！
> なのでまずはそれを参考にしつつ進めていきましょうか！
> https://vite.dev/guide/ssr

🐶 **シッポ**

> またドキュメントあるんだ。
> アンズちゃんは開発もだけど、ドキュメントを見つける力もすごいね……

🐱 **アンズ**

> えへへっ。褒めてくれてありがとうございますっ。
> でも、Viteのドキュメントはかなりフレームワーク開発者のことも考えてくれているので、今回の目的としては読みやすいんですよ！

#### entry-server.tsxを使ってエントリポイントのindex.htmlをレンダリングする

じゃあまずは、以下の3ファイルを作成してください！

- `src/server.ts`：リクエストを受け付けるサーバー。Viteの開発サーバーをやめてこちらを使うようにします！
- `src/entry-server.tsx`：SSRにつかうエントリポイント
- `src/entry-client.tsx`：CSRにつかうエントリポイント

そしたら、以下のようにします！

```zsh
pnpm add express
pnpm add -D @types/node @types/express
```

ここまではいいですね？
そしたら、サーバー側でReactをレンダリングするロジックを作っていきますよ！

<実装方針>で言うところの、「1.expressサーバーがあらゆるリクエストに対して一律でindex.htmlをエントリポイントとする」「2.index.htmlをentry-server.tsxをつかって、Reactコンポーネントが反映されたSSR済みのhtmlに変換する」です！

まずは、`entry-server.tsx`から！
この子は、本当にブラウザとやりとりをしているexpressサーバーから呼び出されて、描画したいReactコンポーネントを文字列にレンダリングして返す働きをします。
つまり、この子がどんな文字列を返すか？　がSSRの結果そのものということになります。

```tsx:entry-server.tsx

import { renderToString } from "react-dom/server";
import { Counter } from "./pages/counter";

export type renderFunc = () => Promise<string>;

export const render: renderFunc = async () => {
  const html = renderToString(<Counter />);
  return html;
};
```

せっかくなので、前回動作確認用に、せんぱいに作ってもらった`Counter`をレンダリングしてみましたよ！

🐶 **シッポ**

> えっ。これだけ？

🐱 **アンズ**

> ……はい？　どうされましたか？

🐶 **シッポ**

> これって、Reactが提供してる、コンポーネントのレンダリング結果を文字列に出力するAPIを呼んでるだけだよね？
> Server Side Renderingって、もっとこう、あるんじゃないの？考慮しないといけないたくさんの事項が。

🐱 **アンズ**

> ふふっ。そうですね。たしかにServer Side Renderingなんて言われると、すごく難しそうなイメージがあるんですが、やってること自体は案外シンプルなんですよ！
> なので、はい！　これだけです！

これで無事、SSRのコアロジックができたので、次です！
そもそもブラウザからのリクエストを受け止めるサーバー部分を作っていきますよ！
フロントエンドやってるとぱっとイメージできないかもしれませんが、普段のWeb開発なら、ruby on rails, Django, hono, expressとかが使われる部分ですよ！

```ts:server.ts
import fs from "node:fs";
import path from "node:path";
import express from "express";
import { createServer as createViteServer } from "vite";

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

  app.use("*all", async (req, res, next) => {
    const url = req.originalUrl;

    try {
      // (1)expressサーバーがあらゆるリクエストに対して一律で`index.html`をエントリポイントとする
      let template = fs.readFileSync(
        path.resolve(process.cwd(), "index.html"),
        "utf-8",
      );

      // ViteのHMRなどにつかうclientモジュールを突っ込んでくれています。ここは「おまじない」くらいの理解で大丈夫です！
      template = await vite.transformIndexHtml(url, template);

      // サーバーサイドのエントリポイントの読み込みです！
      //ssrLoadModule経由で呼び出すことでHMRが提供されたり、ブラウザ用コードと同じようにESMで書いたもの
      // Node環境で動くように変換してくれたりします。
      // .tsxも自動でトランスパイルしてくれます！
      const { render } = await vite.ssrLoadModule("src/entry-server.tsx");

      // (2)`index.html`を、`entry-server.tsx`をつかって、Reactコンポーネントが反映されたSSR済みのhtmlに変換する
      const appHtml = await render();
      const html = template.replace(`<!--ssr-outlet-->`, () => appHtml);

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

createServer();
```

```html:index.html
<!-- SSRの結果を文字列置換するので、プレースホルダーとなるコメントを追記しました！ -->
<!doctype html>
<html lang="ja">
  <head>
    <meta charset="UTF-8" />
  </head>
  <body>
    <div id="app"><!--ssr-outlet--></div>
  </body>
</html>
```

と、こんな感じですね。
コメントにも書きましたが、SSRをする上での肝は

```tsx
const html = template.replace(`<!--ssr-outlet-->`, () => appHtml);
```

ここの部分です！
さっきの`entry-server.tsx`が返した文字列を、`index.html`のコメントと文字列置換しているだけなんですよ！

ってあれ、どうしましたせんぱい？

🐶 **シッポ**

> …………。
> `entry-server.tsx`が簡単だったから油断した……！

🐱 **アンズ**

> ご、ごめんなさいっ。難しかったですか？？

🐶 **シッポ**

> だ、だいじょうぶ。express触っておけばよかったかな……
> でも、ギリギリわかるかも。
> 詳しくは分からないけど、多分httpのハンドラを２つ（`vite.middlewares`と`async(req,....`）登録してサーバーを起動してるんだよね？
> それで、２つ目のハンドラでは`index.html`を読み込んで、`entry-server.tsx`の`render`でReactをレンダリングしてる。

🐱 **アンズ**

> えっ！　そうですそうです！　わかってるじゃないですか！
> 詳しくは分からなくても大丈夫です。そこだけわかってれば十分ですよ！

::: details vite.middlewaresってなに？

🐶 **シッポ**

> アンズちゃん。さっきのコードなんだけど

🐱 **アンズ**

> さっきのっていうと、`server.ts`の方ですかね？

🐶 **シッポ**

> そう。２つ目のハンドラ(`async(req,....`)は、SSRに絶対必要だって分かるんだけど、`vite.middlewares`って必要なのかな？
> もう自分でサーバー書いてるんだし、何に使われてるのか分からなくて。

🐱 **アンズ**

> なるほど、たしかに説明足りてなかったかもですね。
> じゃあせんぱい、Viteの開発サーバーって、ただレスポンス返す以外に何をしてくれてましたっけ？

🐶 **シッポ**

> ええっと、HMRはしてくれてると思うんだけど

🐱 **アンズ**

> いいですね。正解です♪　
> 他にも、tsx->jsへのトランスパイルも開発サーバーから呼ばれてたり、`/public`に置いた画像とかを静的ファイルとして配信してくれるのも、開発サーバーの機能なんですよ！

🐶 **シッポ**

> こうして聞くと、たしかに色々やってくれてるんだね。
> 普段フレームワークを使ってて見かける`/public`もここなんだ。

🐱 **アンズ**

> それでこれらの機能って、SSRするための自分でサーバーロジック書いてても普通に使えたら便利だと思いませんか？

🐶 **シッポ**

> 　あっ。なるほど。そういうことか。
> そういう開発サーバーの便利機能を、ミドルウェアとして提供してくれてるんだ。

🐱 **アンズ**

> さすが察しがいいですね！
> もう少しだけ書くと、expressの`app.use`は登録順に呼ばれるので、せんぱいのサーバーロジックの前に一旦リクエストを横取りしてる形になります！

:::

#### server.tsを実行する

🐱 **アンズ**

> さて。せんぱい、ギリギリって言ってましたけど、よくぞここまでついてきました！
> ここまで来たら、もうSSRの山場は超えたようなものですよ！

🐶 **シッポ**

> ほんとかなあ。
> そうだと嬉しいんだけど。

🐱 **アンズ**

> あ、あれ？　いつの間にか私、せんぱいの信頼を失ってませんか！？
> server.tsはたしかにちょっと難しかったかもしれませんけど……！　本当にもうちょっとなんです！

🐶 **シッポ**

> ごめんごめん。嘘だとは思ってないよ。
> じゃあ続きをお願いします

🐱 **アンズ**

> はいっ！
> じゃあ次はさっき作ったところまでを実行してみましょう！

`server.ts`をnodeで実行したいんですが、ここはViteくんのサポートが受けられないので、自分でトランスパイルしないといけません。
ビルドしてから実行したり・`Deno`でも良いんですが、今回はせっかくViteを使ってるので、Viteと同じくesbuildを内部的に使ってる`tsx`を使ってみましょうか。

```zsh
pnpm add -D tsx
```

そうしたら、tsxを使って`server.ts`を実行します。

```json:package.json

  "scripts": {
    "dev": "tsx src/server.ts"
  },
```

`http://localhost:5173/`にアクセスすると、

![](/images/frontend-framework-2/sample1.png)

ちゃんと表示されてますよね！

🐶 **シッポ**

> おお、ほんとだ。すごい。これでSSRができたんだ。
> って、あれ？　ボタンが動かないよ
> クリックしてもカウントが増えない。なにか間違えたかな？

🐱 **アンズ**

> 実はそうなんです！　それ、素晴らしい気付きですよ！
> それがちょうど今からお話したかったことで、何も間違ってないので安心してください♪

🐶 **シッポ**

> そうなんだ、よかった。
> ここまでもちょっと理解が怪しいから、動かなかった時のデバッグとかしんどいからね。

🐱 **アンズ**

> そういうデバッグが実はいい勉強になったりするので、勉強中はぜひたくさんバグらせてほしいんですが、私がいる限りそうはさせませんよっ！
> なんちゃって……。

#### hydrationをする

🐱 **アンズ**

> さて、ボタンが動かない原因なんですが、よく考えると当たり前なんです。
> だってブラウザ上ではせんぱいのJavaScriptが動いていないんです。

🐶 **シッポ**

> 動いてない……？
> でもちゃんと表示されてるよ？　ほら。

🐱 **アンズ**

> ふふふっ。とても教え甲斐がありますね♪
> じゃあ、せんぱい。ブラウザの開発ツールを開いて、`localhost`から返されてるhtmlを見てみてください。

![](/images/frontend-framework-2/sample2.png)

🐱 **アンズ**

> `/@vite/client`はViteがHMRするために入れてくれてる子です。
> (読み飛ばしていいですが、これは、`server.ts`で使ってる`vite.transformIndexHtml`がやってくれています)
> せんぱいの書いたJavaScript、見当たらないですね？

🐶 **シッポ**

> ほんとだ。
> うーん。なんで？

🐱 **アンズ**

> ふふっ。ごめんなさい、あんまり反応がいいので意地悪しちゃいました。
> もう一回、実装手順を見直してみましょうか！

:::message
Vite×ReactでのSSR実装方針(再掲)

- [x] 1. expressサーバーがあらゆるリクエストに対して一律で`index.html`を**エントリポイントとする**
- [x] 2. **`index.html`を`entry-server.tsx`をつかって、Reactコンポーネントが反映されたSSR済みのhtmlに変換する**
- [ ] 3. `index.html`には、`<script>`があるから、ブラウザは `/src/entry-client.tsx`をリクエストする
- [ ] 4. Viteがesbuildをつかって、tsxをjsにトランスパイルして、`/src/entry-client.tsx`のレスポンスとして返す
- [ ] 5. `/src/entry-client.tsx`からのimportに対して同じように、リクエスト→トランスパイル→レスポンスの流れ
- [ ] 6. `React.hydrateRoot`をつかって、レンダリング済みのhtmlを`/src/entry-client.tsx`でhydrateする

:::

> 完了した部分にはチェックをつけておきました。
> せんぱい、なにか気がつくことないですか？

🐶 **シッポ**

> あ、あれ？　全然終わってない？

🐱 **アンズ**

> はいっ。`server.ts`の実装が重かったのと、画面で見えるようになったのでつい気を抜いてしまうんですが、(3)〜(6)が残ってるんです！
> といっても、ここは挙動理解のために分割してるだけなので、作業内容はさっきより全然楽ですよ♪

まずは、「(3) `index.html`には、`<script>`があるから、ブラウザは **`/src/entry-client.tsx`**をリクエストする」です！

index.htmlに、ブラウザ上で動くJavaScriptを`<script>`で入れてあげないといけないんです！

🐶 **シッポ**

> あっ。よく考えたらそうか。
> これまでは、Reactの実行結果を、文字列としてhtmlに入れてただけだもんね。

🐱 **アンズ**

> はい！　正解です！
> なので、ブラウザ上でReact(`onButtonClicked`もそうですね！)を動かすためのJavaScriptを`index.html`に指定してあげないといけません。
> せんぱいこれ、どうすればいいか分かったりしますか？

🐶 **シッポ**

> 手順から察するに多分だけど、こういうことだよね？

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

🐱 **アンズ**

> はい！　ばっちりです！
> これで、手順(3)は完了ですね。

(4), (5)はViteが勝手にやってくれる部分(`server.ts`で`vite.middlewares`を書きましたからね！)なので、
残すは「(6)`React.hydrateRoot`をつかって、レンダリング済みのhtmlを`/src/entry-client.tsx`でhydrateする」だけですね。
`src/entry-client.tsx`は以下のようにします！

```tsx:src/entry-client.tsx
import { hydrateRoot } from "react-dom/client";
import { Counter } from "./pages/counter";

function hydrateApp() {
  hydrateRoot(document.getElementById("app")!, <Counter />); // (6): hydrationする
}

hydrateApp();
```

やってることは、`hydrateRoot`を呼んでるだけです！
実は、`index.html`で、`<div id="app"><!--ssr-outlet--></div>` としてあったので`document.getElementById("app")`の中身は、`renderToString(<Counter/>)`した結果になっているんです。

ここで呼んでる`hydrateRoot`は、

- 第1引数である`document.getElementById("app")`の中身は、
- 第2引数である`<Counter />`をレンダリングしたのと同じものだから、

この`<Counter/>`を使ってhtmlを動くようにしてね(これをhydrationといいます！)という命令なんですよ！

🐱 **アンズ**

> せんぱい、これでもう一回動かすとどうですか？

🐶 **シッポ**

> ……！　動いてる！　カウンターアプリが動作してるよ！

🐱 **アンズ**

> やりましたね！　これで無事SSRの完成です！

🐶 **シッポ**

> にしても、普段なら一瞬で作れるSSRのカウンターアプリを動作させるだけでこんなに大変なんだね

🐱 **アンズ**

> そうなんですよー。
> 自分で再発明すると、フレームワークのありがたさが身にしみますよね
> ……っと、ついSSRが完成して感傷に浸りそうになりましたが、まだ先がありますからね！

:::message
自作フレームワークまでの道順(再掲)

- [x] Viteの導入
- [x] TypeScript,Reactの設定
- [x] SSR(server side rendering)
- [ ] React Router(router,loader)の導入
- [ ] Monorepoで管理する

:::

> ここまでで３つ終わりました！　あとは２項目です！
> もう半分以上は進んでることになります！　
> 次回はReact-Routerを導入してルーティングを作っていきますよ！
