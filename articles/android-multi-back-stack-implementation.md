---
title: "Multi Back Stackを実装する"
emoji: "✨"
type: "tech"
topics: [Android]
published: false
---

[Navigationライブラリの2.4.0](https://developer.android.com/jetpack/androidx/releases/navigation#2.4.0-alpha01)からMulti Back Stackという複数のnavigatioh graphを戻るアクション含め一括管理してくれる便利な機能が提供されています。
が、[公式ドキュメント](https://developer.android.com/guide/navigation/multi-back-stacks)だけだと実装の流れを理解し難かったので備忘録として書きます。
ざっくりした流れは下記になります。（[公式サンプル](https://github.com/android/architecture-components-samples/tree/master/NavigationAdvancedSample)をベースにしています）
- 各タブで扱うFragment, navigationグラフのxmlを定義する
- ベースとなるActivityを作成しFragmentContainerViewを持たせる
- 各タブで扱うnavigationグラフをFragmentContainerViewのパラメータに設定
- ActivityでNavigationライブラリのメソッドを呼び出す

より具体的な手順を下記に書いていきます。

### 各タブで扱うFragment, navigationグラフのxmlを定義する
公式サンプルでは下記のFragmentとnavigationグラフを準備しています。

- Homeタブ
  - Fragment
    - About.kt
    - Title.kt
  - navigationグラフ
    - home.xml
- Leaderboardタブ
  - Fragment
    - Leaderboard.kt
    - UseProfile.kt
  - navigationグラフ
    - list.xml
- Registerタブ
  - Fragment
    - Register.kt
    - Registered.kt
  - navigationグラフ
    - form.xml

### 各navigationグラフを束ねるnavigationグラフを作成する
公式サンプルではnav_graph.xmlと定義されておりコードは下記のようにシンプルです。

```xml
<navigation
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/nav_graph"
    app:startDestination="@+id/home">

    <include app:graph="@navigation/home"/>
    <include app:graph="@navigation/list"/>
    <include app:graph="@navigation/form"/>

</navigation>
```


### ベースとなるActivityを作成しFragmentContainerViewを持たせパラメータを設定する
大元のActivityで使われるレイアウトファイル（公式サンプルでいうactivity_main.xml）にnav_graph.xmlを紐付けます。


```xml
<androidx.fragment.app.FragmentContainerView
        android:id="@+id/nav_host_container"
        android:name="androidx.navigation.fragment.NavHostFragment"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        app:defaultNavHost="true"
        app:navGraph="@navigation/nav_graph" />
```


### ActivityでNavigationライブラリのメソッドを呼び出す
後はNavHostFragmentからnavControllerを取り出し、Navigationライブラリに準備されている拡張関数を呼び出すだけです。

```kotlin
val navHostFragment = supportFragmentManager.findFragmentById(R.id.nav_host_container) as NavHostFragment
navController = navHostFragment.navController

val bottomNavigationView = findViewById<BottomNavigationView>(R.id.bottom_nav)
bottomNavigationView.setupWithNavController(navController)

appBarConfiguration = AppBarConfiguration(setOf(R.id.titleScreen, R.id.leaderboard,  R.id.register))
setupActionBarWithNavController(navController, appBarConfiguration)
```

### おまけ
Multi Back Stackを使うと上記のような簡単に実装できます。
また、タブを押した時に追加でUIの変更を行いたい場合には`NavController.addOnDestinationChangedListener`を使うとタブ押下イベントを拾うことができます。

### 参考サイト
- https://developer.android.com/guide/navigation/multi-back-stacks
- https://medium.com/androiddevelopers/navigation-multiple-back-stacks-6c67ba41952f
- https://github.com/android/architecture-components-samples/tree/master/NavigationAdvancedSample

