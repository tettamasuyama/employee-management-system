# API一覧（社員管理システム）

## 1. 認証API

| No | エンドポイント | メソッド | 認証 | 権限 | 概要 |
|----|--------------|----------|------|------|------|
| 1 | /api/auth/register | POST | 不要 | 全員 | ユーザーを新規登録 |
| 2 | /api/auth/login | POST | 不要 | 全員 | ログインしJWTを取得 |

---

## 2. 管理者API（ADMIN）

| No | エンドポイント | メソッド | 認証 | 権限 | 概要 |
|----|--------------|----------|------|------|------|
| 3 | /api/employees | GET | 必要 | ADMIN | 社員一覧を取得 |
| 4 | /api/employees/{employeeNumber} | PUT | 必要 | ADMIN | 社員情報を登録または更新 |
| 5 | /api/employees/{employeeNumber} | DELETE | 必要 | ADMIN | 社員情報を削除 |
| 6 | /api/employees/import | POST | 必要 | ADMIN | CSVによる一括登録・更新 |

---

## 3. 社員API（EMPLOYEE）

| No | エンドポイント | メソッド | 認証 | 権限 | 概要 |
|----|--------------|----------|------|------|------|
| 7 | /api/employees | GET | 必要 | EMPLOYEE | 社員一覧を取得（氏名・メールアドレスのみ） |
| 8 | /api/employees/me | GET | 必要 | EMPLOYEE | 自身の社員情報を取得 |
| 9 | /api/employees/me | PUT | 必要 | EMPLOYEE | 自身の氏名・メールアドレスを更新 |

---

# API共通仕様

## 命名ルール

- エンドポイントは名詞で統一する
- 操作はHTTPメソッドで表現する

| メソッド | 内容 |
|----------|------|
| GET | 取得 |
| POST | 作成 |
| PUT | 更新 |
| DELETE | 削除 |

---

## 認証

JWT認証を採用する。

認証が必要なAPIはリクエストヘッダーに

```
Authorization: Bearer {token}
```

を付与する。

---

## 権限

### ADMIN

- 社員一覧取得
- 社員登録
- 社員更新
- 社員削除
- CSV一括登録

### EMPLOYEE

- 社員一覧取得（氏名・メールアドレスのみ）
- 自身の社員情報取得
- 自身の氏名・メールアドレス更新

---

# 共通レスポンス

## 成功

```json
{
  "status": "SUCCESS",
  "data": {},
  "message": "success message"
}
```

## エラー

```json
{
  "status": "ERROR",
  "data": null,
  "message": "error message"
}
```

---

## ステータスコード

| コード | 内容 |
|---------|------|
| 200 | 取得・更新・削除成功 |
| 201 | 作成成功 |
| 400 | 入力エラー |
| 401 | 認証エラー |
| 404 | データが存在しない |
| 409 | データ重複 |
| 500 | サーバーエラー |

---

# API詳細

---

## POST /api/auth/register

### リクエスト

```json
{
  "email": "example@example.com",
  "password": "password123"
}
```

### レスポンス

```json
{
  "status": "SUCCESS",
  "data": {
    "token": "jwt-token",
    "role": "EMPLOYEE"
  },
  "message": "ユーザー登録が完了しました。"
}
```

---

## POST /api/auth/login

### リクエスト

```json
{
  "email": "example@example.com",
  "password": "password123"
}
```

### レスポンス

```json
{
  "status": "SUCCESS",
  "data": {
    "token": "jwt-token",
    "role": "EMPLOYEE"
  },
  "message": "ログインが完了しました。"
}
```

---

## GET /api/employees

### リクエストヘッダー

```
Authorization: Bearer {token}
```

### レスポンス

```json
{
  "status": "SUCCESS",
  "data": [
    {
      "employeeNumber": "11111",
      "name": "山田太郎",
      "email": "example@example.com",
      "department": "SALES",
      "position": "MANAGER",
      "hireDate": "2000-01-01",
      "status": "ACTIVE"
    }
  ],
  "message": "社員一覧の取得が完了しました。"
}
```

---

## GET /api/employees/me

### リクエストヘッダー

```
Authorization: Bearer {token}
```

### レスポンス

```json
{
  "status": "SUCCESS",
  "data": {
    "employeeNumber": "11111",
    "name": "山田太郎",
    "email": "example@example.com",
    "department": "SALES",
    "position": "MANAGER",
    "hireDate": "2000-01-01",
    "status": "ACTIVE"
  },
  "message": "自身の情報の取得が完了しました。"
}
```

---

## PUT /api/employees/{employeeNumber}

### リクエストヘッダー

```
Authorization: Bearer {token}
```

### リクエスト

```json
{
  "name": "山田太郎",
  "email": "example@example.com",
  "department": "SALES",
  "position": "MANAGER",
  "hireDate": "2000-01-01",
  "status": "ACTIVE"
}
```

### レスポンス

```json
{
  "status": "SUCCESS",
  "data": {
    "employeeNumber": "11111",
    "name": "山田太郎",
    "email": "example@example.com",
    "department": "SALES",
    "position": "MANAGER",
    "hireDate": "2000-01-01",
    "status": "ACTIVE"
  },
  "message": "登録または更新が完了しました。"
}
```

---

## PUT /api/employees/me

### リクエストヘッダー

```
Authorization: Bearer {token}
```

### リクエスト

```json
{
  "name": "山田太郎",
  "email": "example@example.com"
}
```

### レスポンス

```json
{
  "status": "SUCCESS",
  "data": {
    "employeeNumber": "11111",
    "name": "山田太郎",
    "email": "example@example.com",
    "department": "SALES",
    "position": "MANAGER",
    "hireDate": "2000-01-01",
    "status": "ACTIVE"
  },
  "message": "更新が完了しました。"
}
```

---

## DELETE /api/employees/{employeeNumber}

### リクエストヘッダー

```
Authorization: Bearer {token}
```

### レスポンス

```json
{
  "status": "SUCCESS",
  "data": null,
  "message": "社員情報を削除しました。"
}
```

---

## POST /api/employees/import

### リクエストヘッダー

```
Authorization: Bearer {token}
```

### リクエスト

CSVファイル

### レスポンス

```json
{
  "status": "SUCCESS",
  "data": "CSV取込完了",
  "message": "CSVの取込みが完了しました。"
}
```
