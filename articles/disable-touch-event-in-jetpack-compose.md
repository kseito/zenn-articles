---
title: Composableのイベント伝達を止める
emoji: "🚀"
type: "tech"
topics: ["Android", "Jetpack Compose"]
published: true
published_at: 2022-02-10 20:00
---

Jetpack Composeを使っていると、タップ・ドラッグなどのタッチイベントを他のComposableに伝えず、指定したComposableで消費したいケースがあります。
調査した所、消費したいイベントによっていくつか選択肢がありそうでした。
自分で試した下記３ケースの実装例を紹介します。
- タップ
- （特定方向の）ドラッグ
- タッチイベント全般

### タップ
タップだけ消費できれば良い場合は、clickable Modifierに空のイベントを定義しつつindicationをnullにしてRipple Effectが表示されないようにします。

```kotlin
.clickable(
    interactionSource = remember { MutableInteractionSource() },
    indication = null,
    onClick = {},
)
```

### （特定方向の）ドラッグ
特定方向のスワイプを無効にしたいケースではdraggable Modifierが使えます。
ただ**特定方向**と書いている通り、draggableはorientationを指定する必要がある都合上、指定した方向のスワイプしか無効にできません。

```kotlin
.draggable(
    interactionSource = remember { MutableInteractionSource() },
    state = remember { DraggableState {} },
    orientation = Orientation.Horizontal,
)
```

### タッチイベント全般
スワイプの方向に関係なくタッチイベントを無効にしたい場合はPointerInputScopeの拡張関数であるdetectDragGesturesが使えます。
下記の処理ではドラッグイベントを全て消費することによって他のビューにイベントを与えないようにしています。

```kotlin
.pointerInput(Unit) {
    detectDragGestures { change, _ -> change.consumeAllChanges() }
}
```

### おまけ（条件付きタッチイベントの無効）
常にタップやタッチイベントを無効にするというケースは稀で、特定の条件の時のみ無効にしたいというケースがより一般的かと思うのでそのケースも載せておきます。

```kotlin
Box(
    modifier = if (isLoading) {
        Modifier.disableTouchEvent()
    } else {
        Modifier
    }
)
```
