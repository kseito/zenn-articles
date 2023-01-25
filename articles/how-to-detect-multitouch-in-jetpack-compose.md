---
title: Jetpack Composeでマルチタップを検知する
emoji: "🚀"
type: "tech"
topics: ["Android", "Jetpack Compose"]
published: true
published_at: 2022-01-15 17:00
---

Jetpack Composeでタッチイベントを制御する際にマルチタップを検知する方法がわからなかったので調査しました。

## 解決策
`PointerEvent`に`changes`というプロパティがあります。
`changes`は`List`型になっておりタッチしている指の数だけ`PointerInputChange`が追加されるようになっているため、
下記のように、このプロパティのサイズを確認すればマルチタップしているかどうかが判定できます。
```kotlin
if (event.changes.size == 2) {
    // 指２本でタップしている場合に行いたい処理
}
```

## 補足
公式ドキュメントからはこの使い方が正しいかどうかはわかりませんでした。
が、Jetpack Composeの`AwaitPointerEventScope.awaitFirstDown()`を見ると最初のDownイベントを検知するために`event.changes[0]`を確認しているのでそこまで的外れではないのかなぁと思っています。
