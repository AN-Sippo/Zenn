---
title: "後輩ちゃんと学ぶ自作フレームワーク！！！Vite×React Router(基礎編)"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["react", "vite"]
published: true
---

:::message
この記事の目的
普段はNext.jsなどのフレームワークを使って開発しているエンジニアが、内部を理解するためにフレームワークを自作してみる記事です。
執筆時点で筆者が公開しているフレームワークはこちらにあります
https://github.com/AN-Sippo/sippa

:::

## 登場人物たち

| 見た目 　                            | 名前   | プロフィール                                            |
| ------------------------------------ | ------ | ------------------------------------------------------- |
| ![](/images/general/Sippo.png =100x) | シッポ | 筆者。Webフロントエンドが少しだけ分かる。               |
| ![](/images/general/anzu.png =100x)  | アンズ | Sippoの後輩。技術に詳しい。とりあえずなんでも知っている |

## フレームワークって魔法じゃないの？？

🐶 **シッポ**

> 最近、Next.jsとかRemixとかのフレームワークを使って開発できるようになってきたんだよ。
> もう立派なフロントエンドエンジニアでしょ。

🐱 **アンズ**

> 　いいじゃないですか。せんぱい、実務でもフロントエンドやってましたもんね。

🐶 **シッポ**

> でもさ、このフレームワークってどうやって動いてるんだろう
> SSRとかHotReloadとか、ファイルベースルーティングとか、`npm run build` の裏側で動いてる `next build` みたいなCLIコマンドも、結局何やってるのか分からないし。
> これはもうアレだよね。魔法だよね。

🐱 **アンズ**

> 魔法って笑
> そんなことないですよ！
> 結局中身はバンドラ・トランスパイラ・ルーター・SSR・開発サーバーみたいな機能に分割できて、それを簡単に使えるようにまとめてるだけだったりするんですよ。

:::message
フレームワークの基本的な機能

1. バンドラ：複数のJavaScriptファイルを１つにまとめる(webpack,rollup,rolldownなど)
2. トランスパイラ：TypeScriptやJSXをJavaScriptに直す(babel,esbuildなど)
3. ルーター：「`/page`へのアクセスなら`Page.tsx`を表示する」のようなルートとコンポーネントの対応付け(ReactRouter, Tanstack Routerなど)
4. SSR・SSG・ISR：レンダリングをサーバー側やビルド時に済ませること
5. 開発サーバー: ローカルのPCで動かす、開発用のサーバー(`next dev`などで立ち上がるやつ)

:::

🐶 **シッポ**

> そうなんだ？
> うーん、聞いても良く分からないね

🐱 **アンズ**

> じゃあせんぱいも作ってみたらどうです？
> 最近だと、Viteっていうパッケージがさっき説明した機能をほとんど提供していて、それをベースにしたフレームワークが増えてきてたりしますよ。
> Viteにルーティングを追加して、全体をラップすれば立派なフレームワークの完成ですよ

🐶 **シッポ**

> え？
> フレームワークって自分でもつくれるの？

🐱 **アンズ**

> そりゃそうですよ！
> Next.jsとかのOSSだって実際に作ってるcontributorsがいるわけですし。
> もちろんそこまでのレベルに磨き上げるのは大変だと思いますけど、勉強ついでに自分専用で作るくらいなら、全然やれますよ！！

🐶 **シッポ**

> じゃあやってみようかな。
> 困ったら教えてくれる？

🐱 **アンズ**

> もちろんです！一緒にフレームワーク作ってみましょう！

## まずはViteを動かしましょう！！

🐱 **アンズ**

> せんぱい、まずViteって使ったことあります？？

🐶 **シッポ**

> ないです

🐱 **アンズ**

> 実はNuxt3, Svelte, Astro, Remixとかの有名OSSもViteをベースに作られてるんですよ！
> 残念ながら、一番と言っていいほど有名なNext.jsはwebpackやTurbopackを使ってるので、イメージ湧きにくいかもしれないですけど……

🐶 **シッポ**

