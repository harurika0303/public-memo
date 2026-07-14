# SQL Server テーブルのロック状況確認手順

## 目次
1. [概要](#概要)
2. [方法1: 動的管理ビュー(DMV)を使う](#方法1-動的管理ビューdmvを使う)
3. [方法2: sp_lockを使う(レガシー)](#方法2-sp_lockを使うレガシー)
4. [方法3: アクティビティモニターを使う(SSMS GUI)](#方法3-アクティビティモニターを使うssms-gui)
5. [方法4: ブロッキングの原因を特定する](#方法4-ブロッキングの原因を特定する)
6. [方法5: 拡張イベントでロック待ちを監視する](#方法5-拡張イベントでロック待ちを監視する)
7. [ロックモードの読み方](#ロックモードの読み方)
8. [よくある対処法](#よくある対処法)

---

## 概要

SQL Serverでテーブルのロック状況を把握するには、主に以下のアプローチがあります。

| 方法 | 用途 |
|---|---|
| DMV (`sys.dm_tran_locks`) | 現在のロック情報をリアルタイムで確認 |
| `sp_lock` | 簡易的なロック確認(非推奨・互換性維持用) |
| アクティビティモニター | GUIで視覚的に確認 |
| ブロッキングクエリ | ブロックの原因セッションを特定 |
| 拡張イベント | 長期的な監視・ロック待ちの記録 |

---

## 方法1: 動的管理ビュー(DMV)を使う

最も推奨される方法です。`sys.dm_tran_locks` を使うことで、現在保持されている、または要求されているすべてのロック情報を取得できます。

### 基本クエリ

```sql
SELECT 
    tl.resource_type,
    tl.resource_database_id,
    DB_NAME(tl.resource_database_id) AS database_name,
    OBJECT_NAME(p.object_id, tl.resource_database_id) AS table_name,
    tl.resource_associated_entity_id,
    tl.request_mode,
    tl.request_type,
    tl.request_status,
    tl.request_session_id,
    es.login_name,
    es.host_name,
    es.program_name,
    es.status AS session_status
FROM sys.dm_tran_locks tl
LEFT JOIN sys.partitions p 
    ON tl.resource_associated_entity_id = p.hobt_id
LEFT JOIN sys.dm_exec_sessions es 
    ON tl.request_session_id = es.session_id
WHERE tl.resource_type = 'OBJECT'
ORDER BY tl.request_session_id;
```

### 特定のテーブルに絞り込む

```sql
SELECT 
    tl.request_session_id,
    tl.resource_type,
    tl.request_mode,
    tl.request_status,
    es.login_name,
    es.host_name
FROM sys.dm_tran_locks tl
LEFT JOIN sys.dm_exec_sessions es 
    ON tl.request_session_id = es.session_id
WHERE tl.resource_associated_entity_id = OBJECT_ID('dbo.対象テーブル名')
   OR tl.resource_associated_entity_id IN (
        SELECT hobt_id FROM sys.partitions 
        WHERE object_id = OBJECT_ID('dbo.対象テーブル名')
   );
```

### 実行中のSQL文も併せて確認する

```sql
SELECT 
    tl.request_session_id,
    OBJECT_NAME(p.object_id) AS table_name,
    tl.request_mode,
    tl.request_status,
    er.status AS query_status,
    er.wait_type,
    er.wait_time,
    st.text AS sql_text
FROM sys.dm_tran_locks tl
LEFT JOIN sys.partitions p 
    ON tl.resource_associated_entity_id = p.hobt_id
LEFT JOIN sys.dm_exec_requests er 
    ON tl.request_session_id = er.session_id
OUTER APPLY sys.dm_exec_sql_text(er.sql_handle) st
WHERE tl.resource_type = 'OBJECT'
ORDER BY tl.request_session_id;
```

---

## 方法2: sp_lockを使う(レガシー)

古いバージョンとの互換性のために残されているシステムストアドプロシージャです。SQL Server 2012以降では非推奨(将来のバージョンで削除予定)ですが、簡易確認には使えます。

```sql
EXEC sp_lock;
```

特定のセッションIDに絞る場合:

```sql
EXEC sp_lock @spid1 = 55;
```

> ⚠️ **注意**: `sp_lock` は将来のバージョンで削除される可能性があるため、新規開発では DMV (`sys.dm_tran_locks`) の使用を推奨します。

---

## 方法3: アクティビティモニターを使う(SSMS GUI)

GUIで手軽に確認したい場合はこちらが便利です。

### 手順
1. SQL Server Management Studio (SSMS) でサーバーに接続する
2. オブジェクトエクスプローラーでサーバー名を右クリック
3. **[アクティビティ モニター]** を選択(またはキーボードショートカット `Ctrl + Alt + A`)
4. **[プロセス]** セクションを展開し、`ブロックの原因` 列でブロッキングの有無を確認
5. **[リソースの待機の概要]** セクションでロック待ちの統計を確認

### 確認できる主な情報
- 現在実行中のプロセス一覧
- 各プロセスがブロックされているかどうか
- 待機時間・待機の種類
- CPU/IO使用率との相関

---

## 方法4: ブロッキングの原因を特定する

ロックが原因で処理が停止している(ブロッキングが発生している)場合の調査手順です。

### ブロッキングチェーンを確認する

```sql
SELECT 
    r.session_id AS ブロックされているSPID,
    r.blocking_session_id AS ブロックしているSPID,
    r.wait_type,
    r.wait_time AS 待機時間ミリ秒,
    r.status,
    st.text AS 実行中のSQL,
    OBJECT_NAME(tl.resource_associated_entity_id) AS 対象テーブル
FROM sys.dm_exec_requests r
CROSS APPLY sys.dm_exec_sql_text(r.sql_handle) st
LEFT JOIN sys.dm_tran_locks tl 
    ON r.session_id = tl.request_session_id AND tl.resource_type = 'OBJECT'
WHERE r.blocking_session_id <> 0;
```

### ブロッキングの元(先頭)セッションを特定する

```sql
SELECT session_id, blocking_session_id, wait_type, wait_time, status
FROM sys.dm_exec_requests
WHERE blocking_session_id = 0
  AND session_id IN (
      SELECT blocking_session_id FROM sys.dm_exec_requests WHERE blocking_session_id <> 0
  );
```

### 原因セッションの実行内容を確認

```sql
DECLARE @spid INT = 55; -- 調査したいSPIDに置き換え

DBCC INPUTBUFFER(@spid);

SELECT text 
FROM sys.dm_exec_sql_text(
    (SELECT sql_handle FROM sys.dm_exec_requests WHERE session_id = @spid)
);
```

---

## 方法5: 拡張イベントでロック待ちを監視する

一時的なスナップショットではなく、継続的にロック待ちを記録・分析したい場合は拡張イベント(Extended Events)を使用します。

```sql
CREATE EVENT SESSION [LockMonitoring] ON SERVER
ADD EVENT sqlserver.lock_acquired(
    WHERE ([sqlserver].[database_name] = N'対象データベース名')
),
ADD EVENT sqlserver.blocked_process_report
ADD TARGET package0.event_file(
    SET filename = N'C:\Temp\LockMonitoring.xel'
)
WITH (MAX_MEMORY = 4096 KB, MAX_DISPATCH_LATENCY = 5 SECONDS);

-- セッション開始
ALTER EVENT SESSION [LockMonitoring] ON SERVER STATE = START;
```

> 💡 **補足**: `blocked_process_report` を有効にするには、事前に `sp_configure 'blocked process threshold'` を設定しておく必要があります。

```sql
EXEC sp_configure 'show advanced options', 1;
RECONFIGURE;
EXEC sp_configure 'blocked process threshold', 5; -- 5秒以上のブロックを検知
RECONFIGURE;
```

---

## ロックモードの読み方

`request_mode` 列に表示される主なロックモード一覧です。

| モード | 意味 |
|---|---|
| S | 共有ロック(読み取り) |
| X | 排他ロック(更新・削除・挿入) |
| IS | 意図共有ロック |
| IX | 意図排他ロック |
| U | 更新ロック |
| Sch-S | スキーマ安定性ロック |
| Sch-M | スキーマ変更ロック |

`request_status` 列の意味:

| ステータス | 意味 |
|---|---|
| GRANT | ロック取得成功 |
| WAIT | ロック待機中(ブロッキング発生の可能性) |
| CONVERT | ロックモードの変換待ち |

---

## よくある対処法

ロックやブロッキングが問題になっている場合の一般的な対処方針です。

- **トランザクションを短くする**: 不要な処理をトランザクション内に含めない
- **適切な分離レベルを選択する**: 必要以上に厳しい分離レベル(SERIALIZABLEなど)を避ける
- **インデックスを見直す**: インデックス不足によるテーブルスキャン・広範囲ロックを防ぐ
- **READ COMMITTED SNAPSHOT の検討**: 読み取りがロック待ちを起こさないようにする
  ```sql
  ALTER DATABASE 対象データベース名 SET READ_COMMITTED_SNAPSHOT ON;
  ```
- **NOLOCKヒントの使用は慎重に**: ダーティリードのリスクがあるため用途を限定する

---

## 参考: 全体を一目で確認できる統合クエリ

```sql
SELECT 
    es.session_id,
    er.blocking_session_id,
    DB_NAME(tl.resource_database_id) AS database_name,
    OBJECT_NAME(p.object_id) AS table_name,
    tl.request_mode,
    tl.request_status,
    er.wait_type,
    er.wait_time,
    es.login_name,
    es.host_name,
    es.program_name,
    st.text AS sql_text
FROM sys.dm_tran_locks tl
LEFT JOIN sys.partitions p 
    ON tl.resource_associated_entity_id = p.hobt_id
LEFT JOIN sys.dm_exec_sessions es 
    ON tl.request_session_id = es.session_id
LEFT JOIN sys.dm_exec_requests er 
    ON tl.request_session_id = er.session_id
OUTER APPLY sys.dm_exec_sql_text(er.sql_handle) st
WHERE tl.resource_type = 'OBJECT'
ORDER BY er.blocking_session_id DESC, tl.request_session_id;
```
