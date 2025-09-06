---
title: "Roborazzi + Robolectric + Showkaseを用いたスクリーンショットテスト"
emoji: "🤖"
type: "tech"
topics: ["Android", "ScreenshotTesting", "Roborazzi", "Showkase", "JetpackCompose"]
published: false
---

## はじめに
Roborazzi、Robolectric、Showkaseを組み合わせるとすでに存在するComposeのプレビューに対してスクリーンショットテストを行うことができます。  
それをCIで実行すれば比較的シンプルな仕組みでVRT（Visual Regglession Test）を整備できます。

## 前提知識
各ライブラリの説明を簡単に記載します。

### Roborazzi
Androidアプリのスクリーンショットテストを行うためのライブラリです。  
Robolectricと組み合わせることで、実機やエミュレーターを使用せずにJVMのみでUIテストとスクリーンショットの撮影が可能です。
Composeに対応しており、UIの変更を視覚的に検知するVRT（Visual Regression Testing）を簡単に実現できます。

### Robolectric
AndroidアプリのコードをJVM上で実行するためのテスティングフレームワークです。  
実機やエミュレーターを起動することなく、高速にAndroidのAPIを使ったテストが実行できます。  

### Showkase
ShowkaseはJetpack Composeの`@Preview`が付いたComposable関数を自動収集するライブラリです。  
デザインシステムのコンポーネントカタログの作成や、全てのプレビューに対する一括テストを可能にします。

## 導入手順

### 1. 準備

まず、必要な依存関係を追加します。

**libs.versions.toml**
```toml
[versions]
roborazzi = "1.48.0"
showkase = "1.0.5"
robolectric = "4.16"

[libraries]
roborazzi-compose = { group = "io.github.takahirom.roborazzi", name = "roborazzi-compose", version.ref = "roborazzi" }
roborazzi-rule = { group = "io.github.takahirom.roborazzi", name = "roborazzi-junit-rule", version.ref = "roborazzi" }
robolectric = { group = "org.robolectric", name = "robolectric", version.ref = "robolectric" }
showkase = { group = "com.airbnb.android", name = "showkase", version.ref = "showkase" }
showkase-annotation = { group = "com.airbnb.android", name = "showkase-annotation", version.ref = "showkase" }
showkase-processor = { group = "com.airbnb.android", name = "showkase-processor", version.ref = "showkase" }
```

**build.gradle**
```kotlin
dependencies {
    testImplementation(libs.roborazzi.compose)
    testImplementation(libs.roborazzi.rule)

    testImplementation(libs.robolectric)
    
    debugImplementation(libs.showkase)
    implementation(libs.showkase.annotation)
    ksp(libs.showkase.processor)
}
```

### 2. 実装手順

#### Showkaseモジュールの定義

アプリケーションモジュールにShowkaseRootを定義します。

**ShowkaseModule.kt**
```kotlin
@ShowkaseRoot
class ShowkaseModule : ShowkaseRootModule
```

#### パラメータライズドテストの実装

すべてのShowkaseコンポーネントを自動的にテストするパラメータライズドテストを作成します。

**ShowkaseParameterizedTest**
```kotlin
@OptIn(ExperimentalMaterialApi::class)
@GraphicsMode(GraphicsMode.Mode.NATIVE)
@RunWith(ParameterizedRobolectricTestRunner::class)
@Config(sdk = [35])
class ShowkaseParameterizedTest(private val testCase: TestCase) {

    @get:Rule
    val composeRule = createComposeRule()

    @Test
    fun captureShowkaseComponent() {
        val component = testCase.showkaseBrowserComponent

        composeRule.setContent {
            component.component()
        }

        composeRule
            .onRoot()
            .captureRoboImage(
                filePath = "screenshots/${component.componentName}_${component.group}.png"
                    .replace(" ", "_"),
            )
    }

    companion object {
        class TestCase(val showkaseBrowserComponent: ShowkaseBrowserComponent) {
            override fun toString() = showkaseBrowserComponent.componentKey
        }

        @ParameterizedRobolectricTestRunner.Parameters(name = "[{index}] {0}")
        @JvmStatic
        fun components(): Iterable<Array<*>> = Showkase.getMetadata().componentList.map {
            arrayOf(TestCase(it))
        }
    }
}
```

### 3. ポイント
- ShowkaseでPreviewアノテーションがついているComposable関数を収集
- 収集したComposable関数をパラメータとしてパラメータライズドテストを実行する
- パラメータライズドテストでRoborazziのcaptureRoboImageメソッドを実行することでスクリーンショットを生成

一連の処理が実行されることによってPreviewアノテーションが付いている全てのComposable関数のスクリーンショットを実現しています。

## 結果・動作確認

テストを実行すると、`screenshots/`ディレクトリに各コンポーネントのスクリーンショットが生成されます。  
テストする際は下記のコマンドを使い分けます。

```bash
# 新規スクリーンショットの記録
./gradlew recordRoborazziDebug

# スクリーンショットテストの実行
./gradlew compareRoborazziDebug

# 比較結果の確認
./gradlew verifyRoborazziDebug
```

生成されるファイル例：
- `screenshots/BottomBar_Navigation.png`

## まとめ
このような仕組みを整備することで下記のような恩恵を受けることができます。

- 新しいコンポーネントを追加しても自動的にテスト対象になります
- CI/CDに統合することでPRごとにUIの変更を確認できレビューの品質が向上します
- 生成AIを使ってコードを書く際、ガードレールの1つとして有用

特に最後の恩恵は、これから生成AIがコードを書く機会は益々増えていくため、開発速度を上げる仕組みとして重要性が高まると思っています。

## 参考資料
- https://github.com/DeNA/android-modern-architecture-test-handson
- https://github.com/takahirom/roborazzi
- https://github.com/airbnb/Showkase
- https://github.com/kseito/RewardedTodo/pull/464
