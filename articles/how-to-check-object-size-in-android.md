---
title: Android Studioでアプリ起動中に生成されるオブジェクトのメモリサイズを調べる
emoji: "🤖"
type: "tech"
topics: ["Android", "Android Studio"]
published: true
published_at: 2021-07-08 16:00
---

DaggerのModuleとComponentの紐付けについて考えていると、DIする全てのクラスをSingletonComponentに紐付けておけば良いのではと考えたりすることがあります。
この案が現実的なのか、インスタンス化されたクラスがどの程度のサイズになるのか調べればわかるのではと思い調べてみました。

## Memory Profilerでアプリのメモリ使用量を調べる
実行中のAndroidアプリのメモリ使用量を調査するときはAndroid StudioのMemory Profilerを使います。
詳しいことは[公式ドキュメント](https://developer.android.com/studio/profile/memory-profiler?hl=ja)に記載されているので端折ります。

こちらを使ってDaggerのSingletonComponentで生成されるクラスのメモリ使用量を調べると下記のようになりました。

![Mempry Profilerからダンプしたメモリ情報１](./how-to-check-object-size-in-android/ide-image1.png)

なぜかTodoRepositoryのインスタンスが４つもできてますね…
インスタンスによって使用しているメモリサイズが異なるのが気になります。
試しにTodoRepositoryを提供しているメソッドに@Singletonを付けると下記のようになりました。

![Mempry Profilerからダンプしたメモリ情報２](./how-to-check-object-size-in-android/ide-image2.png)

インスタンスが１つになりました。
シングルトンにできていない状態でも5KB程度かつ今回調査したアプリでDaggerのSingletonComponentで生成しているクラスは100未満、仮に100個のクラスをインスタンス化したとしてもメモリ使用量は500KBと考えると全く問題なさそうです。

## 所感
Memory Profilerで調査する限りは問題なさそうなのですが、もっと確信を持って問題ないと言えるようになるには、
オブジェクトをインスタンス化してメモリに展開する時にJVM上でどのような処理が行われているのか、コンパイルがどのようにjavaファイルをclassファイルに変換しているのか等、より低レイヤーな部分も理解する必要がありそうです。
学習は続く…
