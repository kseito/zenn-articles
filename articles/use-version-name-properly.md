---
title: バージョンコード取得方法の使い分け
emoji: "🤖"
type: "tech"
topics: ["Android"]
published: true
published_at: 2021-08-18 18:00
---

Android開発においてバージョンコードを取得する方法は私の知る限り２つあります。
１つはBuildConfigの定数を参照する方法、もう１つはPackageManagerから取得する方法です。
この２つをどう使い分ければ良いのか考えてみました。

## バージョンコードの取得方法
改めてコードベースでバージョンコードの取得方法を確認してみます。それぞれ下記のような実装になります。

### BuildConfigの定数を参照する
```kotlin
val versionCode: Int = BuildConfig.VERSION_CODE
```

### PackageManagerから取り出す
```kotlin
try {
    val context = requireContext()
    val info: PackageInfo = context.packageManager.getPackageInfo(context.packageName, 0)
    val version: Int = info.versionCode
} catch (e: PackageManager.NameNotFoundException) {
    e.printStackTrace()
}
```
※API Level 28からは`versionCode`ではなく`getLongVersionCode()`を参照した方が良いみたいです。

## 使い分け
結論から書くと、マルチモジュールを採用しているかどうかを基準にすると良いです。
なぜならマルチモジュール環境下の場合はappモジュールのBuildConfigを他のモジュールから参照できないため、BuildConfigを使った取得方法は使えません。
（正確には、使えないことはないですが他モジュールがappモジュールを参照する必要が出てくるため、モジュール構造的に無理が出てきたり循環参照が発生しやすくなったりします。）
対してPackageManagerから取得する方法はAndroid SDKが参照できる箇所なら問題なく使えるためマルチモジュール環境でも問題ありません。
バージョンネームの使い分けも同様の考え方で行けると思います。

## 参考サイト
https://stackoverflow.com/questions/4616095/how-can-you-get-the-build-version-number-of-your-android-application
