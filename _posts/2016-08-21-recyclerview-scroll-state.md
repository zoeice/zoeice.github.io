---
layout:     post
title:      "监听RecyclerView的滚动状态"
description: 因为要根据滚动状态做一些功能。自定义RecyclerView来实现下
date:        2016-08-21 14:42:00
author:     zoeice
image:      https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-android-n.jpg
optimized_image: https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-android-n.jpg?x-oss-process=image/resize,w_380
catalog:    false
category:   android
tags:
    - Android
    - View
---

因为要根据滚动状态做一些功能。自定义RecyclerView来实现下
代码如下：

```java
/**
 * 带滚动状态监听的自定义RecyclerView
 */
public class StateRecyclerView extends RecyclerView {
    private OnRecyclerScrollListener onRecyclerScrollListener;

    public StateRecyclerView(Context context) {
        this(context, null);
    }

    public StateRecyclerView(Context context, @Nullable AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public StateRecyclerView(Context context, @Nullable AttributeSet attrs, int defStyle) {
        super(context, attrs, defStyle);
        init();
    }

    /**
     * 初始化监听事件
     */
    private void init() {
        addOnScrollListener(onScrollListener);
    }

    /**
     * 滚动监听的对象
     */
    private OnScrollListener onScrollListener = new OnScrollListener() {
        @Override
        public void onScrollStateChanged(RecyclerView recyclerView, int newState) {
            super.onScrollStateChanged(recyclerView, newState);
            if (onRecyclerScrollListener != null) {
                onRecyclerScrollListener.onStateChanged(StateRecyclerView.this, newState);
                if (newState == RecyclerView.SCROLL_STATE_IDLE) {
                    if (!recyclerView.canScrollVertically(1)) {
                        onRecyclerScrollListener.onScrollToBottom();
                    }
                    if (!recyclerView.canScrollVertically(-1)) {
                        onRecyclerScrollListener.onScrollToTop();
                    }
                }
            }
        }

        @Override
        public void onScrolled(RecyclerView recyclerView, int dx, int dy) {
            super.onScrolled(recyclerView, dx, dy);
            if (onRecyclerScrollListener != null) {
                if (dy > 0) {
                    onRecyclerScrollListener.onScrollDown(StateRecyclerView.this, dy);
                } else {
                    onRecyclerScrollListener.onScrollUp(StateRecyclerView.this, dy);
                }
            }
        }
    };

    /**
     * 设置滚动状态的监听
     *
     * @param onRecyclerScrollListener 设置监听 or 设置null清除监听
     */
    public void setOnRecyclerScrollListener(OnRecyclerScrollListener onRecyclerScrollListener) {
        if (onRecyclerScrollListener == null) {
            removeOnScrollListener(onScrollListener);
        } else {
            this.onRecyclerScrollListener = onRecyclerScrollListener;
        }
    }

    /**
     * 当在ScrollStateRecyclerView上的滚动事件发生了，OnRecyclerScrollListener添加到ScrollStateRecyclerView就能接收到消息
     * <p/>
     *
     * @see StateRecyclerView#setOnRecyclerScrollListener(OnRecyclerScrollListener)
     */
    public interface OnRecyclerScrollListener {
        /**
         * 当RecyclerView的滚动状态被改变，会回调
         *
         * @param recyclerView 滚动状态改变的RecyclerView
         * @param newState     更新的滚动状态. {@link #SCROLL_STATE_IDLE}:静止,没有滚动,
         *                     {@link #SCROLL_STATE_DRAGGING}:正在被外部拖拽,一般为用户正在用手指滚动
         *                     or {@link #SCROLL_STATE_SETTLING}:自动滚动开始.
         */
        void onStateChanged(RecyclerView recyclerView, int newState);

        /**
         * 当RecyclerView向上滚动，会回调
         *
         * @param recyclerView 滚动的RecyclerView
         * @param dy           垂直滚动的总值
         */
        void onScrollUp(RecyclerView recyclerView, int dy);

        /**
         * 当RecyclerView向下滚动，会回调
         *
         * @param recyclerView 滚动的RecyclerView
         * @param dy           垂直滚动的总值
         */
        void onScrollDown(RecyclerView recyclerView, int dy);

        /**
         * 当RecyclerView向上滚动到顶部，会回调
         */
        void onScrollToTop();

        /**
         * 当RecyclerView向下滚动到底部，会回调
         */
        void onScrollToBottom();
    }
}
```
