---
layout:     post
title:      "监听ScrollView的滚动状态"
description: 因为要根据ScrolView滚动状态做一些功能。自定义ScrollView来实现下
date:        2016-08-22 14:42:00
author:     zoeice
image:      https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-android-n.jpg
optimized_image: https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-android-n.jpg?x-oss-process=image/resize,w_380
catalog:    false
category:   android
tags:
    - Android
    - View
---

因为要根据ScrolView滚动状态做一些功能。自定义ScrollView来实现下
代码如下：

```java
public class StateScrollView extends ScrollView {
    private OnStateScrollListener onStateScrollListener = null;
    private boolean isScrolling = false;

    public StateScrollView(Context context) {
        this(context, null);
    }

    public StateScrollView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public StateScrollView(Context context, @Nullable AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
        init();
    }

    private void init(){
        setOnTouchListener(new View.OnTouchListener() {
            private int lastY = 0;
            private int touchEventId = -9983761;
            Handler handler = new Handler() {
                @Override
                public void handleMessage(Message msg) {
                    super.handleMessage(msg);
                    View scroller = (View) msg.obj;
                    if (msg.what == touchEventId) {
                        if(onStateScrollListener != null) {
                            if (lastY == scroller.getScrollY()) {
                                //改变为停止滚动状态
                                if (isScrolling) {
                                    isScrolling = false;
                                    onStateScrollListener.onStateChanged(StateScrollView.this, false);
                                }
                            } else {
                                handler.sendMessageDelayed(handler.obtainMessage(touchEventId, scroller), 5);
                                lastY = scroller.getScrollY();
                            }
                        }
                    }
                }
            };

            public boolean onTouch(View v, MotionEvent event) {
                if (event.getAction() == MotionEvent.ACTION_UP) {
                    handler.sendMessageDelayed(handler.obtainMessage(touchEventId, v), 5);
                } else if(event.getAction() == MotionEvent.ACTION_MOVE) {
                    int curScrollY = v.getScrollY();
                    //滚动事件
                    if (isScrolling) {
                        if(curScrollY - lastY > 0){
                            onStateScrollListener.onScrollDown(StateScrollView.this, curScrollY);
                        } else{
                            onStateScrollListener.onScrollUp(StateScrollView.this, curScrollY);
                        }
                    }
                }
                return false;
            }
        });
    }

    public void setOnStateScrollListener(OnStateScrollListener onStateScrollListener) {
        this.onStateScrollListener = onStateScrollListener;
    }

    @Override
    protected void onOverScrolled(int scrollX, int scrollY, boolean clampedX, boolean clampedY) {
        super.onOverScrolled(scrollX, scrollY, clampedX, clampedY);
        if(onStateScrollListener != null) {
            if (clampedY) {
                if (isScrolling) {
                    isScrolling = false;
                    //滚动到顶部或者底部
                    if (scrollY > 0) {
                        onStateScrollListener.onScrollToBottom();
                    } else {
                        onStateScrollListener.onScrollToTop();
                    }
                }
            } else {
                //改变为滚动状态
                if (!isScrolling) {
                    isScrolling = true;
                    onStateScrollListener.onStateChanged(StateScrollView.this, true);
                }
            }
        }
    }

    public interface OnStateScrollListener {
        /**
         * 当 ScrollView 的滚动状态被改变，会回调
         */
        void onStateChanged(ScrollView scrollView, boolean isScrolling);

        /**
         * 当 ScrollView 向上滚动，会回调
         *
         * @param scrollView 滚动的ScrollView
         * @param dy         垂直滚动的总值
         */
        void onScrollUp(ScrollView scrollView, int dy);

        /**
         * 当 ScrollView 向下滚动，会回调
         *
         * @param scrollView 滚动的ScrollView
         * @param dy         垂直滚动的总值
         */
        void onScrollDown(ScrollView scrollView, int dy);

        /**
         * 当 ScrollView 向上滚动到顶部，会回调
         */
        void onScrollToTop();

        /**
         * 当 ScrollView 向下滚动到底部，会回调
         */
        void onScrollToBottom();
    }
}
```

设置监听：

```java
scrollView.setOnStateScrollListener(new StateScrollView.OnStateScrollListener() {
    @Override
    public void onStateChanged(ScrollView scrollView, boolean isScrolling) {
        if(isScrolling) {
            Log.d("TEST", "isScrolling");
        } else {
            Log.d("TEST", "end scroll");
        }
    }

    @Override
    public void onScrollUp(ScrollView scrollView, int dy) {
        Log.d("TEST", "onScrollUp");
    }

    @Override
    public void onScrollDown(ScrollView scrollView, int dy) {
        Log.d("TEST", "onScrollDown");
    }

    @Override
    public void onScrollToTop() {
        Log.d("TEST", "onScrollToTop");
    }

    @Override
    public void onScrollToBottom() {
        Log.d("TEST", "onScrollToBottom");
    }
});
```
