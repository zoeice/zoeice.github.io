---
layout:     post
title:      Flutter的MVVM模式
subtitle:   
description: 
date:        2019-08-01 13:00:00
author:     zoeice
image:      https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-spring.jpg
optimized_image: https://zoeice-blog.oss-cn-shanghai.aliyuncs.com/post-bg-spring.jpg?x-oss-process=image/resize,w_380
catalog:    true
category:   Flutter
tags:
    - Flutter
    - MVVM
---

## 创建项目
在Android Studio里，File -> New -> New Flutter Project -> Flutter Application

创建/lib/core/base/base_view.dart
```dart

import 'package:flutter/material.dart';
import 'package:provider/provider.dart';
import 'package:fluttermvvmdemo/app/app_module.dart';

import 'base_viewmodel.dart';

// ignore: must_be_immutable
abstract class BaseView<T extends BaseViewModel> extends StatefulWidget {
  T viewModel = locator<T>();

  final Widget Function(BuildContext context, T model, Widget child) builder;
  final Function(T) onViewModelReady;

  BaseView({this.builder, this.onViewModelReady});

  @override
  Widget build(BuildContext context) {
    return ChangeNotifierProvider<T>.value(
        value: viewModel,
        child: Consumer<T>(builder: builder));
  }

}

abstract class BaseViewState<T extends BaseViewModel> extends State<BaseView<T>> {

  @override
  void initState() {
    if (widget.onViewModelReady != null) {
      widget.onViewModelReady(widget.viewModel);
    }
    super.initState();
  }
}
```

创建/lib/core/base/base_viewmodel.dart
```dart
import 'package:flutter/widgets.dart';

import 'enum/view_state.dart';

class BaseViewModel extends ChangeNotifier {
  ViewState _state = ViewState.Idle;

  ViewState get state => _state;

  void setState(ViewState viewState) {
    _state = viewState;
    notifyListeners();
  }
}
```

创建/lib/core/base/base.dart
```dart
import 'dart:async';

import 'package:flutter/material.dart';
import 'package:rxdart/rxdart.dart';

/// normal click event
abstract class Presenter {
  /// 处理点击事件
  ///
  /// 可根据 [action] 进行区分 ,[action] 应是不可变的量
  void onClick(String action);
}

/// ListView Item Click
abstract class ItemPresenter<T> {
  /// 处理列表点击事件
  ///
  /// 可根据 [action] 进行区分 ,[action] 应是不可变的量
  void onItemClick(String action, T item);
}

/// BaseProvide
class BaseProvide with ChangeNotifier {

  CompositeSubscription compositeSubscription = CompositeSubscription();


  /// add [StreamSubscription] to [compositeSubscription]
  ///
  /// 在 [dispose]的时候能进行取消
  addSubscription(StreamSubscription subscription){
    compositeSubscription.add(subscription);
  }

  @override
  void dispose() {
    super.dispose();
    compositeSubscription.dispose();
  }
}
```

来写一个页面
创建/lib/app/modules/dashboard/dashbpard_page.dart
```dart
import 'package:flutter/material.dart';
import 'package:fluttermvvmdemo/core/base/base_view.dart';

import '../../../router.dart';
import 'viewmodel/dashboard_view_model.dart';

/// 仪表盘
// ignore: must_be_immutable
class DashboardPage extends BaseView<DashboardViewModel> {
  @override
  _DashboardViewState createState() => _DashboardViewState();
}

class _DashboardViewState extends State<DashboardPage> {

  @override
  void initState() {
    super.initState();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: Colors.white,
      body: SafeArea(
          child: Column(
        children: <Widget>[
          Text('dashboard'),
          RaisedButton(
            child: const Text('setting'),
            onPressed: () {
              Router.push(context, Router.settingPage, null);
            },
          ),
        ],
      )),
    );
  }
}

```

创建对应的ViewModel: /lib/app/modules/dashboard/viewmodel/dashboard_view_model.dart
```dart
import 'package:fluttermvvmdemo/core/base/base_viewmodel.dart';

import '../../../api_service.dart';

class DashboardViewModel extends BaseViewModel {
  final APIRepo _repo;

  /// 结果
  String _response = "";

  String get response => _response;

  set response(String response) {
    _response = response;
    notifyListeners();
  }

  /// 是否处于loading状态
  bool _loading = false;

  bool get loading => _loading;

  set loading(bool loading) {
    _loading = loading;
    notifyListeners();
  }

  /// 构造函数
  DashboardViewModel(this._repo);

}
```