> おお、聞いたことある奴らばっかりだね。
> そんなに凄いやつなんだね"バイト"って。

🐱 **アンズ**

> そうなんですよ！
> でも実はこれ、"バイト"じゃなくて"ヴィート"と読むみたいですよ。
> なんと、[公式ドキュメントの１行目](https://vite.dev/guide/#overview)に発音が書いてあります！

🐶 **シッポ**

> ドキュメントに発音書いてあることなんてあるんだ。
> たまにある絶対に初見じゃ読めないやつか……

🐱 **アンズ**

> それじゃあまずは、Viteを動かしていきましょう
> Viteだけでも割とフレームワークを使ってる感覚が強いんですよ！

## Viteの初回セットアップ

🐱 **アンズ**

じゃあ、まずはViteのドキュメント(https://vite.dev/guide/)を開いてください！
`npm create vite@latest`で簡単にプロジェクトが作れちゃうんですが、今回はフレームワークを作ることが目的なので、勉強もかねて[手動でセットアップ](https://vite.dev/guide/#manual-installation)していきましょう！

まず、次のコマンドを叩いてください！
フロントエンドやってたせんぱいなら、意味はわかりますよね？？

```shell
pnpm init
pnpm add vite
```

そしたら、package.jsonに以下を追加してください！
他にもbuildとかpreviewとかありますが、一旦これだけでいいです！

```js
  "scripts": {
    "dev": "vite"
  },
```

そして、最後に`index.html`を作成してください！
Viteではこのhtmlファイルを基準に全てを読み込みます！

JavaScriptを追加した時でも、「最終的にこのhtmlとどう関連するのか」って視点で見ていくと見通しが良くなるので、意識してみてください！
私も必要なときは基本に立ち返って説明しますね！

```html
<!doctype html>
<html lang="ja">
  <head>
    <meta charset="UTF-8" />
  </head>
  <body>
    <h1>見えてますか！？！？</h1>
  </body>
</html>
```

じゃあここまでで結果を見てみましょう！

```shell
pnpm run dev
```

こうなっていれば成功です！
![](/images/frontend-framework-1/sample1.png)

## TypeScriptの設定

設定といっても、開発サーバーでViteはTypeScriptを勝手にコンパイルしてくれます。
つまり、`src/main.ts`を作成して、index.htmlにパスを書くだけで動いてくれるんです！
`Vite supports import .ts files out of the box`とドキュメントにあります！(参考：https://vite.dev/guide/features#typescript)

つまりこういう事です！

念のためtypescriptを入れて……

```shell
pnpm add -D typescript
```

```ts
// src/main.ts(せっかくなのでtsの機能をつかって実験です！)
type A = {
  a: string;
  b: number;
};

const a: A = { a: "my name is Anzu", b: 0 };

console.log("Hello world", a.a);
```

```html
<!-- index.html -->
<!doctype html>
<html lang="ja">
  <head>
    <meta charset="UTF-8" />
  </head>
  <body>
    <h1>見えてますか！？！？</h1>
    <script type="module" src="/src/main.ts"></script>
  </body>
</html>
```

ついでに、tsconfigも設定しておきましょう！

```json
// tsconfig.json
{
  "compilerOptions": {
    "target": "es2022",
    "useDefineForClassFields": true,
    "lib": ["es2022", "DOM", "DOM.Iterable"],
    "module": "es2022",
    "skipLibCheck": true,

    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",

    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src"]
}
```

こうなっていれば成功です！
![](/images/frontend-framework-1/sample6.png)

ちなみに……
ここまでの処理を整理すると以下のようなことが起きています！

1. Viteの開発サーバーはアクセスに対して`index.html`を返す
2. ブラウザは`<script>`を見つけ、`http://localhost:5173/src/main.ts`にアクセスする
3. Viteの開発サーバーはブラウザが解釈できるよう、esbuildを使ってjsにトランスパイルしてレスポンスする

開発者ツールのネットワークタブで、`main.ts`ファイルを見てみると、トランスパイルされたJavaScriptが返ってきてることがよくわかりますよ。
![](/images/frontend-framework-1/sample2.png)

:::details 💡 補足：type="module"ってなに？
🐶 **シッポ**

> あの、後輩ちゃん。質問があるんだけど

🐱 **アンズ**

> おっ。感心ですね。
> 素直に質問するのは大事ですよ。ポイントあげちゃいます！

🐶 **シッポ**

> さっきの、`type="module"`ってなに？
> たまに見かけるんだけど、実は良くわかってなくて

🐱 **アンズ**

> あー、これですね。たしかに説明してなかったですね。
> ちなみにこれ、ちょっと難しい話なので理解できなくても大丈夫です！

> JavaScriptって、大昔はファイルの分割なんてしてなかった……というか、必要なかったんですよ。

🐶 **シッポ**

> 聞いたことあるかも。せいぜいちょっと動きを付ける程度で、ここまでリッチなwebアプリを作らなかったからだっけ？

🐱 **アンズ**

> そうです！さすがわかってるじゃないですか🎵
> それで、最近になって、「やっぱりブラウザでもファイル分割したい〜」ってなって生まれたのが、ESModules(ESM)なんですよ。
> せんぱい、ファイル分割して開発してたアプリをブラウザで動かしたい時、普通ならどうしますか？

🐶 **シッポ**

> そりゃ、webpackとかで１ファイルにまとめるんじゃないの？

🐱 **アンズ**

> そうです！それが王道の手法でした。
> 一方ESMでは、ブラウザでも`import hoge.js`を見つけると、`/hoge.js`にリクエストを飛ばして、importを追いかけてくれるんですよ！

🐶 **シッポ**

> それはすごいね。

🐱 **アンズ**

> とはいえ、webpackなどのバンドラが不要になるわけじゃないのでそこは注意ですよ！
> それじゃあ最初の質問に戻りますけど、`type=module`は、「指定したJavaScriptをESMとして扱ってね」という指示になります。
> これで、開発中にいちいちバンドルしなくても、ESMで必要なファイルだけブラウザがリクエストを投げてくれるようになるんです。
> Viteは開発サーバーではESMを積極的に使って行く方針なんですよ！

:::

:::details 💡 補足：じゃあ型チェックもViteがやってくれるの？
🐶 **シッポ**

> あと、もう１つ質問があって、

🐱 **アンズ**

> ２つ目ですか！
> せんぱいが興味津々で聞いてくれて私も嬉しいです！
> 何でも聞いてくださいっ！

🐶 **シッポ**

> ViteがTypeScriptを勝手にトランスパイルしてくれるって言ったでしょ？
> じゃあもう`tsc --noEmit`とかで型チェックは要らなくなって、Viteが代わりのコマンドを用意してくれてるってこと？

🐱 **アンズ**

> あっ、たしかにそう思いますよね！
> それは実はやってくれてないんです！
> たぶん、せんぱいが言いたいのは、「`vite typecheck`みたいなコマンドがあるんじゃないか？」ってことですよね？

🐶 **シッポ**

> そうそう。トランスパイルしてくれてるなら、そこもやってくれてるほうが自然だよなって思ったんだけど。

🐱 **アンズ**

> そうなんですよ〜。たしかにそう考えるのが自然ですよね。
> でも、Viteが`ts`->`js`に使ってるesbuildというトランスパイラは、Goで書かれていてとっても早いんですが！早いんですが！型は全部無視してしまう設計になっているので、型チェックまではできないんです。

🐶 **シッポ**

> なるほど。Vite側だけというよりは、esbuildの制約としても型チェックができないんだね。

🐱 **アンズ**

> そうなります！
> なので、型チェックをしたいときはこれまで通り`tsc --noEmit`を叩くか、エディタに搭載されてる言語サーバーを使うのが一般的なんですよ！

:::

:::message

Viteの開発サーバーでは`.ts`ファイルへのアクセスを、Viteがesbuildを使って勝手にトランスパイルしてくれる

:::

## Reactの設定

実はこっちも、`.jsx and .tsx files are also supported out of the box. `と書いてありました！
やってみましょうか！

と、その前にまずは必要なパッケージを入れましょう💪

```shell
pnpm add react react-dom
pnpm add -D @types/react @types/react-dom
```

そしたら、

```tsx
// main.tsx
import { createRoot } from "react-dom/client";

const root = createRoot(document.getElementById("app")!);
root.render(<p>Reactより、わたしです！！</p>);
```

```html
<!-- Reactからレンダリングするので中身を空っぽにしました --->
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

実行すると、
![](/images/frontend-framework-1/sample3.png)

問題なく表示されてますね！

ブラウザに表示されるまでのフローは先程のTypeScriptまでの部分と同じです！
`tsx` -> `ts` -> `js`と、Viteの中にいるesbuildが変換してくれているんですよ！

開発者ツールの`main.tsx`も見てみましょうか。

![](/images/frontend-framework-1/sample4.png)

きちんとjsにまでコンパイルされてるのがわかりますよね！？

:::message

Viteの開発サーバーではJSX,TSXもesbuildが勝手にトランスパイルしてくれる

:::

## カウンターアプリで動作確認

🐱 **アンズ**

> どうですか？
> 結構簡単に動かせましたよね！

🐶 **シッポ**

> Vite、思ったより高機能でびっくりしたよ。
> devコマンドもあるし、TypeScriptもReactもそのまま動く。
> これVite自体がフレームワークみたいなものじゃん

🐱 **アンズ**

> そうなんですよ。
> でもルーティングだったりエラーハンドリングはやってくれないので、そういう「アプリの機能としてあったほうがいいもの」はフレームワーク側で用意してあげる必要がありますよ！

🐶 **シッポ**

> うーん、まだちょっと境目が良く分からないね
> Viteも十分高機能だし、この上のフレームワークって意味あるの……？
> ちょっとラップするだけにならない？？

🐱 **アンズ**

> そうですねえ……
> もうしばらく進めてみましょうか！
> 自前フレームワークを動かすところまで行けばきっと景色が変わるはずです！

🐱 **アンズ**

> と、その前にですね
> せんぱい、この設定でちゃんと動いてると思いますか？

🐶 **シッポ**

> ……？
> 動いてるんじゃないの？

🐱 **アンズ**

> はい！たぶん動いてます！

🐶 **シッポ**

> ええ？

🐱 **アンズ**

> でも今のところ、ちょっとしたテキストを表示しただけですよ。
> これくらいなら、何かミスってても動きそうじゃないです？

🐶 **シッポ**

> そうかも……

🐱 **アンズ**

> なので、最低限インタラクティブにしましょう。
> さくっとカウンターアプリだけ作って動作確認しましょう！
> せんぱい、「ボタンをクリックしたら数値が増えるだけ」でいいので、それを満たすように`counter.tsx`を作って、`main.tsx`を編集してみてください！

🐶 **シッポ**

> まかせてよ。これでいい？

```tsx
//src/pages/counter.tsx
import { useState } from "react";

export const Counter = () => {
  const [count, setCount] = useState(0);

  const onButtonClicked = () => {
    setCount((prev) => prev + 1);
  };

  return (
    <div>
      <p>現在の値は{count}です</p>
      <button onClick={onButtonClicked}>インクリメント</button>
    </div>
  );
};
```

```tsx
// main.tsx
import { createRoot } from "react-dom/client";
import { Counter } from "./pages/counter";

const root = createRoot(document.getElementById("app")!);
root.render(<Counter />);
```

🐱 **アンズ**

> そうですね……
> はい！大丈夫だと思います！
> じゃあ実行してみましょう！

実行結果
![](/images/frontend-framework-1/sample5.png)

🐶 **シッポ**

> 動いてる！良かったぁ……
> これでミスったら自信なくしてたところだったよ

🐱 **アンズ**

> 自信なかったんですね笑
> ともかく、これでちゃんと動いてることが確認できましたね！

ではこれで、Vite × TypeScript × Reactのミニマム構成ができたことになりますよね！
次回はさらに、SSR・できたらReact-Routerの導入まで進めますよ！
