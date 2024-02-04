---
id: "nextjs-ssg-ssr-isr"
title: " Next.jsのSSG/SSR/ISRをMicroCMSのブログテンプレートで理解する"
description: "MicroCMSのブログテンプレートを使用して、Next.jsのSSG/SSR/ISRの挙動を理解する。"
emoji: "💡"
category: "Tips"
tags: ["Next.js","React"]
createdAt: "2024-02-04 22:20:41"
updatedAt: "2024-02-04 22:20:41"
---
## 概要
この記事は、Next.js14のAppRouterを使用した際のSSG/SSR/ISRの動きを理解する為の内容になります。

分かりやすいように、MicroCMSが提供するブログテンプレートを使用して、Next.jsのSSG/SSR/ISRの動きを確認していきます。

この記事では、SSG/SSR/ISRの動きを確認したいだけなので、SSG/SSR/ISRの詳細については、以下の記事が参考になりました！
* [もう迷わないNext.jsのCSR/SSR/SSG/ISR](https://zenn.dev/a_da_chi/articles/105dac5573b2f5)
* [あなたはどのレンダリングパターンを採用しますか？【CSR/SSR/SSG/ISRを図解で解説】](https://www.youtube.com/watch?v=QckiJezDS_E&t=21s)

それでは、実際に確認していきましょう！

## MicroCMSの事前準備
### ブログテンプレートの作成
まず、以下のURLから、MicroCMSにログインします。
https://app.microcms.io/

「テンプレートから選ぶ」を選択します。
![microcms](https://github.com/mugi-tech/blog-contents/blob/main/mugi-notebook-contents/nextjs-ssg-ssr-isr/images/microcms.png?raw=true)

シンプルなブログを選択します。
![simpleblog](https://github.com/mugi-tech/blog-contents/blob/main/mugi-notebook-contents/nextjs-ssg-ssr-isr/images/simpleblog.png?raw=true)

以降、手順に沿って、GitHubにシンプルなブログのソースコードを配置します。

### コンテンツの作成
ブログテンプレートから作成すると以下の3つのAPIが用意されていると思います！
* ライター
* タグ
* ブログ

これらのコンテンツに以下のようにデータを登録していきます。
データは、定義された項目が設定されていれば、任意のデータで構いません！
#### ライターのデータ
ここは、適当でいいです。
![writer](https://github.com/mugi-tech/blog-contents/blob/main/mugi-notebook-contents/nextjs-ssg-ssr-isr/images/writer.png?raw=true)

#### タグのデータ
SSG/SSR/ISRの3つを設定してください。
![tag](https://github.com/mugi-tech/blog-contents/blob/main/mugi-notebook-contents/nextjs-ssg-ssr-isr/images/tag.png?raw=true)

#### ブログのデータ
タグには、とりあえずSSGのタグを設定しておいてください。
![blogs](https://github.com/mugi-tech/blog-contents/blob/main/mugi-notebook-contents/nextjs-ssg-ssr-isr/images/blogs.png?raw=true)
とりあえずはこれで事前準備OKです！
それでは、実際にそれぞれの動きを確認していきましょう！

## SSGの動きを確認
まずは、`npm run dev`コマンドを実行して、ローカル環境で先ほど登録したブログデータが表示されるか確認してみます！

SSGタグを選択すると以下のようなデータが確認できます。
![SSG](https://github.com/mugi-tech/blog-contents/blob/main/mugi-notebook-contents/nextjs-ssg-ssr-isr/images/ssg.png?raw=true)

これは、デフォルトでSSRモードになっているので、MicoroCMSの取得APIでデータを取得している為、表示されています。

では、この[tagId]のpage.tsxだけ、SSGモードにする為、以下のようにソースコードを修正してみます。
※既存のPage関数のソースコードはコメントアウトしておいてください。SSRとISRの時に使います。

```js
export async function generateStaticParams() {
  const tagList = await getTagList();
  const params = tagList.contents.map((tag: Tag) => ({
    tagId: tag.id,
  }));
  return params;
}

export default async function Page({ params }: Props) {
  const tagId = params.tagId;
  const data = await getList({
    limit: LIMIT,
    filters: `tags[contains]${tagId}`,
  });
  return (
    <>
      <ArticleList articles={data.contents} />
      <Pagination totalCount={data.totalCount} basePath={`/tags/${tagId}`} />
    </>
  );
}
```
SSGで動作させるには、事前にレンダリングしたページが必要です。
その為には、全てのタグに紐づくブログデータを事前に取得し、レンダリングする必要があります。

そこで、`generateStaticParams()`を使用します。
これにより、事前に[tagId]に入るタグIDを取得し、ページとしてレンダリングできるようにパラメータを取得しておく事で、SSGで動作させることが可能になります。

ソースコードの修正ができたら、`npm run build`を行い、tags/[tagId]の部分がSSGになっている事を確認します。
```
Route (app)                                Size     First Load JS
┌ ○ /                                      465 B          88.2 kB
├ ○ /_not-found                            0 B                0 B
├ λ /articles/[slug]                       544 B          88.3 kB
├ ○ /favicon.ico                           0 B                0 B
├ λ /p/[current]                           471 B          88.2 kB
├ λ /search                                465 B          88.2 kB
├ λ /search/p/[current]                    471 B          88.2 kB
├ ● /tags/[tagId]                          471 B          88.2 kB
├   ├ /tags/5t2o-rybk
├   ├ /tags/99wku3a8datb
├   └ /tags/8-kmadxvfj
└ λ /tags/[tagId]/p/[current]              471 B          88.2 kB
+ First Load JS shared by all              77.3 kB
  ├ chunks/2443530c-ee4addf37741a97b.js    50.5 kB
  ├ chunks/488-0609857d86a657e1.js         24.7 kB
  ├ chunks/main-app-05382dc900a6a006.js    215 B
  └ chunks/webpack-80cefc145faf0c58.js     1.81 kB

Route (pages)                              Size     First Load JS
─ ○ /404                                   181 B          74.9 kB
+ First Load JS shared by all              74.7 kB
  ├ chunks/framework-8883d1e9be70c3da.js   45.1 kB
  ├ chunks/main-97fa5c9906ad70d7.js        27.6 kB
  ├ chunks/pages/_app-b555d5e1eab47959.js  195 B
  └ chunks/webpack-80cefc145faf0c58.js     1.81 kB

λ  (Server)  server-side renders at runtime (uses getInitialProps or getServerSideProps)
○  (Static)  automatically rendered as static HTML (uses no initial props)
●  (SSG)     automatically generated as static HTML + JSON (uses getStaticProps)
```
想定通りにSSGになってますね。

では、この状態で`npm run start`を実行して、SSGタグを選択して、追加したブログデータがあるか確認します。
![SSG2](https://github.com/mugi-tech/blog-contents/blob/main/mugi-notebook-contents/nextjs-ssg-ssr-isr/images/ssg2.png?raw=true)
きちんと表示されますね。
SSGなので、先ほどよりも読み込み時間が早かった感覚があります。

SSGである事を確認する為、試しに新しくデータを追加してデータがブラウザに表示されないかどうかを確認してみます。
![SSG3](https://github.com/mugi-tech/blog-contents/blob/main/mugi-notebook-contents/nextjs-ssg-ssr-isr/images/ssg3.png?raw=true)

リロードしても表示されませんね。
![SSG2](https://github.com/mugi-tech/blog-contents/blob/main/mugi-notebook-contents/nextjs-ssg-ssr-isr/images/ssg2.png?raw=true)
それも当然で、SSGで動かしているので、ビルド時点の内容でしか表示されない為、新規で追加した分は表示されないというわけです！

なるほど、、
SSGの動きは確認できたので、次はSSRの動きを確認してみます！

## SSRの動きを確認
次は、SSRの動きを確認する為、先ほど修正したソースコードを削除し、コメントアウトした部分を元に戻します。

さらに、`export const revalidate = 60;`の部分は、`export const revalidate = 0;`に変更してください。

※ここを0にしないと、MicroCMSのAPIのキャッシュが効いているせいか上手く動かないので、強制的にSSRにするため、`export const revalidate = 0;`にしてます。。

```js
export const revalidate = 0; 

export default async function Page({ params }: Props) {
  const { tagId } = params;
  const data = await getList({
    limit: LIMIT,
    filters: `tags[contains]${tagId}`,
  });
  const tag = await getTag(tagId);
  return (
    <>
      <ArticleList articles={data.contents} />
      <Pagination totalCount={data.totalCount} basePath={`/tags/${tagId}`} />
    </>
  );
}
```
ソースコードの修正ができたら、SSRのタグをつけたブログデータを新規で追加します。

その後、`npm run build`を行い、tags/[tagId]の部分がSSRになっている事を確認します。
```
Route (app)                                Size     First Load JS
┌ ○ /                                      478 B          88.2 kB
├ ○ /_not-found                            0 B                0 B
├ λ /articles/[slug]                       543 B          88.3 kB
├ ○ /favicon.ico                           0 B                0 B
├ λ /p/[current]                           478 B          88.2 kB
├ λ /search                                478 B          88.2 kB
├ λ /search/p/[current]                    478 B          88.2 kB
├ λ /tags/[tagId]                          476 B          88.2 kB
└ λ /tags/[tagId]/p/[current]              476 B          88.2 kB
+ First Load JS shared by all              77.3 kB
  ├ chunks/2443530c-ee4addf37741a97b.js    50.5 kB
  ├ chunks/488-0609857d86a657e1.js         24.7 kB
  ├ chunks/main-app-05382dc900a6a006.js    215 B
  └ chunks/webpack-70c0de71f6fd9094.js     1.79 kB

Route (pages)                              Size     First Load JS
─ ○ /404                                   181 B          74.9 kB
+ First Load JS shared by all              74.7 kB
  ├ chunks/framework-8883d1e9be70c3da.js   45.1 kB
  ├ chunks/main-97fa5c9906ad70d7.js        27.6 kB
  ├ chunks/pages/_app-b555d5e1eab47959.js  195 B
  └ chunks/webpack-70c0de71f6fd9094.js     1.79 kB

λ  (Server)  server-side renders at runtime (uses getInitialProps or getServerSideProps)
○  (Static)  automatically rendered as static HTML (uses no initial props)
```
SSRになってますね。

では、この状態で`npm run start`を実行して、SSRタグを選択して、追加されたブログデータがあるか確認します。
![SSR1](https://github.com/mugi-tech/blog-contents/blob/main/mugi-notebook-contents/nextjs-ssg-ssr-isr/images/ssr1.png?raw=true)
※うまく表示されない場合は、.nextフォルダを削除して、再度`npm run build`してみてください。

次にSSRで動いているかを確認する為、新規でブログデータを追加して、追加したデータがブラウザに表示されるかどうかを確認します。
![SSR2](https://github.com/mugi-tech/blog-contents/blob/main/mugi-notebook-contents/nextjs-ssg-ssr-isr/images/ssr2.png?raw=true)
今度は、追加されてますね！
これは、SSRで動いている為、接続の度に毎回取得APIが動きます。
その結果、常に最新の情報がブラウザに表示される形ですね！

最後にISRの動きを確認してみましょう！

## ISRの動きを確認
最後に、ISRの動きを確認する為、`export const revalidate = 0;`の部分を、`export const revalidate = 60;`に変更してください。

この設定をすることによって、60秒の間はキャッシュされたレスポンスが返却される為、その間はSSGのような動きになり、30秒を過ぎるとSSRの動きになります。

ソースコードの修正ができたら、ISRのタグをつけたブログデータを新規で追加します。

その後、`npm run build`を行い、tags/[tagId]の部分がISR(表記はSSR)になっている事を確認します。
```
Route (app)                                Size     First Load JS
┌ ○ /                                      475 B          88.2 kB
├ ○ /_not-found                            0 B                0 B
├ λ /articles/[slug]                       543 B          88.3 kB
├ ○ /favicon.ico                           0 B                0 B
├ λ /p/[current]                           475 B          88.2 kB
├ λ /search                                475 B          88.2 kB
├ λ /search/p/[current]                    475 B          88.2 kB
├ λ /tags/[tagId]                          463 B          88.2 kB
└ λ /tags/[tagId]/p/[current]              475 B          88.2 kB
+ First Load JS shared by all              77.3 kB
  ├ chunks/2443530c-ee4addf37741a97b.js    50.5 kB
  ├ chunks/488-0609857d86a657e1.js         24.7 kB
  ├ chunks/main-app-05382dc900a6a006.js    215 B
  └ chunks/webpack-7bcf537c6aab09e8.js     1.8 kB

Route (pages)                              Size     First Load JS
─ ○ /404                                   181 B          74.9 kB
+ First Load JS shared by all              74.7 kB
  ├ chunks/framework-8883d1e9be70c3da.js   45.1 kB
  ├ chunks/main-97fa5c9906ad70d7.js        27.6 kB
  ├ chunks/pages/_app-b555d5e1eab47959.js  195 B
  └ chunks/webpack-7bcf537c6aab09e8.js     1.8 kB

λ  (Server)  server-side renders at runtime (uses getInitialProps or getServerSideProps)
○  (Static)  automatically rendered as static HTML (uses no initial props)
```
ISR(表記はSSR)になってますね。

では、この状態で`npm run start`を実行して、ISRタグを選択して、追加されたブログデータがあるか確認します。
![ISR1](https://github.com/mugi-tech/blog-contents/blob/main/mugi-notebook-contents/nextjs-ssg-ssr-isr/images/isr1.png?raw=true)

次にISRで動いているかを確認する為、新規でブログデータを追加して、追加したデータがブラウザに表示されるかどうかを確認します。
![ISR1](https://github.com/mugi-tech/blog-contents/blob/main/mugi-notebook-contents/nextjs-ssg-ssr-isr/images/isr1.png?raw=true)

ISRの為、やはりすぐには追加されないですね。
これは、60秒間キャッシュするような設定になっているからです。
60秒待って、再度更新してみます。
![ISR2](https://github.com/mugi-tech/blog-contents/blob/main/mugi-notebook-contents/nextjs-ssg-ssr-isr/images/isr2.png?raw=true)
今度は、追加されてますね！
ISRで動いている事が確認できました！

## まとめ
今回は、SSG/SSR/ISRの動きをMicroCMSにブログテンプレートを用いて、確認してみました。
ブログサイトのような静的サイトならば、SSRではなくSSG/ISRで十分そうですね。
今度、ブログを作る際はSSGかISRで作ってみようと思います！