---
title: "Next.js（Vercel）でHydration failedを予防したい？"
emoji: "💧"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [nextjs, react]
published: true
---

# TL; DR

Next.jsのDevサーバのタイムゾーンをUTCにしちゃおう！

```json:package.json
{
  "scripts": {
    "dev": "TZ='Etc/UTC' next dev"
  }
}
```

# 症状
Next.jsでは、React 18の [Hydrationという機能](https://nextjs.org/docs/messages/react-hydration-error) によって、サーバサイドで生成されたHTML構造とクライアントでレンダリングされたHTMLの構造が異なる場合に

```plain:Minified React Error #418
Hydration failed because the initial UI does not match what was rendered on the server.
```

```plain:Minified React Error #423
There was an error while hydrating. Because the error happened outside of a Suspense boundary, the entire root will switch to client rendering.
```

```plain:Minified React Error #425
Text content does not match server-rendered HTML.
```

のようなエラーが発生します。

私の場合は、これがローカルのDevサーバでは発生しないのに本番でこのエラーが出て困っていました。

## Hydrationについて
Next.jsではページロードを高速化するためにサーバサイドで予めHTMLの大枠を生成しておき、クライアントがページ（HTML）にアクセスした直後はインタラクティブな要素がまだ動かない状態になっているようです。
JSを読み込み終わって実行できるようになったら、インタラクティブな要素にイベントリスナを追加することでページの準備が整います。
この、事前に読み込んだHTMLにJSでイベントリスナなどを付与していくはたらきのことを **Hydration** と呼びます。

_参考: [Pre-rendering and Data Fetching](https://nextjs.org/learn/basics/data-fetching/pre-rendering)_

これは親切にも、特に設定することなく利用できる機能です。
意識することなく使えてしまっているために、知らないエラーが出て戸惑う人も大勢いそうではあります…。

# 原因
現在時刻をコンポーネント（render関数）内に直接記述するなど、タイムゾーン依存で表示が変わるような実装をしていました。

```tsx:イメージ
const component = () => {
  return (
    <div>
      {DateTime.fromJSDate(new Date()).toFormat(
        'yyyy-MM-dd HH:mm:ss'
      )}
    </div>
  )
}
```

Vercelにデプロイする場合、サーバサイドの機能はServerless Functionsで実行されることになります。そしてHydrationのために事前にHTMLを生成する処理も例外ではありません。
Serverless Functions上ではタイムゾーンがUTCに設定されており、UTC以外のタイムゾーン設定のコンピュータからアプリケーションにアクセスした場合はタイムゾーン依存の表示に差が生じ、Hydration failedのエラーが発生します。

しかし、Devサーバでは当然サーバサイドもローカルのマシンで動いているので、クライアントとサーバが同じシステムのタイムゾーンを使うこととなり、開発環境では事前にサーバサイドで生成されたHTMLとクライアントで実際にレンダリングされたHTMLに差が生じないため、Hydration failedのエラーは本番環境でのみ発生することとなります。

:::message
- `a` タグの中で `a` タグを使う
- `p` タグの中で `p` タグを使う
- `p` タグの中で `div` タグを使う

など、HTMLのルールを犯していてこのエラーが出ることもあるのですが、そういう場合はローカルでも同じエラーが出るので比較的簡単に原因が特定できるはずです。
:::

# 対策

Next.jsのDevサーバのタイムゾーンをUTCに設定すれば、開発環境と本番環境（Vercel）との差異が小さくなるのでは？と思ったわけであります。

Devサーバのタイムゾーンは `TZ` 環境変数で簡単に設定できます。

```json:package.json
{
  "scripts": {
    "dev": "TZ='Etc/UTC' next dev"
  }
}
```

かなり限定的なエラーと対応ではありますが、本番環境でminifyされたReactが吐くエラーよりも開発環境の吐くエラーの方がスタックトレースなど充実していて解決が楽なので、同じエラーに悩む方の一助になればと思い共有します。

# ちなみに…
上に示したイメージのコードは下のように修正することでエラーを解消できます（`useEffect` はクライアントサイドでのみ実行されるため）。

```tsx:イメージ（修正前）
const component = () => {
  return (
    <div>
      {DateTime.fromJSDate(new Date()).toFormat(
        'yyyy-MM-dd HH:mm:ss'
      )}
    </div>
  )
}
```

```tsx:イメージ（修正後）
const component = () => {
    const [formattedDate, setFormattedDate] = useState<string>()
    
    useEffect(() => {
      setFormattedDate(
        DateTime.fromJSDate(new Date()).toFormat('yyyy-MM-dd HH:mm:ss')
      )
    }, [])

  return (
    <div>{formattedDate}</div>
  )
}
```