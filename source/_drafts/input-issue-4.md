title: Input - Issue 4
draft: true
tags:
- kotlin

- android
categories:
- input

### Problems

* 手动隐藏 Android 的键盘
  * [Java 方法](https://stackoverflow.com/questions/43061216/dismiss-keyboard-on-button-click-that-close-fragment/43061451)
  * [Kotlin 方法，通过扩展 AppCombatActivity，更推荐这个方法](https://code.luasoftware.com/tutorials/android/android-hide-keyboard-on-activity-start/)

* 用 intent 启动一个不能返回的 Activity
  
  * finish()
  
* 给 Button 增加圆角，根据状态改变颜色

  * selector

  ```xml
  <selector xmlns:android="http://schemas.android.com/apk/res/android">
      <item android:state_enabled="true">
          <shape android:shape="rectangle">
              <solid android:color="@color/primaryColor" />
              <corners android:radius="8dp" />
          </shape>
      </item>
      <item android:state_enabled="false">
          <shape android:shape="rectangle">
              <solid android:color="#d2d2d2" />
              <corners android:radius="8dp" />
          </shape>
      </item>
  </selector>
  ```

* net::ERR_CLEARTEXT_NOT_PERMITTED

  * https://blog.csdn.net/qq_33721320/article/details/84400825
  * https://stackoverflow.com/questions/45940861/android-8-cleartext-http-traffic-not-permitted
  
* Android WebView 不能弹出alert的对话框

  * https://blog.csdn.net/weixin_40763897/article/details/88951765
  * https://blog.csdn.net/niuzaiwenjie/article/details/80775339
  
* RecyclerView 在 Scroll 中不滚动

  * https://stackoverflow.com/questions/27083091/recyclerview-inside-scrollview-is-not-working

* 模拟器无法访问 127.0.0.1

  * 10.0.2.2
  
* button set backgroundTint

  * https://stackoverflow.com/questions/29801031/how-to-add-button-tint-programmatically/37324427
  
* Fragment back button not work

  * ```kotlin
        override fun onSupportNavigateUp(): Boolean {
            onBackPressed()
            return true
        }	
    ```

  * 

* class.java和javaClass区别

  * https://blog.csdn.net/u012947056/article/details/78925701
