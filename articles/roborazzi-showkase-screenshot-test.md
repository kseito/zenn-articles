---
title: "Roborazzi + Robolectric + Showkaseを用いたスクリーンショットテスト"
emoji: "📸"
type: "tech"
topics: ["Android", "ScreenshotTesting", "Roborazzi", "Showkase", "JetpackCompose"]
published: true
---

## はじめに
この記事では、Roborazzi、Robolectric、Showkaseを組み合わせ、既存のComposeプレビューをそのまま活用したスクリーンショットテストの導入方法を紹介します。
開発が進む中で発生しがちなUIの意図しない変更（リグレッション）を、CI上で自動検知するVRT（Visual Regression Test）の仕組みを、比較的シンプルに構築する方法を紹介します。

## 前提知識
各ライブラリの説明を簡単に記載します。

### Roborazzi
Androidアプリのスクリーンショットテストを行うためのライブラリです。  
Robolectricと組み合わせることで、実機やエミュレーターを使用せずにJVMのみでスクリーンショットの撮影が可能です。
Jetpack Composeに対応しており、UIの変更を視覚的に検知するVRT（Visual Regression Test）を簡単に実現できます。

### Robolectric
AndroidアプリのコードをJVM上で実行するためのテスティングフレームワークです。  
実機やエミュレーターを起動することなく、高速にAndroidのAPIを使ったテストが実行できます。  

### Showkase
ShowkaseはJetpack Composeの`@Preview`が付いたComposable関数を自動収集するライブラリです。  
全てのプレビューに対する一括テストを可能にします。

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

#### パラメタライズドテストの実装

すべてのShowkaseコンポーネントを自動的にテストするパラメタライズドテストを作成します。

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
- 収集したComposable関数をパラメータとしてパラメタライズドテストを実行する
- パラメタライズドテストでRoborazziのcaptureRoboImageメソッドを実行することでスクリーンショットを生成

一連の処理が実行されることによってPreviewアノテーションが付いている全てのComposable関数のスクリーンショットを実現しています。

## 結果・動作確認
テストを実行すると、`screenshots/`ディレクトリに各コンポーネントの下記のようなスクリーンショットが生成されます。  

![スクリーンショットテストの実行結果](/images/roborazzi-showkase-screenshot-test/SettingScreenPreview_Default_Group_compare.png)

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

## CI/CDでのVRT実現

実際のCI/CD環境でVRTを動作させるには、上記のコマンドを組み合わせた運用が必要です。

### 基本的なワークフロー

1. **ベースライン作成**: mainブランチで`recordRoborazziDebug`を実行し、スクリーンショットを保存[^1]
2. **変更検知**: Pull Requestで`compareRoborazziDebug`を実行し、差分を検出
3. **結果確認**: `verifyRoborazziDebug`で差分がある場合はCIを失敗させ、レビュー時に差分画像を確認

この仕組みにより、UIに意図しない変更が加わった際に自動的にCIで検知され、安全性を保ちながら開発を進めることができます。

## まとめ
このような仕組みを整備することで、下記のような恩恵を受けることができます。

### テスト対象の自動化
新しいコンポーネントに`@Preview`を追加するだけで自動的にテスト対象となるため、テストの追加漏れがなくなり、メンテナンスコストが削減されます。

### レビュー品質の向上
CI/CDに統合することで、Pull RequestごとにUIの変更点を画像で確認できます。  
これにより、コードレビューだけでは見つけにくい視覚的な変化を逃さず、レビューの質と効率が上がります。

### 安全なコード生成の支援
生成AIを使ってコードを書く際に、このテストが意図しないUIの変更を検知するガードレールとして機能します。

特に最後の恩恵は、これから生成AIがコードを書く機会は益々増えていくため、開発速度を上げる仕組みとして重要性が高まると思っています。

## 参考資料
- https://github.com/DeNA/android-modern-architecture-test-handson
- https://github.com/takahirom/roborazzi
- https://github.com/airbnb/Showkase
- https://github.com/kseito/RewardedTodo/pull/464

[^1]: スクリーンショットの保存場所には様々な選択肢があります。Roborazzi公式リポジトリでは[Companion Branch](https://github.com/takahirom/roborazzi-compare-on-github-comment-sample?tab=readme-ov-file#about-the-companion-branch-approach)という手法が紹介されています。
