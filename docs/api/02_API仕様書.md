# API一覧（社員管理アプリ）

## 1. 認証系API

| No | エンドポイント | メソッド | 認証 | 権限 | 概要 |
|----|--------------|----------|------|------|------|
| 1 | /api/auth/register | POST | 不要 | 全員 | ユーザーを新規登録 |
| 2 | /api/auth/login | POST | 不要 | 全員 | ログインしJWTを取得 |

---

## 2. 管理者専用API（admin）

| No | エンドポイント | メソッド | 認証 | 権限 | 概要 |
|----|--------------|----------|------|------|------|
| 3 | /api/employees | GET | 必要 | admin | 社員一覧を取得 |
| 5 | /api/employees/{employeeNumber} | POST | 必要 | admin | 社員情報を新規登録or更新 |
| 6 | /api/employees/{employeeNumber} | DELETE | 必要 | admin | 社員情報を削除 |
| 7 | /api/employees/import | POST | 必要 | admin | CSVで一括登録・更新 |

---

## 3. 社員専用API（employee）

| No | エンドポイント | メソッド | 認証 | 権限 | 概要 |
|----|--------------|----------|------|------|------|
| 10 | /api/employees | GET | 必要 | employee | 社員一覧を取得（氏名・メールアドレスのみ）
| 11 | /api/employees/me | GET | 必要 | employee | 自身の情報を取得 |
| 12 | /api/employees/me | PUT | 必要 | employee | 氏名・メールアドレスのみ更新 |

---

## 補足

### ■ 命名ルール
- エンドポイントは名詞で統一する（例：`/employees`）
- 操作はHTTPメソッドで表現する
  - GET：取得
  - POST：作成
  - PUT：更新
  - DELETE：削除

### ■ 認証
- JWT認証を前提とする
- headerに付与する
- 認証が必要なAPIは `Authorization: Bearer {token}` を使用する

### ■ 権限
- admin：全操作可能( 閲覧、登録、削除、更新、CSVによる一括登録 )
- employee : 自身の情報を登録 + 社員情報の閲覧( 名前、メールアドレスのみ )

## 4. レスポンス形式

### 成功
```json
{
  "status": "SUCCESS",
  "data": {},
  "message": "success message"
}
```

### エラー
```json
{
  "status": "ERROR",
  "data": null,
  "message": "error message"
}
```

### ステータスコード
| コード | 意味 |
|-------|------|
| 200 | 成功(送信・更新・削除) |
| 201 | 成功(作成) |
| 400 | 入力エラー |
| 401 | 認証エラー |
| 404 | データなし |
| 409 | データ重複 |
| 500 | サーバーエラー |

---

# API仕様

### ◼︎POST /api/auth/register (ユーザー登録)

#### リクエスト
```json
{
  "email": "example@example.com",
  "password": "password123",
  "message": "アカウントの作成が完了しました。"
}
```

#### レスポンス
```json
{
  "status": "SUCCESS",
  "data": {
      token :"jwt-token" ,
      role :"EMPLOYEE" or "ADMIN"
  }
  "message": null
}
```

### ◼︎POST /api/auth/login (ログイン)

#### リクエスト
```json
{
  "email": "example@exapmle.com",
  "password": "password123",
}
```

#### レスポンス
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

### ◼︎GET /api/employees (社員一覧を取得)

#### リクエストヘッダー

Authorization: Bearer {token}

#### レスポンス
```json
{
  "status": "SUCCESS",
  "data": [
    {
      "employeeNumber": "11111",
      "name": "山田太郎",
      "email": "example@example.com",
      "department": "営業部",
      "position": "部長",
      "joiningDate": "2000-01-01",
      "status": "ACTIVE"
    }
  ],
  "message": "社員一覧の取得が完了しました。"
}
```
## ◼︎GET /api/employees/me (自身の社員情報を取得)

#### リクエストヘッダー

Authorization: Bearer {token}


#### リクエスト
```json
    {
  "name": "山田太郎",
  "email": "example@example.com"
}
```

#### レスポンス
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
  "message": "自身の情報の取得が完了しました"
}
```



## ◼︎PUT /api/employees/{employeeNumber} (社員情報を更新・登録)

#### リクエストヘッダー

Authorization: Bearer {token}

#### リクエスト
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

#### レスポンス
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

## ◼︎POST /api/employees/me (自身の情報の登録・更新)

#### リクエストヘッダー

Authorization: Bearer {token}


#### リクエスト
```json
    {
  "name": "山田太郎",
  "email": "example@example.com"
}
```

#### レスポンス
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
  "message": "登録または更新が完了しました"
}
```

### ◼︎DELETE /api/auth/{employeeNumber} (社員情報の削除)

#### リクエストヘッダー

Authorization: Bearer {token}

#### レスポンス
```json
{
  "status": "SUCCESS",
  "data": null,
  "message": "社員情報を削除しました"
}
```

## ◼︎POST /api/employees/import (CSVで一括登録・更新)

#### リクエストヘッダー

Authorization: Bearer {token}

#### リクエスト
CSVファイル

#### レスポンス
```json
{
  "status": "SUCCESS",
  "data": "CSV取込完了",
  "message": "読み取りが完了しました"
}
```