---
layout: post
title: "Flutter Webview app"
date: 2024-07-14 13:51:23 +0900
categories: Flutter
---

Flutter 이용하여 가볍게 웹뷰를 띄워보자. "Must Have 코드팩토리의 플러터 프로그래밍" 도서를 참고했다.

### 코드
**main.dart**
```dart
import 'package:flutter/material.dart';
import 'package:flutter_sampler/screen/home_screen.dart';

void main() {
  runApp(const MaterialApp(
    home: HomeScreen(),
    debugShowCheckedModeBanner: false,
  ));
}
```
- 앱을 실행하면 HomeScreen 위젯 호출
- `debugShowCheckModeBanner` 옵션으로 디버그 모드 표시를 설정할 수 있음

**home_screen.dart**
```dart
import 'package:flutter/material.dart';
import 'package:webview_flutter/webview_flutter.dart';

class HomeScreen extends StatelessWidget {
  String homeUrl = 'https://www.peanuts.com';
  WebViewController webViewController = WebViewController()
    ..loadRequest(Uri.parse('https://www.peanuts.com'))
    ..setJavaScriptMode(JavaScriptMode.unrestricted);

  HomeScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          backgroundColor: Colors.red,
          title: const Text(
            'Snoopy World',
            style: TextStyle(color: Colors.white, fontWeight: FontWeight.w600),
          ),
          centerTitle: true,
          actions: [
            IconButton(
                onPressed: () {
                  webViewController.loadRequest(Uri.parse(homeUrl));
                },
                icon: const Icon(
                  Icons.home,
                  color: Colors.white,
                ))
          ],
        ),
        body: WebViewWidget(
          controller: webViewController,
        ));
  }
}
```
- `Scaffold` 위젯으로 `AppBar` 와 `WebviewWidget` 을 담은 body 를 구현함
- AppBar 에서 action 속성에 홈 아이콘 버튼을 추가함(추가된 버튼은 순서대로 우측 상단에 나열됨)
- 상단에 미리 정의해둔 `webViewController` 를 `WebViewWidget` 의 `controller` 로 등록함

### 결과물
![작업 결과]({{site_url}}/src/imgs/flutter_webview_sample.gif)

### 소스 코드
https://github.com/alohaymee/flutter-sampler/tree/webview-sample