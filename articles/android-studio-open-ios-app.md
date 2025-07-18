---
title: "Android StudioでiOSアプリを開いて実行する方法"
emoji: "🤖"
type: "tech"
topics: ["Android","AndroidStudio"]
published: true
---

本記事では、Android Studioを使ってiOSアプリを開き実行するまでの手順を解説します。
Android Studio Narwhal以降、Swiftコードの認識や「Xcode Application」実行構成がサポートされiOSアプリの開発ができるようになりました。
普段Android開発をメインに行っている方が、iOSアプリの挙動確認などをしたい場合に便利な機能だと思います。

## 手順

1.  **iOSプロジェクトの準備**
    既存のiOSプロジェクト（`.xcodeproj` または `.xcworkspace` ファイルが含まれるもの）を用意します。

2.  **Android Studioでプロジェクトを開く**
    Android Studioのメニューから `ファイル` > `開く` を選択し、プロジェクトのルートディレクトリではなく、**`.xcodeproj` または `.xcworkspace` ファイルを直接指定して**開きます。

3.  **実行構成の確認**
    プロジェクトが開かれ、画面上部の実行構成（Run Configuration）にターゲットアプリが「Xcode Application」として表示されていれば準備完了です。ここからビルドや実行が行えます。

### プロジェクトの読み込みに成功した時
![Android StudioでiOSアプリを開く](/images/android-studio-ios-configuration.webp)

## まとめ

この機能は主にKotlin Multiplatform向けに提供されていますが、ネイティブiOSアプリ開発でも利用できます。
普段Androidアプリを開発しているエンジニアが、iOSアプリの挙動確認や簡単な修正を行いたい場合にXcodeを別途開く手間が省け使い慣れたAndroid Studio内で作業を完結できるのでとても便利です。

## 参考

*   [Recommended IDEs for Kotlin Multiplatform development | JetBrains](https://www.jetbrains.com/help/kotlin-multiplatform-dev/recommended-ides.html#android-studio)
