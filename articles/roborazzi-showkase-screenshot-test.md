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
TBD

### Robolectric
TBD

### Showkase
TBD

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
- 生成AIを使ってコードを書く際、ガードレールの1つとして有用です

特に最後の恩恵は、これから生成AIがコードを書く機会は益々増えていくと思うので、TBD

## 参考資料
- https://github.com/DeNA/android-modern-architecture-test-handson
- https://github.com/takahirom/roborazzi
- https://github.com/airbnb/Showkase
