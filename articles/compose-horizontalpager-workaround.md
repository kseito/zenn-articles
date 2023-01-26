---
title: HorizontalPagerで最初に表示するページを指定する
emoji: "🚀"
type: "tech"
topics: ["Android", "Jetpack Compose"]
published: true
published_at: 2021-12-19 12:00:00
---

公私共にJetpack Composeを使ってコードを書く頻度が増えてきました。
宣言的UIという書き方にまだまだ不慣れということもあり、使い方がわからず（ワンチャンライブラリ側の不具合ということもあるかもしれない）ハマることもしばしば…。
今回はそんなハマり所の中で、個人的に多くの時間を費やしてしまったモノを取り上げてみます。

## 環境
Jetpack Compose関連のライブラリのバージョンは下記の通りです。

```
implementation "androidx.compose.material:material:1.0.4"
implementation "androidx.compose.ui:ui:1.0.4"
implementation "androidx.compose.ui:ui-tooling:1.0.4"
implementation "androidx.compose.foundation:foundation:1.0.4"
implementation "com.google.accompanist:accompanist-pager:0.20.0"
```

## 問題
タイトルにもある通り、HorizontalPagerを使って最初に表示ポジションを0ではなくnページ目を表示したいという要求がありました。
それに対してJetpack Composeを使ってコードを書いてみたのですがなかなか動かずハマりました。
問題となるコードは下記になります。

```kotlin
val position = 3// ここで初回表示したいポジションを指定する
val pagerState = rememberPagerState(position)

HorizontalPager(
    state = pagerState,
) {
    // ページ単位で表示するレイアウトを表示
}
LaunchedEffect(pagerState) {
    snapshotFlow { pagerState.currentPage }.collect {
        // ページが変化したときに呼び出す処理
    }
}
```

上記のコードだと、LaunchedEffect内のページが変化したときの処理がHorizontalPager初期化時に呼ばれません。
HorizontalPager自体のポジションは期待したものになっていたのでページが変化したときに特に処理を呼び出さないのであれば上記のコードでも問題なく動きます。
調査した結果、HorizontalPagerの初期化前にポジションを渡してしまうとLaunchedEffectでページ変化の処理を観測できないようです。
よくよく考えると当たり前ですね。

## 解決策
下記のように、HorizontalPagerが呼び出された後にポジションを指定すると、ページ変化の処理を観測しているLaunnedEffectが意図した通り呼ばれるようになります。

```kotlin
val position = 3// ここで初回表示したいポジションを指定する
val pagerState = rememberPagerState()

HorizontalPager(
    state = pagerState,
) {
    // ページ単位で表示するレイアウトを表示
}
LaunchedEffect(pagerState) {
    snapshotFlow { pagerState.currentPage }.collect {
        // ページが変化したときに呼び出す処理
    }
}
LaunchedEffect(Unit) {
    pagerState.scrollToPage(position)
}
```

## 所感
宣言的UIは、先に状態がありそれをUIに流し込むことで表示したいUIを実現する手法だと考えていて、今回のように宣言順序を考慮しないと行けないケースはなかなか気づきづらいです。
これはLaunchedEffectという副作用のある処理を書いていることに起因している気がするので、HorizontalPagerにViewPagerでいうonPageSelected的なコールバックが追加されれば解決しそうです。
まだ0.20.0なのでこれからに期待。

## 参考サイト
https://google.github.io/accompanist/pager/
https://google.github.io/accompanist/pager/#reacting-to-page-changes
