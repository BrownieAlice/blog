---
title: "Arch LinuxのCode-OSSの拡張機能の検索先を変更する"
date: 2020-10-13T23:20:40+09:00
categories:
  - "TechNote"
thumbnail: "code.png"
---

Arch Linux の公式リポジトリにある VSCode の OSS 版の[Code-OSS](https://www.archlinux.jp/packages/community/x86_64/code/)の拡張機能の検索先が変更されていました.
従来は[Visual Studio Marketplace](https://marketplace.visualstudio.com/)でしたが, 現在は[Open VSX Registry](https://open-vsx.org/)に登録された拡張機能のみを検索するようです.  
Open VSX は VSCode の拡張機能を登録する OSS なレジストリなようですが, 登録されている拡張機能はそこまで多くなく[PlatformIO](https://platformio.org/)などは登録されていないため実用上少し課題があります.
`/usr/lib/code/product.json` の以下の箇所を

```json
"extensionsGallery": {
    "serviceUrl": "https://open-vsx.org/vscode/gallery",
    "itemUrl": "https://open-vsx.org/vscode/item"
}
```

次のように変えれば従来の Marketplace を利用することはできます.

```json
"extensionsGallery": {
    "serviceUrl": "https://marketplace.visualstudio.com/_apis/public/gallery",
    "cacheUrl": "https://vscode.blob.core.windows.net/gallery/index",
    "itemUrl": "https://marketplace.visualstudio.com/items"
}
```

なお, Visual Studio Marketplace には[利用規約](https://cdn.vsassets.io/v/M146_20190123.39/_content/Microsoft-Visual-Studio-Marketplace-Terms-of-Use.pdf)があるため, これに**同意する必要**があります.  
[Can I put the extensionsGallery on my vscode fork? · Issue #31168 · microsoft/vscode · GitHub](https://github.com/microsoft/vscode/issues/31168)や[Eclipse Open VSX Registry | projects.eclipse.org](https://projects.eclipse.org/proposals/eclipse-open-vsx-registry)で述べられているように, VSCode ではない Code-OSS が Marketplace を利用していいかどうかには議論があります.
