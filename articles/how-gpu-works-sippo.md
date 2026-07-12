---
title: "GPUがモニターに色を描画するまでを、擬似コードで理解する備忘録"
emoji: "🖲️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["gpu", "3dcg", "shader", "webgl", "初心者"]
published: true
---

:::message
この記事について

友人との会話でGPUの話にまったくついていけなかったので、調べながら自分用に軽くまとめた備忘録です。
擬似コードで理解するようにしたら、なかなかわかりやすかった気がするので、雑に公開します。

この記事でつかっているコードブロックは、すべて擬似コードです。
動きません。不正確です。
でも概念的な理解の助けになると思います。
:::

```ts
// 共通の基本型
type vec2 = [number, number]; // UV座標など (x, y)
type vec3 = [number, number, number]; // 法線ベクトルなど (x, y, z)
type ColorRGB = [number, number, number]; // 色 (R, G, B)
```

- GPUの呼び出し、つまり`draw call`は、オブジェクトごと、メッシュごとに呼ばれる。
- 例えば、`humanModel.draw()`を呼んだとき、CPU側では以下のような処理が走る

```ts
// ==========================================
// １：シェーダー初期化
// ==========================================

// 1. 自前で書いたシェーダーの「文字列」を読み込む
const myVertexShaderSource: string = "void main() { ... }";
const myFragmentShaderSource: string = "void main() { ... }";

// 2. GPUの中に空の「プログラムオブジェクト（シェーダーを載せる席）」を作る
const myShaderProgram = GPU.createShaderProgram();

// 3. GPUにコードを送りつけて、GPU内部でコンパイル・リンク（合体）させる
// ※これはめちゃくちゃ重い処理
GPU.compileAndLinkShader(
  myShaderProgram,
  myVertexShaderSource,
  myFragmentShaderSource,
);

// ==========================================
// 2：draw(1)
// ==========================================

// 最初に「全身の頂点データ」を1回だけドンとセットする
// （※ここで頂点はとにかく全部渡しておく）
GPU.bindVertexBuffer(character.vertexBuffer);

// --- 1回目のDraw (顔の描画) ---
// 1. テクスチャ切り替え
GPU.setTextureSlot(0, faceAlbedoTexture);
GPU.setTextureSlot(1, faceNormalTexture);
GPU.useShaderProgram(myShaderProgram); // シェーダーもマテリアルの一部

// 2. 「顔」のindexBufferだけを渡してDrawを呼ぶ！
// indexBuffer: VertexBufferのインデックス３つをまとめたもの[[0,1,2],[1,2,3],...]のようになっている。
// この３つの頂点が１つのメッシュを構成する
// GPUに渡す段階で[Vertex,Vertex,...]のようにすると、同じ頂点を重複して送ることになって重いので、メッシュはindexBufferで表現する。
GPU.drawElements(character.faceIndexBuffer);
// ➔ GPUは全身の頂点から、顔の頂点だけを拾い上げて三角形を作り、顔テクスチャを貼る

// ==========================================
// 3：draw(2)
// ==========================================

// 1. テクスチャ切り替え（これで複数テクスチャを描画することができる）
GPU.setTextureSlot(0, clothesTexture);
GPU.setTextureSlot(1, clothesNormalTexture);

// 2. 「服」のindexBufferだけを渡してDrawを呼ぶ！
GPU.drawElements(character.clothesIndexBuffer);
// ➔ GPUは同じ全身の頂点から、服の頂点だけを拾い上げて三角形を作り、服テクスチャを貼る
```

:::details webGL,openGLの正体

ちなみに、ここで

```ts
GPU.setTextureSlot(1, faceNormalTexture);
```

とかのインターフェースを提供しているのが、グラフィックAPIと呼ばれるWebGL,OpenGLです。

ネイティブ環境で使われるのがOpenGL
Webから使われるのがWebGL

【ちなみに&ちなみに】
より細かくメモリ配置などを制御できるようなインターフェースを提供しているのが以下。

- Vulkan: オープン規格
- DirectX: Microsoft
- Metal: Apple

:::

で、`drawElements`の先では以下の情報を受け取ることになる。

```ts

// ポリゴンの頂点が順不同で入ってくる。色情報は頂点が持っている
type Vertex{
    position: vec3; // [number,number,number]
    color:Color;
    uv: vec2; // テクスチャのどこを参照するか([0-1,0-1]) . [number,number]
    normal: vec3; // 法線。本来頂点に法線は定義できないけど、その頂点を包含する面すべての法線を平均して作る
}

List < Vertex > vertexBuffer; // オブジェクトごとに作られる
indexBuffer = [
  [0, 1, 2], // vertexのインデックス３つ組。これがMeshを形成する単位
  [1, 2, 3],
];
```

これを、プリミティブアセンブリと呼ばれる工程で以下の情報に再構成される。

```ts
type Primitive = {
  vertex0: Vertex;
  vertex1: Vertex;
  vertex2: Vertex;
};
```

- これをVertexシェーダーが処理して以下の情報になる

```ts

type Vertex{
    position:vec2;// 画面上の位置 ← changed!!
    color:Color;
    uv: vec2; // テクスチャのどこを参照するか([0-1,0-1])
    normal: vec3; // 法線
    depth:float; // カメラからの深さ ← new!!
}

type Primitive = {
    vertex0: Vertex;
    vertex1: Vertex;
    vertex2: Vertex;
}

```

