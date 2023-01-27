---
title: SharedPreferencesの保存速度を計測してみた
emoji: "🤖"
type: "tech"
topics: ["Android"]
published: true
published_at: 2020-02-11 09:23
---

SharedPreferencesの処理はUIスレッドに割と書いたりしますが、保存する回数や文字列の長さが増えて問題ないのか確かめてみました。
下記のようなコードで `wordCount`の値を調整する感じです。
５回計測を行い平均値を出してみます。

```kotlin
val wordCount = 1

(0 until 10_000).forEach {
    val prefs = getSharedPreferences("hoge", Context.MODE_PRIVATE)
    prefs.edit().putString("save_value", "a".repeat(wordCount)).apply()
}
```
## 実行環境
- Androidエミュレータ（Pixel2 API 28）
- RAM 1536MB

## 結果
### wordCount=1
(1907 + 2065 + 1907 + 1833 + 1917) / 5 = 1925.8ms

### wordCount=100
(1818 + 1742 + 1806 + 1779 + 1797) / 5 = 1788.4ms

### wordCount=10000
(3542 + 3694 + 3708 + 4433 + 3915) / 5 = 3858.4ms

### おまけ(ループ回数=1_00_000, wordCount=100)
(13859 + 12701 + 12809 + 12754 + 13489) / 5 = 13122.4ms

## 感想
保存する回数や文字列の長さにより保存にかかる時間は増えました。
しかし文字数は１万文字を１回保存するのに0.4msぐらいなのでUIスレッドで実行しても問題なさそうです。
WEB APIから受け取ったJsonをString配列としてキー名を分けて`SharedPreferences`に保存するとかそれくらいやばいことをすれば話は別ですが・・・
結論、`SharedPreferences`の保存処理はUIスレッドを考慮しなくていいくらい早い。
