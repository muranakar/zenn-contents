---
title: "【Flutter】スクリーンショットを取る方法"
emoji: "👌"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter, Dart]
published: true
published_at: 2025-03-02 07:00 # 未来の日時を指定する
---

## はじめに

個人開発でスクリーンショットの機能がほしく、サンプルコードを実装しメモとして記します。今回は`screenshot`パッケージを使用して、Widget のスクリーンショットを撮影する方法を解説します。

## 実装例

![](https://storage.googleapis.com/zenn-user-upload/b5860138be70-20250205.gif)

## サンプルコード

https://github.com/muranakar/screenshot_sample

## ライブラリ

https://pub.dev/packages/screenshot

## スクリーンショットの基本実装

まず、スクリーンショットを撮影するための基本的な実装です。`ScreenshotController`を使用して、Widget のキャプチャを管理します。

```dart:main.dart
// コントローラーとキャプチャした画像を保持する変数の準備
ScreenshotController screenshotController = ScreenshotController();
Uint8List? imageFile;

// キャプチャ処理の実装
void _captureScreenshot() async {
  final image = await screenshotController.capture();
  setState(() {
    imageFile = image;
  });
}
```

`ScreenshotController`は、Widget のキャプチャを制御するためのクラスです。`capture()`メソッドを呼び出すことで、指定した Widget の現在の状態を`Uint8List`形式の画像データとして取得できます。

## キャプチャ対象の Widget 実装

次に、実際にキャプチャする Widget の実装方法です。

```dart:main.dart
Screenshot(
  controller: screenshotController,
  child: // 省略
)
```

キャプチャしたい Widget を`Screenshot`ウィジェットでラップすることです。これにより、`ScreenshotController`がその Widget の領域を特定し、キャプチャすることができます。

## 非表示 Widget のキャプチャ

画面に表示されていない Widget もキャプチャできる機能を実装です。これは、カスタマイズされたプレビュー画像を生成する際などに便利です。

```dart:main.dart
void _captureInvisibleWidget() async {
  final image = await screenshotController.captureFromWidget(
    // 省略
  );
  setState(() {
    imageFile = image;
  });
}
```

`captureFromWidget`メソッドは、実際に UI に表示されていない Widget もキャプチャできる機能です。このメソッドは、指定された Widget を一時的にメモリ上でレンダリングし、そのイメージを取得します。これにより、現在の画面状態に依存せずに、画像を生成することができます。

以上です。