- これをラスタライザが処理する
- ラスタライザは、「それぞれの三角形が、画面上のどのピクセルを包含するか」を調べ(ピクセル包含テスト)、
- 包含するピクセルについては、そのPrimitiveを構成する頂点からの距離に応じて色を補完する
- ここでは、三角形の内部にあるピクセル数分だけデータ(フラグメント)がつくられる。
- ここで、色の重ね合わせは行われていないため、フラグメント数 > 画面ピクセル数　となるケースも、かなりあることに注意

```ts
type Fragment = {
  pixielPosition: vec2; // 画面上のピクセル位置

  // 以下、本来は頂点が持っていた情報だが、ラスタライザが補完して、フラグメントの情報に変える
  depth: float; // カメラからの距離
  uv: vec2; // テクスチャのどこを使うか
  color: Color;
  normal: vec3;
};

type RasterizerOutput = Array<FragmentData>;
```

ここで、CPU側からの呼び出し時に渡されてる(以下のようなことを、draw callのときにしている。冒頭の擬似コード参照)、情報を参照して、texture情報を取り出す

```ts
GPU.setTextureSlot(1, faceNormalTexture);
```

```ts
// 画像からUV座標を使ってデータを取り出す「テクスチャ」のインターフェース
// このテクスチャの1dotのことをテクセルと呼ぶ
// つまり、uvとはピクセルからテクセルの対応づけになる
interface Texture2D<T> {
  sample(uv: vec2): T;
}

// --- 各マップの型定義 ---

// ① ノーマルマップ: XYZの「向き」をRGBの3色で表現しているため、vec3 が返る
type NormalMap = Texture2D<vec3>;

// ② アルベドマップ: 純粋な「色」のため、ColorRGB (透過がある場合はRGBA) が返る
type AlbedoMap = Texture2D<ColorRGB>;

// ③ メタリックマップ: 金属度合い（0.0: 非金属 ～ 1.0: 金属）なので、1つの数値(float)が返る
type MetallicMap = Texture2D<float>;

// ④ ラフネスマップ: ザラザラ度合い（0.0: ツルツル ～ 1.0: ザラザラ）なので、1つの数値(float)が返る
type RoughnessMap = Texture2D<float>;

// ⑤ AOマップ: 遮蔽による暗さ（0.0: 完全に影 ～ 1.0: 影なし）なので、1つの数値(float)が返る
type AOMap = Texture2D<float>;

// [補足]このfloat三連発は効率が悪いから、これを１つのTexture2D<ColorRGB>にまとめて、RGBにそれぞれ突っ込むのをチャンネルパッキングという

// ⑥ エミッションマップ: 発光する「色」と強さなので、ColorRGB が返る
type EmissionMap = Texture2D<ColorRGB>;

// 外部からシェーダーに渡されるマテリアル（素材）データ
type MaterialData = {
  shaderProgram: ShaderProgramObject;

  albedoTex: AlbedoMap;
  normalTex: NormalMap;
  metallicTex: MetallicMap;
  roughnessTex: RoughnessMap;
  aoTex: AOMap;
  emissionTex: EmissionMap;
};
```

- これをフラグメントシェーダーが処理する

```ts
function fragmentShader(
  input: FragmentInput,
  material: MaterialData,
): ColorRGB {
  // 1. すべてのマップから、このピクセル(UV座標)に該当するデータを抽出(sample)する
  const albedo: ColorRGB = material.albedoTex.sample(input.uv);
  const normal: vec3 = material.normalTex.sample(input.uv);
  const metallic: float = material.metallicTex.sample(input.uv);
  const roughness: float = material.roughnessTex.sample(input.uv);
  const ao: float = material.aoTex.sample(input.uv);
  const emission: ColorRGB = material.emissionTex.sample(input.uv);

  // 2. 抽出したデータを元に、物理ベースの光の計算（PBR）を行う
  // （※計算式の詳細は省略しますが、先ほどのデータが引数として渡されます）
  const finalColor = calculatePBR(
    shader,
    albedo,
    normal,
    metallic,
    roughness,
    ao,
    emission,
  );

  return finalColor;
}
```

最終的な出力は、画面のHeight×Widthと同じ大きさのframeBufferに行います。

```ts
const frameBuffer: Array<PixelHeight, PixelWidth, ColorRGB>;
// zBuffer:frameBufferに現在書き込まれてる色の、Fragment.depth
// 同じピクセルに対して複数のフラグメントが見つかったときは、このzBufferを参照して、浅い方を残す
// (だって、depthsが深い方は、重なってて、本来は見えていないはずだから)
const zBuffer: Array<PixelHeight, PixelWidth, float>;

function processFragment(fragment, material) {
  // いま、描画されているフラグメントより、自分のほうが手前にあるとき
  if (zBuffer[fragment.pixelPosition] > fragment.depth) {
    zBuffer[fragment.pixelPosition] = fragment.depth;

    // Early-Z Cullingと呼ばれる技術: 先にdepthを比較してからフラグメントシェーダーを走らせる
    const finalColor = fragmentShader(fragment, material);
    frameBuffer[fragment.pixelPosition] = finalColor;
  }
}
```

ここで作られたframeBufferが、そのままモニターなどに出力されます。

## 自分で書くことのできるシェーダー

#### 1. 頂点シェーダー

- 要するに、頂点の位置をいじる
- 頂点をサイン波などをつかってブレさせることで、風で揺れてるような挙動が実現できる。

#### 2. フラグメントシェーダー

- 光のあたり具合が0.5以上ならめっちゃ白く（トゥーンシェーダー）などする
- 「光のあたり具合」は、normal mapとかから、数学的に計算できる
