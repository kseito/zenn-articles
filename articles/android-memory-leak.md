---
title: 2020年のAndroidアプリ開発でメモリリークは考慮する必要があるのか
emoji: "🤖"
type: "tech"
topics: ["Android"]
published: true
published_at: 2020-10-25 21:00
---

同僚とメモリリークの話になり自分の理解不足を痛感したので調査してみました。
業務で開発しているアプリはメモリリークの問題で困っていないですが、事前にメモリリークしないようにコードを記述しておけば未然に防げる問題なので学んでおいて損はなさそう。

## 前提
動作確認した環境は下記。
- Android Studio 4.0.1
- Pixel 3a XL(Android Emulator) 
- Android 10

また、今回はActivityがメモリリークしているか確認します。
試すパターンは下記３つ
- 内部クラスのstatic参照
- BroadcastReceiver解除忘れ
- インナークラスで親をプロパティ保持

## メモリリークを検知する方法
1. アプリ起動
2. 画面を回転する
3. Android SutdioのProfilerから強制的にGCを走らせる
4. Activityのfinalizeメソッドが呼ばれるか確認

## 正常系
まずは空のActivityの時にfinalizeメソッドは呼ばれるのか確認します。

**ソースコード**
```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        Log.d(TAG, "onCreate")
    }

    override fun onDestroy() {
        Log.d(TAG, "onDestroy")
        super.onDestroy()
    }

    protected fun finalize() {
        Log.d(TAG, "finalize")
    }
}
```

上記のようなコードを書きました。
結果は下記のようになります。

**結果**
```
D/MainActivity: onCreate
D/MainActivity: onDestroy
D/MainActivity: onCreate
D/MainActivity: finalize
```

当たり前ですが画面回転を行ってからGCが走ると画面回転前のMainActivityインスタンスは解放されます。


## 内部クラスのstatic参照
非staticな内部クラスを親がstaticプロパティとして保持するパターンです。

**ソースコード**
```kotlin
class MainActivity : AppCompatActivity() {

    companion object {
        private var innerClass: SomeInnerClass? = null
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        Log.d(TAG, "onCreate")

        if (innerClass == null) {
            innerClass = SomeInnerClass()
        }
    }

    inner class SomeInnerClass()
}
```

**結果**
```
D/MainActivity: onCreate
D/MainActivity: onDestroy
D/MainActivity: onCreate
```

強制的にGCした時にfinalizeメソッドが呼ばれていないのでメモリリークを起こしています。
MainActivityが破棄されても、staticプロパティが生きているので参照が残っているためGCで解放されない、という流れでしょうか。
このあたりの挙動はどうすれば確認できるか謎なので割愛。

## BroadcastReceiver解除忘れ
Androidでよくある実装として、各種リスナーやレシーバーをActivityに登録して使うやり方があります。
その時、画面終了時に登録解除を忘れたりするとメモリリークするらしいので確認してみます。

**ソースコード**
```kotlin
class MainActivity : AppCompatActivity() {

    private var TAG = MainActivity::class.simpleName

    private var localBroadcastReceiver: BroadcastReceiver? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        Log.d(TAG, "onCreate")
    }

    override fun onStart() {
        super.onStart()

        registerBroadCastReceiver()
    }

    override fun onStop() {
        super.onStop()

          //あえてレシーバーの登録解除をスキップする
//        if (localBroadcastReceiver != null) {
//            unregisterReceiver(localBroadcastReceiver)
//        }
    }

    private fun registerBroadCastReceiver() {
        localBroadcastReceiver = object : BroadcastReceiver() {
            override fun onReceive(context: Context, intent: Intent) {
            }
        }
        registerReceiver(
            localBroadcastReceiver,
            IntentFilter("android.net.conn.CONNECTIVITY_CHANGE")
        )
    }
}
```

**結果**
```
D/MainActivity: onCreate
D/MainActivity: onDestroy
D/MainActivity: onCreate
```

画面回転後にGCを走らせてもfinalizeメソッドが呼ばれませんでした。
これはBroadcastReceiverがActivityの強参照を保持しているかららしいです。

## インナークラスで親をプロパティ保持

**ソースコード**
```kotlin
class MainActivity : AppCompatActivity() {

    private var TAG = MainActivity::class.simpleName

    private lateinit var innerClass: InnerClass

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        Log.d(TAG, "onCreate")

        innerClass = InnerClass()
        innerClass.activity = this
    }

    override fun onDestroy() {
        Log.d(TAG, "onDestroy")
        super.onDestroy()
    }

    protected fun finalize() {
        Log.d(TAG, "finalize")
    }

    inner class InnerClass {
        lateinit var activity: Activity
    }
}
```

**結果**
```
D/MainActivity: onCreate
D/MainActivity: onDestroy
D/MainActivity: onCreate
D/MainActivity: finalize
```

MainActivityとInnerClass間で循環参照になっているのにGCで問題なく解放されてますね・・・謎
BroadcastReceiverと事情は同じはずなのでメモリリークすると思ってました。
昔はこれでメモリリークしてたようなのですが、今現在は問題ないようです。


## 所感
公式の[GCに関する情報](https://source.android.google.cn/devices/tech/dalvik/gc-debug?hl=ja)を確認する限りAndroid 10でも変更が入ってたりするのでそのあたりの変更により以前に比べてメモリリークしにくくなっているのかなぁと推測します。
また、ハードウェアの進化でヒープ領域が以前に比べて大きくなったことも考慮すると、メモリリークに対してそこまで神経質にならなくても良いのではと思います。
それよりも昨今Androidアプリ開発だとアーキテクチャがどうとかクラス間が疎結合になってるとか可読性とか責務の分離とか、そう言った事柄の方が重視されてる気がします。
メモリやヒープを気にしなくてよくなった結果、コードの中身を考える余裕ができアーキテクチャが重視されるようになったと考えると、コードの中身が整ったら次は何が重視されるのだろうかとふと思いました。
全然まとまってないけど終わり。

## 参考サイト
https://www.geeksforgeeks.org/memory-leaks-in-android/
https://qiita.com/amay077/items/3df253f66724c56faaff
https://tomokey.blogspot.com/2011/05/android.html
