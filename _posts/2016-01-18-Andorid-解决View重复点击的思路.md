---
layout: mypost
title: Andorid-解决View重复点击的思路
categories: [Android]
---
>最近遇到一道面试题，题目是在App开发中，如何防止多次点击支付或者多次点击提交订单？这次的关键是避免View的重复点击的解决办法。

###脑中最开始想到的办法
可以是通过手动记录最后的点击时间,再计算时间间隔来判断是否重复点击
```  
btTest1.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                long nowTime = System.currentTimeMillis();
                if (nowTime - mLastClickTime > TIME_INTERVAL) {
                    // do something
                    mLastClickTime = nowTime;
                } else {
                    Toast.makeText(MainActivity.this, "不要重复点击", Toast.LENGTH_SHORT).show();
                }
            }
        });
```
抑或可以封装一下采用抽象处理
```  
public abstract class IClickListener implements View.OnClickListener {
        @Override
        public final void onClick(View v) {
            if (System.currentTimeMillis() - mLastClickTime >= TIME_INTERVAL) {
                onIClick(v);
                mLastClickTime = System.currentTimeMillis();
            } else {
                onReClick(v);
            }
        }

        protected abstract void onIClick(View v);

        protected abstract void onReClick(View v);
    }
```
使用方法
```
 btTest2.setOnClickListener(new IClickListener() {
            @Override
            protected void onIClick(View v) {
                //do something
            }

            @Override
            protected void onReClick(View v) {
                Toast.makeText(MainActivity.this, "不要重复点击", Toast.LENGTH_SHORT).show();
            }
        }); 
```
问题虽然解决了，但是还是有很多很明显的缺点
1.侵入性过大
2.第三方控件无法处理
3.不可逆
4.代码不美观
###优雅的处理方式
##### 1.使用设计模式
```
 public static class ClickProxy implements View.OnClickListener {

        private View.OnClickListener origin;
        private IreClick mIreClick;

        public ClickProxy(View.OnClickListener origin, IreClick mIreClick) {
            this.origin = origin;
            this.mIreClick = mIreClick;
        }

        public ClickProxy(View.OnClickListener origin) {
            this.origin = origin;
        }

        @Override
        public void onClick(View v) {
            if (System.currentTimeMillis() - mLastClickTime >= TIME_INTERVAL) {
                Log.i(TAG, "ClickProxy in");
                origin.onClick(v);
                mLastClickTime = System.currentTimeMillis();
            } else {
                if (mIreClick != null) mIreClick.onReClick();
            }
        }

        public interface IreClick {
            void onReClick();//重复点击
        }
    }

```
使用
```
 btTest3.setOnClickListener(new ClickProxy(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                //do something
            }
        }, new ClickProxy.IreClick() {
            @Override
            public void onReClick() {
                Toast.makeText(MainActivity.this, "不要重复点击", Toast.LENGTH_SHORT).show();
            }
        }));
```
##### 2.Aop拦截
集成沪江的Aspectjx框架
```
//root gradle
 dependencies {
        classpath 'com.hujiang.aspectjx:gradle-android-plugin-aspectjx:2.0.1'
        }
//app或module gradle
apply plugin: 'android-aspectjx'    //插件
compile 'org.aspectj:aspectjrt:1.8.9'   //jar
```
Aspect代码
```
@Aspect
public class DoubleClickAspect {
    private Long mLastClickTime = 0L;
    private final Long TIME_INTERVAL = 1000L;
    @Around("execution(* android.view.View.OnClickListener.onClick(..))")
    public void clickFilterHook(ProceedingJoinPoint joinPoint) {
        if (System.currentTimeMillis() - mLastClickTime >= TIME_INTERVAL) {
            mLastClickTime = System.currentTimeMillis();
            try {
                joinPoint.proceed();
            } catch (Throwable throwable) {
                throwable.printStackTrace();
            }
        } else {
            Log.e("ClickFilterHook", "不要重复点击");
        }
    }
}
```
注解代码
```
@Target({METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface DoubleCLickMethod {
}
```
使用
```
btTest5.setOnClickListener(new View.OnClickListener() {
        @DoubleCLickMethod
        @Override
        public void onClick(View v) {
            Log.i(TAG, "btn5 click");
        }
    });
```
#####3.Rxjava结合
主要用到Rxjava中的操作符throttleFirst，生成RxView
```
public class RxView {
        public static void setOnClickListeners(final Action1<View> action, @NonNull View... target) {
            for (View view : target) {
                RxView.onClick(view).throttleFirst(1000, TimeUnit.MILLISECONDS).subscribe(new Consumer<View>() {
                    @Override
                    public void accept(@io.reactivex.annotations.NonNull View view) throws Exception {
                        action.onMyClick(view);
                    }
                });
            }
        }


    @CheckResult
    @NonNull
    private static Observable<View> onClick(@NonNull View view) {
        checkNotNull(view, "view == null");
        return Observable.create(new ViewClickOnSubscribe(view));
    }
    private static class ViewClickOnSubscribe implements ObservableOnSubscribe<View> {
        private View view;

        public ViewClickOnSubscribe(View view) {
            this.view = view;
        }

        @Override
        public void subscribe(@io.reactivex.annotations.NonNull final ObservableEmitter<View> e) throws Exception {
            checkUiThread();

            View.OnClickListener listener = new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    if (!e.isDisposed()) {
                        e.onNext(view);
                    }
                }
            };
            view.setOnClickListener(listener);
        }
    }
    public interface Action1<T> {
        void onMyClick(T t);
    }
    public static final class Preconditions {
        public static void checkArgument(boolean assertion, String message) {
            if (!assertion) {
                throw new IllegalArgumentException(message);
            }
        }

        public static <T> T checkNotNull(T value, String message) {
            if (value == null) {
                throw new NullPointerException(message);
            }
            return value;
        }

        public static void checkUiThread() {
            if (Looper.getMainLooper() != Looper.myLooper()) {
                throw new IllegalStateException(
                        "Must be called from the main thread. Was: " + Thread.currentThread());
            }
        }
        private Preconditions() {
            throw new AssertionError("No instances.");
        }
    }

}
```
使用
```
  RxView.setOnClickListeners(this, btTest4);

    @Override
    public void onMyClick(View v) {
        switch (v.getId()) {
            case R.id.btn4:
                Log.i(TAG, "btn4 click");
                break;
            default:
                break;
        }
    }
```
可以看到 结合AOP或Rxjava的 代码已经高度解耦，使用方式也很简单。

PS ：[参考代码](https://github.com/tok-gogogo/DoubleClickTest)