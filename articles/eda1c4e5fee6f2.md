---
title: "============【Flutter】SQLiteでのユニットテストの実装方法"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

## はじめに

Flutter アプリケーションで Sqlite データベースのテストを行う方法について、詳しく説明していきます。Flutter でデータベースのテストを行う際には、いくつかの制限事項や注意点がありますが、適切な方法を使うことで効果的なテストが可能です。

## テスト実装の基本的な考え方

Sqflite でのテストには主に 2 つのアプローチがあります：

1. Flutter Driver を使用したテスト
2. sqflite_common_ffi を使用したテスト

今回は、より一般的でシンプルな`sqflite_common_ffi`を使用したアプローチを詳しく見ていきます。

## sqflite_common_ffi を使用したテスト実装

### 基本的なテストの実装例

```dart
// テストに必要なパッケージのインポート
import 'package:flutter_test/flutter_test.dart';
import 'package:sqflite/sqflite.dart';
import 'package:sqflite_common_ffi/sqflite_ffi.dart';

// テスト初期化用の関数
void sqfliteTestInit() {
  // FFI実装の初期化
  sqfliteFfiInit();
  // グローバルファクトリの設定
  databaseFactory = databaseFactoryFfi;
}

void main() {
  // テストの前に初期化を実行
  sqfliteTestInit();

  test('データベースの基本的な操作のテスト', () async {
    // インメモリデータベースを開く
    var db = await openDatabase(inMemoryDatabasePath);

    // テーブルの作成
    await db.execute('''
      CREATE TABLE Product (
        id INTEGER PRIMARY KEY,
        title TEXT
      )
    ''');

    // データの挿入テスト
    await db.insert('Product', {'title': 'テスト商品1'});
    await db.insert('Product', {'title': 'テスト商品2'});

    // データの取得と検証
    var result = await db.query('Product');
    expect(result, [
      {'id': 1, 'title': 'テスト商品1'},
      {'id': 2, 'title': 'テスト商品2'}
    ]);

    // データベースを閉じる
    await db.close();
  });
}
```

### Widget テストでの実装例

Widget テストでは、isolate を使用しない特別な設定が必要です。以下が実装例です：

```dart
// Widgetテスト用の実装
import 'package:flutter_test/flutter_test.dart';
import 'package:sqflite/sqflite.dart';
import 'package:sqflite_common_ffi/sqflite_ffi.dart';

void main() {
  // FFI実装の初期化
  sqfliteFfiInit();

  // isolateを使用しないファクトリの設定
  databaseFactory = databaseFactoryFfiNoIsolate;

  testWidgets('データベース操作を含むWidgetのテスト', (WidgetTester tester) async {
    // データベースのセットアップ
    var db = await openDatabase(
      inMemoryDatabasePath,
      version: 1,
      onCreate: (db, version) async {
        await db.execute(
          'CREATE TABLE Test (id INTEGER PRIMARY KEY, value TEXT)'
        );
      }
    );

    // テストデータの挿入
    await db.insert('Test', {'value': 'テストデータ'});

    // データの検証
    var result = await db.query('Test');
    expect(result, [
      {'id': 1, 'value': 'テストデータ'}
    ]);

    await db.close();
  });
}
```

## テスト実装時の注意点

1. メモリ管理

   - テスト終了時には必ずデータベースを`close()`する
   - インメモリデータベースを使用してテストの独立性を確保

2. 非同期処理

   - すべてのデータベース操作は非同期
   - `async`/`await`を適切に使用する

3. テストの分離
   - 各テストケースは独立して実行できるようにする
   - テストデータは各テストケース内で作成・削除する

## 参照

https://github.com/tekartik/sqflite/blob/master/sqflite/doc/testing.md
