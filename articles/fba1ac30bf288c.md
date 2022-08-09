---
title: "VSCode で C/C++ の code formatting がうまくいかなかった"
emoji: "🖖"
type: "tech"
topics:
  - "cpp"
  - "vscode"
  - "c"
  - "環境構築"
  - "codeformatter"
published: true
published_at: "2021-04-26 15:41"
---

初歩的すぎて誰も教えてくれなかったこと

# TL; DR
⌥⇧F でデフォルトのフォーマッターを選ぶ
(Win では Alt+Shift+F っぽいけど未確認)

# 詳細
`.vscode/settings.json` に
```json
"editor.formatOnSave": true,
```
も追加したし、
[Clang-Format](https://marketplace.visualstudio.com/items?itemName=xaver.clang-format) も入れて `brew install clang-format` もしたし、
`.clang-format` に設定も書いたのにフォーマットされない…
OUTPUTにも何も出てない……

というときは、VSCodeでフォーマッターが衝突している可能性がある

[C/C++](https://marketplace.visualstudio.com/items?itemName=ms-vscode.cpptools) にはデフォルトでフォーマッターが入っているので、その上に Clang-Format を入れると、VSCodeはどっちを使えばいいのか困ってしまうらしい
*(じゃあなんかエラー吐いてくれよって思うけど)*

⌥⇧F で強制的にフォーマットさせると選択ダイアログが出るので好きな方を選ぶ
:::message
⌥⇧F が動かないときは、コマンドパレット (⌘⇧P / Ctrl+Shift+P) で `Format Document` を探してみる
:::

するとデフォルトのフォーマッターが `.vscode/settings.json` に設定される
```json
"[c]": {
  "editor.defaultFormatter": "ms-vscode.cpptools"
},
```

Clang-Format いらなかった