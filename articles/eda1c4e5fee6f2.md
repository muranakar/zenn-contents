---
title: "【Flutter】SQLiteでのユニットテストの実装方法"
emoji: "😎"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter, Dart, SQlite]
published: true
published_at: 2025-02-21 07:00 # 未来の日時を指定する
---

## 使用するライブラリ

https://pub.dev/packages/sqflite

https://pub.dev/packages/sqflite_common_ffi

## サンプルコード

https://github.com/muranakar/sqflite_unit_test

## sqflite のテスト実装の基本的な考え方

Sqflite でのテストには主に 2 つのアプローチがあります：

1. `Flutter Driver` を使用したテスト
2. `sqflite_common_ffi` を使用したテスト

一般的でシンプルな`sqflite_common_ffi`を使用した方法を説明します。

## sqflite_common_ffi を使用したテスト実装

### ライブラリの追加

`pubspec.yaml` に ` sqflite`　`sqflite_common_ffi `を追加します。

```
  sqflite: ^2.0.0+3
  sqflite_common_ffi: ^2.3.4+4
```

### テスト実装

```dart
void main() {
  // FFI実装の初期化
  sqfliteFfiInit();

  // isolateを使用しないファクトリの設定
  databaseFactory = databaseFactoryFfiNoIsolate;

  test('SQLite database operations test', () async {
    // データベースのセットアップ
    var db = await openDatabase(inMemoryDatabasePath, version: 1,
        onCreate: (db, version) async {
      await db
          .execute('CREATE TABLE Test (id INTEGER PRIMARY KEY, value TEXT)');
    });

    // テストデータの挿入
    await db.insert('Test', {'value': 'テストデータ'});

    // データの検証
    var result = await db.query('Test');
    expect(result, [
      {'id': 1, 'value': 'テストデータ'}
    ]);

    // データベースを閉じることを忘れずに
    await db.close();
  });
}
```

### Widget テストでの実装例

Widget テストでは、isolate を使用しない特別な設定が必要です。

```dart
void main() {
  // FFI実装の初期化
  sqfliteFfiInit();

  // !!!isolateを使用しないファクトリの設定！！！
  databaseFactory = databaseFactoryFfiNoIsolate;

  testWidgets('SQLite database operations in Widget test', (WidgetTester tester) async {
    // データベースのセットアップ
    var db = await openDatabase(inMemoryDatabasePath, version: 1,
        onCreate: (db, version) async {
      await db
          .execute('CREATE TABLE Test (id INTEGER PRIMARY KEY, value TEXT)');
    });

    // テストデータの挿入
    await db.insert('Test', {'value': 'テストデータ'});

    // データの検証
    var result = await db.query('Test');
    expect(result, [
      {'id': 1, 'value': 'テストデータ'}
    ]);

    // データベースを閉じることを忘れずに
    await db.close();
  });
}

```

### 複数の Unittest を実行時の warning 出力

一つの main 関数の中で複数の Unit テストを行う際に、以下のアラートが出力されました。

```
You are changing sqflite default factory.
Be aware of the potential side effects. Any library using sqflite
will have this factory as the default for all operations.
```

https://github.com/tekartik/sqflite/issues/1009

私の場合は ffi 実装の初期化前に、`databaseFactoryOrNull = null;`を実行することで warning が消えました。以下に実装例を記します。

https://github.com/muranakar/sqflite_unit_test/blob/main/test/sqflite_test.dart#L7-L13

## テスト実装時の注意点

- テスト終了時には必ずデータベースを`close()`する
- インメモリデータベースを使用してテストの独立性を確保
- すべてのデータベース操作は非同期

## 参照

https://github.com/tekartik/sqflite/blob/master/sqflite/doc/testing.md
