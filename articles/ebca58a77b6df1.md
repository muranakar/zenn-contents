---
title: "SQLiteの基本的な使用方法"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [Flutter, Dart, SQlite]
published: true
published_at: 2025-01-30 07:00 # 未来の日時を指定する
---

## 使用するライブラリ

https://pub.dev/packages/sqflite

## データベースの基本操作

SQLite では、データは「テーブル」という形で管理されます。テーブルは以下のような特徴があります：

- 表のような形式でデータを保存
- 列（カラム）に名前とデータ型を設定
- 行（レコード）としてデータを保存

例えば、果物を管理するテーブルは以下のようになります：

| id  | name（名前） | type（種類） | price（価格） |
| --- | ------------ | ------------ | ------------- |
| 1   | りんご       | 果物         | 100           |
| 2   | みかん       | 果物         | 80            |

## 基本的なデータベース操作

### 1. テーブルの作成

テーブルを作成する時は、各列の名前とデータ型を指定します。

```dart
// テーブルを作成する例
// 以下の列を持つテーブルを作成：
// - id: 自動で増える数値のID（主キー）
// - name: 商品名を入れる列
// - type: 種類を入れる列
// - price: 価格を入れる列
await db.execute('''
  CREATE TABLE products (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,  // NULLを許可しない
    type TEXT,
    price INTEGER DEFAULT 0  // デフォルト値を設定
  )
''');
```

### 2. データの追加

データを追加する方法は 2 つあります：

#### insert メソッドを使う方法（推奨）

```dart
// データを追加する例
// Mapの形式でデータを指定します
int recordId = await db.insert('products', {
  'name': 'りんご',
  'type': '果物',
  'price': 100
});
```

#### SQL 文を直接書く方法

```dart
// SQLを直接書いてデータを追加
await db.rawInsert(
  'INSERT INTO products(name, type, price) VALUES (?, ?, ?)',
  ['りんご', '果物', 100]
);
```

### 3. データの検索

データを検索する方法も 2 つあります：

#### query メソッドを使う方法（推奨）

```dart
// 全てのデータを取得
var allProducts = await db.query('products');

// 特定の列だけを取得
var nameAndPrice = await db.query(
  'products',
  columns: ['name', 'price']
);

// 条件を指定して取得（100円以上の商品）
var expensiveProducts = await db.query(
  'products',
  where: 'price >= ?',
  whereArgs: [100],
  orderBy: 'price DESC'  // 価格の高い順に並べ替え
);
```

#### SQL 文を直接書く方法

```dart
// SQLを直接書いて検索
var products = await db.rawQuery(
  'SELECT * FROM products WHERE price >= ?',
  [100]
);
```

### 4. データの更新

```dart
// 特定の商品の価格を更新
var count = await db.update(
  'products',
  {'price': 150},  // 新しい値
  where: 'name = ?',  // 更新する条件
  whereArgs: ['りんご']
);

print('$count件のデータを更新しました');  // 更新された件数が返ります
```

### 5. データの削除

```dart
// 条件を指定して削除
var count = await db.delete(
  'products',
  where: 'price < ?',
  whereArgs: [50]  // 50円未満の商品を削除
);

// テーブルの全データを削除
await db.delete('products');
```

## トランザクション処理

複数の処理をまとめて行う時に使います。以下のような特徴があります：

- 全ての処理が成功するか、全て失敗するか（途中半端な状態にならない）
- エラーが発生すると自動的に処理が巻き戻される
- データの整合性を保つのに重要

```dart
// トランザクションの例
await db.transaction((txn) async {
  try {
    // 新商品を追加
    await txn.insert('products', {
      'name': 'パイナップル',
      'type': '果物',
      'price': 200
    });

    // 在庫切れ商品を削除
    await txn.delete(
      'products',
      where: 'price = ?',
      whereArgs: [0]
    );

  } catch (e) {
    print('エラーが発生しました: $e');
    // ここでエラーが発生すると、
    // 上記で行った追加と削除が取り消されます
    rethrow;
  }
});
```

## 安全な SQL の書き方

SQL インジェクション攻撃を防ぐために、以下のルールを守りましょう：

::: details 　 SQL インジェクションの一例
「taro」に続いて「’ or ‘1’=‘1」が注入（Injection）されたとします。
生成された SQL 文が下記となります。

```
SELECT * FROM user WHERE id = ‘taro’ or ‘1’=‘1’
```

上記の SQL 文にはデータを抽出する条件が 2 つあります。

- 1 つ目は ID が「taro」と合致した場合です。この場合は「taro」の情報が出力されます。ここまでは前述した基本の動きと同じです。

- 2 つ目は‘1’=‘1’が成立した場合です。ただし、‘1’=‘1’は当然なことなのですべての場合においても成立してしまいます。そのため、この部分によってデータベースに格納されているすべてのデータが出力されます。

すなわち情報の漏えいです。

参照：https://www.shadan-kun.com/waf_websecurity/sql_injection/

:::

### 良い例

```dart
// 値は必ず「?」を使ってプレースホルダーで設定
var products = await db.rawQuery(
  'SELECT * FROM products WHERE price <= ? AND type = ?',
  [100, '果物']
);
```

### 悪い例

```dart
// 文字列結合で値を直接埋め込むのは危険！
var price = '100';
var type = '果物';
// 絶対にこうしないでください！
var products = await db.rawQuery(
  'SELECT * FROM products WHERE price <= $price AND type = "$type"'
);
```

## データベース管理用の便利な機能

- テーブルの存在確認
- 名前の一覧で確認

上記を行いたい場合に重要です。

### テーブルの存在確認

```dart
// テーブルが存在するか確認する関数
Future<bool> tableExists(DatabaseExecutor db, String table) async {
  var count = firstIntValue(await db.query(
    'sqlite_master',
    columns: ['COUNT(*)'],
    where: 'type = ? AND name = ?',
    whereArgs: ['table', table]
  ));
  return count > 0;
}

// 使用例
if (await tableExists(db, 'products')) {
  print('productsテーブルは存在します');
} else {
  print('productsテーブルは存在しません');
}
```

### テーブル一覧の取得

```dart
// データベース内の全テーブル名を取得する関数
Future<List<String>> getTableNames(DatabaseExecutor db) async {
  var tableNames = (await db.query(
    'sqlite_master',
    where: 'type = ?',
    whereArgs: ['table']
  ))
    .map((row) => row['name'] as String)
    .toList(growable: false)
    ..sort();
  return tableNames;
}

// 使用例
var tables = await getTableNames(db);
print('データベース内のテーブル: $tables');
```

## まとめ

SQLite の基本的な使い方について説明しました：

- テーブルの作成：データの構造を定義
- データの追加：新しいレコードを追加
- データの検索：条件に合うデータを取得
- データの更新：既存のデータを変更
- データの削除：不要なデータを削除
- トランザクション：複数の処理を安全に実行

重要なポイント：

1. データ型を適切に選択する
2. SQL インジェクション対策を忘れない
3. エラー処理を適切に行う
4. 大量データの場合はトランザクションを使用する

これらの基本を押さえることで、安全で効率的なデータベース操作が可能になります。

## 参照

https://github.com/tekartik/sqflite/blob/master/sqflite/doc/sql.md
