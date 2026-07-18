---
title: "【React Router編】後輩ちゃんと学ぶ自作フレームワーク！！ Hydration failedの正体とloaderの仕組み"
emoji: "🐈️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react", "reactrouter", "vite", "ssr", "typescript"]
published: true
---

:::message
この記事の目的
普段はNext.jsなどのフレームワークを使って開発しているエンジニアが、内部を理解するためにフレームワークを自作してみる記事です。
執筆時点で筆者が公開しているフレームワークはこちらにあります
https://github.com/AN-Sippo/sippa

この記事は第3回です。前回記事は[こちら](https://zenn.dev/sippooo/articles/frontend-framework-2)

:::

## 登場人物たち

| 見た目 　                            | 名前   | プロフィール                                            |
| ------------------------------------ | ------ | ------------------------------------------------------- |
| ![](/images/general/Sippo.png =100x) | シッポ | 筆者。Webフロントエンドが少しだけ分かる。               |
| ![](/images/general/anzu.png =100x)  | アンズ | Sippoの後輩。技術に詳しい。とりあえずなんでも知っている |

**アンズ**

> さて、前回まででViteとexpressサーバーを使ってSSRの仕組みを実装できましたね。

**シッポ**

> できたね。めっちゃ大変だったけど、こうして実装してみると、"hydration"が具体的になにをしてるのかとか、イメージが湧くようになったよ。

**アンズ**

> いいですね。他人が作った技術を「hydrationは乾燥した粉末に水を加えるところから命名されていて〜」って説明されるより、一回自分で体験する方が勉強になりますからね！
> せんぱい、いまめちゃくちゃいい経験をしてますよ！

**シッポ**

> そうかもしれない。
> 今回はReact Routerを使うんだっけ？

**アンズ**

> そうですねっ。そのつもりです！
> まずは全体の流れを見返しましょうか！

:::message
自作フレームワークまでの道順

- [x] Viteの導入
- [x] TypeScript,Reactの設定
- [x] SSR(server side rendering)
- [ ] React Router(router,loader)の導入
- [ ] Monorepoで管理する

:::

> 前回でSSRが完成したのでチェックをつけておきましたよ！
> なので、今回がReact Routerですね！

**シッポ**

> おお、こうしてみると半分超えたんだね。

**アンズ**

> はい、そうなんです！　じゃああと少し。やっていきましょうか！

## React Router導入の実装方針

**アンズ**

> 早速ですがいつも通り全体像から見ていきましょう。今回の手順はこうです！

:::message
React Routerの設定手順

- [ ] (1).`route.ts`にルート定義をする
- [ ] (2).クライアントサイドでルーティングする
- [ ] (3).サーバーサイドでルーティングする
- [ ] (4).loaderの設定をする

:::

**アンズ**

> 前回のSSRより大分シンプルになってますよ！

**シッポ**

> ほんとだね。それぞれの中身も、割と理解しやすいかも。
> ルート定義(URLのパスとReactコンポーネントの対応表)を１つ書いて、それをブラウザとサーバーの２箇所から使う感じだよね。

**アンズ**

> そうですそうです！
> 今回は「(4).loaderの設定をする」が少しだけ難しいですが、(1)〜(3)は前回SSRの実装よりは楽なはずですよ！
> じゃあ、まずは(1)からやっていきましょう！

## 1.route.tsにルート定義をする

**アンズ**

> ここは基本的にReact-Routerの記法に従うだけなので、解説というよりチュートリアル寄りになるかもしれません。

**シッポ**

> チュートリアルか。
> でも、React-Routerを直接触ったことないんだよね。大体、ルーティングはフレームワークがやってくれちゃうから、使う機会がなくて。

**アンズ**

> 意外とみんなそんなものだと思いますよ！
> 「名前はさすがに知ってるけど、実は使ったことない〜」って人、珍しくないんじゃないかなあって、個人的に思っています……！

**シッポ**

> そうなんだ。ちょっと安心したよ。

**アンズ**

> で、早速なんですけども

**シッポ**

> ……?

**アンズ**

> React-Routerって、実はモードが３種類ありまして……

**シッポ**

> えっ、なんかもう難しそうなんだけど

**アンズ**

> だ、だいじょうぶです！　厳密な違いは知らなくていいです！　というか私も詳しくはないです！

**シッポ**

> （アンズちゃんでも知らないことってあるんだ……）

**アンズ**

> 最低限知っておくべきことだけまとめておきましょう！　あとは雰囲気でいいです！

:::message
今回知っておくべきReact-Router

1. モードが３つあり、Declarative→Data→Framework
2. 左は機能が少ない分自由度が高くて、右は高機能な分自由度が低い(Frameworkとか言ってますし！)
3. 今回は真ん中のData modeを使う

:::

:::details なんでData modeなの？

理由は主に２つあります！

1. 今回、フレームワークの自作なのでloaderを提供したいんですけど、それを完全に自前で用意するのは少し大変なので、そのサポートが受けられるものにしました。（Declarativeにはloaderがないんです）
2. フレームワークの自作なので、frameworkモードを使うとやることがなくなってしまいます(笑)

:::

**シッポ**

> 分かった。
> とりあえず今回は、ほどほどに機能があって、ほどほどに自由が効くDataモードを使うんだね！

**アンズ**

> はい！　それだけわかってもらえてたら大丈夫ですよ！
> それじゃ、気を取り直してインストールからやっていきましょうか！

```zsh
pnpm add -D @types/react-router
pnpm add react-router
```

**アンズ**

> 早速React-Routerを書いていきたいんですが、せんぱい、このプロジェクトってどんなページがありましたっけ？

**シッポ**

> えっと、動作確認で使ってる`Counter.tsx`があるでしょ？　他は……あれ？

**アンズ**

> ないですよね笑
> なので、適当にページを増やしておきましょう！

```tsx:src/pages/Index.tsx
export const Index = () => {
  return <h1>ここはルート"/"ですよっ</h1>;
};
```

そうしましたら、今度こそReact Routerを使っていきます！　`src/route.ts`を作成してください！
ここで書くのは、パスとReactコンポーネントの対応表みたいなものです。

```ts:src/route.ts
import type { RouteObject } from "react-router";
import { Index } from "./pages/Index";
import { Counter } from "./pages/counter";

export default [
  {
    path: "/",
    Component: Index,
  },
  {
    path: "/counter",
    Component: Counter,
  },
] as RouteObject[];

```

**アンズ**

> パスと、Reactコンポーネントを対応付けてるだけですよね？
> ここは大分理解しやすいんじゃないでしょうか？

**シッポ**

> うん、そうだね。
> ここはなんとなくやってることが理解できるよ

**アンズ**

> はい！　じゃあ、リストの１つ目「完」ということで

:::message
React Routerの設定手順

- [x] (1).`route.ts`にルート定義をする
- [ ] (2).クライアントサイドでルーティングする
- [ ] (3).サーバーサイドでルーティングする
- [ ] (4).loaderの設定をする

:::

**シッポ**

> え、はやっ。もう終わったの？

**アンズ**

> えへへっ。そうなんです。
> ここはサクッと片付けて次に行きますよっ！

## 2.クライアントサイドでルーティングする

**アンズ**

> そしたら、さっき書いたルート定義を実際に使うところを書いていきましょう。
> 具体的には、React-Routerが提供してる`RouterProvider`を使います！

**シッポ**

> `RouterProvider`……？　初めて聞いたかも

**アンズ**

> React-Routerの提供してるインターフェースなので、知らなくても無理ないです。
> ざっくり言うと、さっきのルート定義をつかって、現在のルートにマッチするコンポーネントを出し分けるコンポーネントです！

**シッポ**

> あ、なるほど。さっき"path"フィールドに指定したパスと現在のURLを、実際に比較してくれてるんだ。
> 擬似コード的にかくと、

```tsx
const RouterProvider = ({ routes }: { routes: RouteObject[] }) => {
  const path = useLocation();

  for (const route of routes) {
    if (match(route.path, path)) {
      return <route.Component />;
    }
  }
};
```

> みたいな実装になってるイメージってこと？

**アンズ**

> えっ。そうです……!　そうなんですけどっ、
> せんぱいがいつの間にか擬似コードを使いこなしてる……
> なんか、他の女の子の影を感じます

**シッポ**

> えっ？　なに？

**アンズ**

> ごっほん。な、なんでもないです。
> ともかく理解はそれでいいです。

**シッポ**

> ……なんか怒ってない？

**アンズ**

> 怒ってません。
> それじゃあ、実際に`RouterProvider`を使っていきますよっ！

**シッポ**

> えぇ。唐突……。

では、`hydrateApp`をこんな風に修正しましょう！
`Counter`を直接レンダリングするのをやめて、RouterProviderにルート定義を渡してコンポーネントを選んでもらうようにしてあります！

```tsx:src/entry-client.tsx
import { hydrateRoot } from "react-dom/client";
import { createBrowserRouter, RouterProvider } from "react-router";
import routes from "./route";

function hydrateApp() {
  const router = createBrowserRouter(routes);

  hydrateRoot(
    document.getElementById("app")!,
    <RouterProvider router={router} />,
  );
}

hydrateApp();
```

これで実行しますと……

| `/`                                           | `/counter`                                    |
| --------------------------------------------- | --------------------------------------------- |
| ![](/images/frontend-framework-3/sample1.png) | ![](/images/frontend-framework-3/sample2.png) |

こんな風に、ちゃんとコンポーネントが切り替わっていれば成功です！

**シッポ**

> アンズちゃん、質問なんだけど

**アンズ**

> はいっ。……もしかしてどこかで詰まりましたか？

**シッポ**

> とりあえず今のルーティングまでは動いてはいるんだけど、なんか`/`にいるときだけ、`Hydration failed`ってエラーが出てるんだよね。

```
react-dom_client.js?v=95b91570:3146 Uncaught Error: Hydration failed because the server rendered HTML didn't match the client. As a result this tree will be regenerated on the client. This can happen if a SSR-ed Client Component used:

- A server/client branch `if (typeof window !== 'undefined')`.
- Variable input such as `Date.now()` or `Math.random()` which changes each time it's called.
- Date formatting in a user's locale which doesn't match the server.
- External changing data without sending a snapshot of it along with the HTML.
- Invalid HTML tag nesting.

It can also happen if the client has a browser extension installed which messes with the HTML before React loaded.
```

**アンズ**

> ……せんぱい、ちゃんと気がついてくれるのさすがです！
> そうなんですよ。現状だとこうなってしまいます。
> せんぱい、まずこの`Hydration Failed`ってどういうエラーかわかりますか？

**シッポ**

> えっと、たしか、`hydrateRoot`を呼んだときのDOMと、`hydrateRoot`に渡したコンポーネントをレンダリングしたDOMが一致しなかったエラー。つまり、SSRの結果とCSRの結果が一致しなかったってこと？

**アンズ**

> その通りです！　じゃあ、いまSSRはどうなってましたっけ？

```tsx:src/entry-server.tsx
import { renderToString } from "react-dom/server";
import { Counter } from "./pages/counter";

export type renderFunc = () => Promise<string>;

export const render: renderFunc = async () => {
  const html = renderToString(<Counter />);
  return html;
};
```

**シッポ**

> あっそうだった。SSRでは固定で`<Counter/>`を返すようにしてるから、
> `/counter`ではCSRでも`<Counter/>`がレンダリングされるからhydration failedが起きなくて、
> `/`ではCSRで`<Index/>`がレンダリングされるからhydration failedが起きるんだ。

**アンズ**

> 表にするとこうなりますよ！

| ルート     | CSR          | SSR          | 結果   |
| ---------- | ------------ | ------------ | ------ |
| `/`        | `<Index/>`   | `<Counter/>` | 不一致 |
| `/counter` | `<Counter/>` | `<Counter/>` | 一致   |

**シッポ**

> なるほどね。
> こう、hydration failedってぼんやりとした理解だったけど、現在アクセスしてるルートによって、起きる・起きないが、ちゃんと変わるの嬉しいよね。

**アンズ**

> なんか分かるかもです……。
> 自分の理解が正しいと証明された気がするんですよね！

**シッポ**

> そう！　それだ！
> そうしたら、次はSSRにルーターを導入するのかな？

**アンズ**

> 正解です！
> 今の進捗はこんな感じなので、「(3).サーバーサイドでルーティングする」やっていきましょう！

:::message
React Routerの設定手順(再掲)

- [x] (1).`route.ts`にルート定義をする
- [x] (2).クライアントサイドでルーティングする
- [ ] (3).サーバーサイドでルーティングする
- [ ] (4).loaderの設定をする

:::

## 3.サーバーサイドでルーティングする

[参考：React-Router API Reference](https://reactrouter.com/start/data/custom#3-lazy-loading)

**アンズ**

> それでは、つぎはサーバー側の実装をしていくんですが、基本的な流れはさっきと同じです。
> まず、`route.ts`の定義をつかって、`entry-server.tsx`を書き換えていきます。
> さきにコードを見てもらいましょうか！

```tsx:src/entry-server.tsx
import { renderToString } from "react-dom/server";
import {
  createStaticHandler,
  createStaticRouter,
  StaticRouterProvider,
} from "react-router";
import route from "./route";

export type renderFunc = (request: Request) => Promise<Response | string>;
let { dataRoutes, query } = createStaticHandler(route);

export const render: renderFunc = async (request: Request) => {
  // loader,actionがあった場合はここで実行されます！
  const context = await query(request);

  // actionの結果、リダイレクトが返ってきたときなどはそれをexpressまで中継しないといけませんよね
  if (context instanceof Response) {
    return context;
  }

  // 色々といい感じにまとめて、ちょっと便利関数を生やして、StaticRouterProviderの型に合わせます
  const router = createStaticRouter(dataRoutes, context);

  const html = renderToString(
    <StaticRouterProvider router={router} context={context} />,
  );
  return html;
};

```

**アンズ**

> クライアントサイドよりはちょっと記述量が多いですが、一旦受け入れて、気になったら後で聞いてください！
> それは別にして、もう一個問題があるんですが、なにかわかりますか？

**シッポ**

> え？　うーん……？

**アンズ**

> ちょっと急すぎましたかね💦
> そうですね……。exportしてる関数の型注目してみるとどうでしょう？

**シッポ**

> あ！　型が変わってる！
> 引数がなかったのにRequestを受けるようになって、返り値もstringじゃなく、Responseとのユニオン型になってる！

**アンズ**

> そのとおりです！
> そこだけ取り出して見比べるとこうなってます！

```tsx:src/entry-server.tsx(旧)
export type renderFunc = () => Promise<string>;
```

```tsx:src/entry-server.tsx(新)
export type renderFunc = (request: Request) => Promise<Response | string>;
```

**アンズ**

> クライアントサイドでは、ルーターが中で`location.href`などを参照して、現在地を知ることができたのに対して、
> サーバーサイドでは、現在地を知る方法がないので、どのルートへのリクエストか？　の情報であるRequestオブジェクトを渡してあげないといけないんです……!
> これが厄介で……。

**シッポ**

> あーそっか。それでRequestが必要なんだね。でもこれって言うほど問題なの？
> たしかにちょっと`server.ts`をいじる必要はあるけど、それだけじゃない？

**アンズ**

> ……。もしそうだったら私もウキウキで`server.ts`を載せて終わりにしてたんですけど……

**シッポ**

> え、な、なに？

**アンズ**

> 実はこのRequestオブジェクト、ついでにResponseオブジェクトも、これ、expressのRequest/Responseとは全然別のオブジェクトなんですよ……

**シッポ**

> うわあ。めんどくさいやつだ。
> なんでそんなことに……

**アンズ**

> 構造としては、React-Routerが使ってるRequest/ResponseがWeb標準のもので、
> express側のオブジェクトが使ってるのがNode.js独自のもの　って感じです。

**シッポ**

> なるほどねえ。
> ……じゃあ、これ、もしかして今から変換のヘルパー関数を書くながれ？

**アンズ**

> ……と、なると地獄なので、Remixチームがアダプターを提供してくれています！

**シッポ**

> おお！　さすがすぎる！

**アンズ**

> なので、ありがたく使わせてもらいつつ、`entry-server.tsx`を呼び出している、`server.ts`側も修正していきます！

```zsh
pnpm add @remix-run/node-fetch-server
```

```ts:src/server.ts(一部抜粋)
app.use(async (req, res, next) => { // これまでは'*all'としてましたが、expressではマッチしたパスが切り取られてしまうので、消しておきます！
  // ....中略 ....

  try{
    // ....中略 ....
    const { render } = (await vite.ssrLoadModule("src/entry-server.tsx")) as {
      render: renderFunc;
    };

    // ここ！　アダプターを挟んでundiciベースのリクエストをつくっています！
    const routerResult = await render(createRequest(req, res));　

    if (routerResult instanceof Response) {
      // リダイレクト等であれば、そのまま返します！
      sendResponse(res, routerResult);
      return;
    }

    // それ以外の場合は、これまで通りにSSR結果で置換します
    const html = template.replace(`<!--ssr-outlet-->`, () => routerResult);
  }
  catch(e){
    // ....中略 ....
  }

})


```

**アンズ**

> せんぱい、どうですか？　これでhydration failed消えたんじゃないでしょうか！

**シッポ**

> あ！　消えてる！　動いてる！

**アンズ**

> やりましたね！　これで、無事SSRでのルーティング基本が完了です！
> というわけで、チェックをつけておきますね！

:::message
React Routerの設定手順(再掲)

- [x] (1).`route.ts`にルート定義をする
- [x] (2).クライアントサイドでルーティングする
- [x] (3).サーバーサイドでルーティングする
- [ ] (4).loaderの設定をする

:::

**アンズ**

> さて、残すはloaderだけです！
> これが、この記事のラスボスです！　気合いれていきましょー！

:::details entry-server.tsxが複雑すぎない？

**シッポ**

> その前になんだけど、

**アンズ**

> あれ？　はい、どうしました？

**シッポ**

> さっきの、`entry-server.tsx`の記述量がクライアントサイドと比べて多い話、やっぱり気になるから教えてもらえる？

**アンズ**

> あっ、そうでした！
> そういえばRequestオブジェクトのアダプターに着目してほしくて、そっちは流したんでしたっけ……
> はい！　もちろん解説しますよっ。

理由としては、おそらくですが主に3つありまして、

###### 1. そもそも`createStaticHandler`の層はReactについて知らないから

実は、このレイヤはReact-Routerが内部でライブラリ的に使ってまして、これを使っているアプリケーションがどんなライブラリで動作しているのかを知りません。
vue.jsでも、svelteでも、ただのhtmlファイルでも、関係ないです。
やってることは、ルート定義に従って、loader,actionを実行することがメインです。
コンポーネントはただの`Function`型として扱って、保持しておくだけです。

ですので、Reactとloader/actionの実行主体が疎結合な分、Reactとの仲介をしてあげるコードが必要になったんです。

###### 2. Redirectなどをhttpサーバーまで渡さないといけないから

こっちはもっと単純です。
React-RouterのData Modeは、httpサーバーを持ちません。
仮にactionを実行した結果、リダイレクトが返ってきたとして、自分じゃどうしようもできません。
なので、せんぱいのフレームワークに、「なんかリダイレクト返ってきたんですけど！？」と伝えるしかないんです。
それで、`query`の返り値が`Context | Response`となっていて、それを一旦フレームワーク側で受け止めてあげないといけないんです。

###### 3. フレームワーク側に色々な裁量を与えるため

これはもっと抽象的な話で、前の２つとも被ります。
React-RouterのData Modeはあくまでもライブラリですので、アプリ全体の流れを決めるのは、あくまでもその上のフレームワーク層です。
今回のコードで見たように、細かく呼び出されているので、フレームワーク側では、

- こういう時はloaderを実行しない
- loaderの結果がこうだったときは、ReactではなくVueでレスポンスする

などの細かい制御がやりやすいです。
つい勢いで、奇抜な例をあげてしまったんですが、(いいことなのかはさておき)こういった制御をすることができるようになっているんです。

:::

## 4. loaderの設定をする

**アンズ**

> さて、記事の冒頭からラスボスですよとお伝えしてきたloaderをやってきます。
> まずせんぱい、loader/actionがそれぞれどんなものか、説明できますか？

**シッポ**

> ええっと……、どっちもSSRするときに必要になる概念で、
> loaderが、そのパスで描画する際に必要な情報をどこかからfetchしてくること。
> actionが、そのパスで実行されてほしい処理をどこかにリクエストしてくること。
> こんな理解をしてるんだけどあってるかな？

**アンズ**

> そうですね！　さすがです！
> もっと言うと、loaderは主にGETリクエストで実行されて、actionは主にPOSTリクエストで実行される。みたいなことも言えたりしそうですね！
> じゃあ、これらをフレームワークとしてサポートしたいとき、どんな実装が必要になるかわかりますか？

**シッポ**

> まず、アプリケーション側からloaderをかけるインターフェースでしょ？　それから、自動で実行するようにして、あとは……、なんだろう？

**アンズ**

> ふふっ。でもかなり惜しいです！　いい線いってますよ！　loader/actionをサポートするために必要な実装は大まかにこんな感じでまとめることができます！

:::message
loader/actionのサポートに必要な実装

1. loader/actionを記述するインターフェース
2. サーバーサイドでの実行
3. クライアントサイドでの実行
4. サーバーからクライアントサイドへのデータ受け渡し

:::

> せんぱいが挙げてくれたのは、1,2,3あたりになりそうですかね？

**シッポ**

> そうなるかな。
> あ、クライアントサイドでの実行も必要なんだ。

**アンズ**

> そうですね！　SSRが実行されるのはあくまでもアプリへの初回アクセスのみなので、アプリ内遷移でのアクセスは全部クライアントサイドナビゲーションになります。
> その時に、loaderが実行できないと困っちゃいますからね！

**シッポ**

> なるほど。そうだね。4.サーバーからクライアントサイドへのデータ受け渡しはたしかに難しいかも。SSRで実行したloaderの結果をCSR環境まで渡さないといけないもんね。

**アンズ**

> サーバーからクライアントサイドへのデータ受け渡しが必要な理由はそれでばっちりです！
> 実はこれ、仕組みはめちゃくちゃ単純なんですけどね

**シッポ**

> えっ、そうなの？

**アンズ**

> そうなんですよ〜。
> 単純というか、力技というか……笑
> この仕組みも道中で説明したほうがきっと分かりやすいと思うので、実装しながら解説しますね！

では、実装やっていきますよっ。
まず、1.loader/actionを記述するインターフェースですが、これはReact-Routerが用意してくれてるものをそのまま使っていきます！

まず、loaderを動作させるためのページを追加します！

```tsx:src/pages/Dog.tsx
import { useLoaderData } from "react-router";

// これがloader関数です
// この関数は自由に書いてもらって、このあとでこの関数をloaderとして設定します！
export const dogLoader = async () => {
  const response = await fetch("https://dog.ceo/api/breeds/image/random");

  if (!response.ok) {
    throw new Error("data load error");
  }

  return await response.json();
};

export default function Dog() {
  const dog = useLoaderData();

  return (
    <div>
      <h1>Dog</h1>
      <img src={dog.message} />
    </div>
  );
}
```

APIが叩ければなんでもいいんですが、今回はDog APIを使ってみました！
犬の画像をランダムで返してくれるAPIで、私は結構気に入ってるんです！

そして、今追加したページとそのloaderを、ルート定義に追加します。
これまでComponentを書いてたオブジェクトに、loaderプロパティが用意されています！　React Routerがインターフェースを用意してくれてるってのは、こういうことです！

```ts:src/route.ts
import type { RouteObject } from "react-router";
import { Index } from "./pages/Index";
import { Counter } from "./pages/counter";
import Dog, { dogLoader } from "./pages/Dog";

export default [
  {
    path: "/",
    Component: Index,
  },
  {
    path: "/counter",
    Component: Counter,
  },
  {
    path: "/dog",
    loader: dogLoader, // ここでloaderを設定します！
    Component: Dog,
  },
] as RouteObject[];

```

**アンズ**

> さて、せんぱい！

**シッポ**

> ……? とりあえず４つのうち１つが終わったところだよね？

**アンズ**

> いえ、これで終わりですっ！

**シッポ**

> ええ！　だってloaderは一番長くて大変だって言ってなかった？

**アンズ**

> そのつもりだったんですけど、
> ライブラリたちが優秀すぎてやることがありませんでした……
> 先程の実装リスト2-4、全部React-Routerが自動でやってくれています……

**シッポ**

> すっご。じゃあこれでもうやることないの？

**アンズ**

> そうなっちゃいますね……

**シッポ**

> なんか悔しそうじゃない？

**アンズ**

> え！　そっ、そんなことないです！　喜ばしいことです！
> （私が前にやったときは、`window.__staticRouterHydrationData`をねじこむ<script>のせいでhydration failedが起きて、それを直すために標準のデータ受け渡しを無効化して、別でセットアップが必要だったのに……。これ、せんぱいに教えたかったのに……！）

**シッポ**

> 本当かなあ

**アンズ**

> さ、さて！　これで実行してみると……
> ![](/images/frontend-framework-3/sample3.png)

しっかり表示されてますよね！

**アンズ**

> どうですか、せんぱい？
> うまくいきました？

**シッポ**

> 動いたよ！……でもこれ、いまいちloaderが実行されてる感触がないね？

**アンズ**

> 感触！？　うーん、そうですねえ。
> では、いくつか確認事項を用意しますので、それを一緒に確認してみましょうか！

:::details (おまけ)アンズ作：loaderが実行されていることの確認

#### 1.URL直接アクセスで、ブラウザからDog APIへのリクエストが飛んでいないこと

おなじみこの画面で、Dog APIと思しきリクエストがなければOKです！
![](/images/frontend-framework-3/check1.png)

#### 2.クライアントサイドナビゲーションで、ブラウザからDog APIへのリクエストが飛んでいること

こちらも同じく、ここに「random」という名前でDog APIにリクエストが飛んでいることが確認できればOKです！
![](/images/frontend-framework-3/check2.png)

#### 3.SSRでのloaderがブラウザにデータを受け渡す現場を確認すること

最後ですが、`/dog`にアクセスしたときにレスポンスされているhtmlファイルを取ってきました！
`window.__staticRouterHydrationData = JSON.parse(....)`としている部分がありますよね？

```html
<body>
  <div id="app">
    <link
      rel="preload"
      as="image"
      href="https://images.dog.ceo/breeds/airedale/n02096051_3700.jpg"
    />
    <div>
      <h1>Dog</h1>
      <img src="https://images.dog.ceo/breeds/airedale/n02096051_3700.jpg" />
    </div>
    <script>
      window.__staticRouterHydrationData = JSON.parse(
        '{"loaderData":{"2":{"message":"https://images.dog.ceo/breeds/airedale/n02096051_3700.jpg","status":"success"}},"actionData":null,"errors":null}',
      );
    </script>
  </div>
  <script type="module" src="/src/entry-client.tsx?t=1784068521576"></script>
</body>
```

SSR時には、こうやってwindowオブジェクトのプロパティとしてloader/actionの結果をセットする、`<script>`をねじ込む仕組みになっているんです！

これで、データが受け渡されていることも確認できますね！
ちなみにこれが、この章冒頭でお話した、「SSRで実行したloaderの結果をCSR環境まで渡す方法」になりますよっ！

:::

**アンズ**

> と、こんな感じでどうでしょうか？

**シッポ**

> うん。
> devtoolsで動作を確認できるとやっぱり納得できるよね。

**アンズ**

> それはよかったです！
> ちなみに、Actionsも全く同じ手順で実装できるので、ぜひやってみてください！
> あ、最後に今回の成果を更新しておきますね！

:::message
React Routerの設定手順(再掲)

- [x] (1).`route.ts`にルート定義をする
- [x] (2).クライアントサイドでルーティングする
- [x] (3).サーバーサイドでルーティングする
- [x] (4).loaderの設定をする

:::

## 終わりに

**アンズ**

> さて、せんぱい！　React-Routerの導入お疲れ様でした！
> これで自前でフレームワークの主要な機能はだいたい実装できたことになります！

**シッポ**

> Viteとかライブラリのサポートを受けながらだけど、ルーティングとSSRを実装して、だいぶフレームワークのブラックボックスが小さくなった気がするよ

**アンズ**

> いいですね！　
> せっかくここまで作ったので、次回は最後に、せんぱいのフレームワークを外から使える状態にもっていきましょう！
> これが終わればいよいよ、「自作のフレームワークでWebアプリを作る」なんてことができるようになっちゃいますよっ！

**シッポ**

> おお、フロントエンドエンジニアの夢だね
> じゃあ、次回もよろしくお願いします！

**アンズ**

> おまかせくださいっ！
