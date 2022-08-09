---
title: "Next.jsでプロジェクトテンプレート？"
emoji: "💭"
type: "tech"
topics:
  - "nextjs"
  - "react"
  - "クソ記事"
published: true
published_at: "2021-02-17 19:17"
---

# TL;DR
`npx create-next-app --example <リポジトリURL> <新規プロジェクト名>`

# 本文
既出 or 常識 だったら大変クソ記事なのですが、個人的な結論を見つけたので共有します

Gatsbyは自作のテンプレートリポジトリを作ってそこから新しいプロジェクトを生成できるのに、Nextは毎回使うフレームワークもカラッポにインストールしていかなきゃいけないよなぁと思っていました

しかし `create-next-app` には `--example` というオプションがあって、その名の通り本来は [公式のExample](https://github.com/vercel/next.js/tree/canary/examples) を使うためのものなのですが引数がリポジトリのURLだったので、もしかしたらと思い自分のリポジトリを指定してみたところ、しれっとコピーされました

ということなので、知らなかった方は秘伝のタレでテンプレートリポジトリを作っておきましょう
これだけで環境構築が1時間くらい短縮されそうです