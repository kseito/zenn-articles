---
title: Androidアプリ開発におけるキャッシュについて整理
emoji: "🤖"
type: "tech"
topics: ["Android"]
published: true
published_at: 2021-07-20 18:00
---

Androidアプリのビルド時間を短縮する上で、キャッシュの仕組みを知るのはとても大事だと思い、Androidアプリ開発で使用する下記３つのキャッシュについて調査してみました。
- Incremental Build
- Gradle Build Cache
- Android Studioのシステムキャッシュ

## Incremental Build
### 何者
アプリをビルドすると`app/build/`配下に生成されるキャッシュたちのことです。
Gradleのタスク単位でキャッシュされており、`Task :app:preBuild UP-TO-DATE`のようにタスクの右端に`UP-TO-DATE`という記述がある場合はキャッシュが使用されたことになります。

### どこにキャッシュされるか
`app/build/`直下に生成されます。
より正確にはモジュールのルートディレクトリの直下に生成されます。

### キャッシュのクリア方法
Android StudioでClean Projectを選択するかCLIで`./gradlew clean`を実行することでクリアされます。

## Gradle Build Cache
### 何者
Gradleタスクのアウトプットをプロジェクトの外側に持つ仕組みで、プロジェクト間でキャッシュの共有ができるようです。
AGPのメジャーアップデート時などはこのキャッシュが悪さをすることが稀にあります。

### どこにキャッシュされるか
`~/.gradle/caches`にキャッシュされます。

### キャッシュのクリア方法
キャッシュ置き場をごっそり削除します。
GUIで消すかCLIの場合は`rm -rf ~/.gradle/caches/`でも消せます。

## Android Studioのシステムキャッシュ
### 何者
Android Studio自体が持つキャッシュのことです。
Project Structureの情報などが対象みたいです。

### どこにキャッシュされるか
[公式サイト](https://www.jetbrains.com/help/idea/invalidate-caches.html)を見る限り明記はされていません。が、おそらくAndroid Studioアプリ内のキャッシュだろうと推察します。
Android Studioのアップデート時にキャッシュ削除しますか？的なダイアログが出てきて削除している対象がここで言っているキャッシュの可能性がありますね。

### キャッシュのクリア方法
Android Studio上で`Invalidate Caches / Restart`を選択します。
困ったときはだいたいこれで解決しますね。

## 参考サイト
- https://proandroiddev.com/caching-in-the-android-build-process-a52641a66b31
