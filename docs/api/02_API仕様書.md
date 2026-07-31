# API仕様書（社員管理システム）

## 1. API一覧

### 1.1 認証API

| No | エンドポイント              | メソッド | 認証 | 権限 | 概要                     |
| -- | -------------------- | ---- | -- | -- | ---------------------- |
| 1  | `/api/auth/register` | POST | 不要 | 全員 | ログインユーザー情報および社員情報を新規登録 |
| 2  | `/api/auth/login`    | POST | 不要 | 全員 | ログイン認証を行いJWTを取得        |

---

### 1.2 社員管理API

| No | エンドポイント                 | メソッド   | 認証 | 権限             | 概要                          |
| -- | ----------------------- | ------ | -- | -------------- | --------------------------- |
| 3  | `/api/employees`        | GET    | 必要 | ADMIN・EMPLOYEE | 社員一覧を取得                     |
| 4  | `/api/employees`        | POST   | 必要 | ADMIN          | 社員情報を新規登録                   |
| 5  | `/api/employees/{id}`   | PUT    | 必要 | ADMIN          | 社員情報を更新                     |
| 6  | `/api/employees/{id}`   | DELETE | 必要 | ADMIN          | 社員情報および対応するユーザー情報を削除        |
| 7  | `/api/employees/import` | POST   | 必要 | ADMIN          | CSVによる社員情報の一括登録・更新          |
| 8  | `/api/employees/me`     | GET    | 必要 | EMPLOYEE       | ログインユーザー自身の社員情報を取得          |
| 9  | `/api/employees/me`     | PUT    | 必要 | EMPLOYEE       | ログインユーザー自身の氏名・社員用メールアドレスを更新 |

---

### 1.3 部署マスターAPI

| No | エンドポイント                 | メソッド | 認証 | 権限             | 概要      |
| -- | ----------------------- | ---- | -- | -------------- | ------- |
| 10 | `/api/departments`      | GET  | 必要 | ADMIN・EMPLOYEE | 部署一覧を取得 |
| 11 | `/api/departments`      | POST | 必要 | ADMIN          | 部署を新規登録 |
| 12 | `/api/departments/{id}` | PUT  | 必要 | ADMIN          | 部署名を更新  |

---

### 1.4 役職マスターAPI

| No | エンドポイント               | メソッド | 認証 | 権限             | 概要      |
| -- | --------------------- | ---- | -- | -------------- | ------- |
| 13 | `/api/positions`      | GET  | 必要 | ADMIN・EMPLOYEE | 役職一覧を取得 |
| 14 | `/api/positions`      | POST | 必要 | ADMIN          | 役職を新規登録 |
| 15 | `/api/positions/{id}` | PUT  | 必要 | ADMIN          | 役職名を更新  |

---

# 2. API共通仕様

## 2.1 命名ルール

* エンドポイントは名詞で統一する。
* 処理内容はHTTPメソッドで表現する。
* JSONの項目名はキャメルケースで記述する。
* データベースのカラム名はスネークケースで記述する。

| HTTPメソッド | 処理内容    |
| -------- | ------- |
| GET      | データ取得   |
| POST     | データ新規登録 |
| PUT      | データ更新   |
| DELETE   | データ削除   |

---

## 2.2 認証

JWT認証を使用する。

認証が必要なAPIには、以下のリクエストヘッダーを付与する。

```http
Authorization: Bearer {token}
```

JWTのユーザー識別情報には、Usersテーブルの`id`を使用する。

---

## 2.3 権限

Usersテーブルの`role`によって権限を判定する。

| 値 | 権限       |
| - | -------- |
| 0 | EMPLOYEE |
| 1 | ADMIN    |

### ADMINが利用できる機能

* 社員一覧取得
* 社員情報登録
* 社員情報更新
* 社員情報削除
* CSV一括登録・更新
* 部署一覧取得
* 部署登録
* 部署更新
* 役職一覧取得
* 役職登録
* 役職更新

### EMPLOYEEが利用できる機能

* 社員一覧取得
* 自身の社員情報取得
* 自身の氏名更新
* 自身の社員用メールアドレス更新
* 部署一覧取得
* 役職一覧取得

---

## 2.4 メールアドレスの管理

UsersテーブルとEmployeesテーブルでは、メールアドレスを別々に管理する。

| 項目              | 登録先               | 用途                    |
| --------------- | ----------------- | --------------------- |
| `loginEmail`    | `users.email`     | ログイン認証に使用するメールアドレス    |
| `employeeEmail` | `employees.email` | 社員情報として表示・管理するメールアドレス |

* `Users.email`と`Employees.email`には異なるメールアドレスを登録できる。
* 社員が自身の`Employees.email`を更新しても、`Users.email`は更新しない。
* ログイン認証では`Users.email`を検索キーとして使用する。
* `Users.email`と`Employees.email`には、それぞれUNIQUE制約を設定する。

---

## 2.5 共通レスポンス

### 成功レスポンス

```json
{
  "status": "SUCCESS",
  "data": {},
  "message": "処理が完了しました。"
}
```

### エラーレスポンス

```json
{
  "status": "ERROR",
  "data": null,
  "message": "エラー内容"
}
```

---

## 2.6 HTTPステータスコード

| コード | 内容                        |
| --- | ------------------------- |
| 200 | 取得・更新・削除成功                |
| 201 | 新規登録成功                    |
| 400 | 入力値エラー                    |
| 401 | 認証エラー                     |
| 403 | 権限不足                      |
| 404 | 対象データが存在しない               |
| 409 | メールアドレス、社員番号、部署名、役職名などの重複 |
| 500 | サーバー内部エラー                 |

---

# 3. 認証API

## 3.1 POST /api/auth/register

ログインに使用するUsers情報と、それに対応するEmployees情報を新規登録する。

UsersテーブルとEmployeesテーブルへの登録は、同一トランザクション内で実行する。

いずれかの登録処理に失敗した場合は、両方の登録を取り消す。

### 認証・権限

| 認証 | 権限 |
| -- | -- |
| 不要 | 全員 |

### リクエスト

```json
{
  "loginEmail": "login@example.com",
  "password": "password123",
  "role": 0,
  "employeeNumber": "11111",
  "name": "山田太郎",
  "employeeEmail": "employee@example.com",
  "departmentId": 4,
  "positionId": 1,
  "hireDate": "2026-04-01",
  "status": "ACTIVE"
}
```

### リクエスト項目

| 項目名            | 型       | 必須 | 登録先                         | 説明                   |
| -------------- | ------- | -- | --------------------------- | -------------------- |
| loginEmail     | String  | ○  | `users.email`               | ログインメールアドレス          |
| password       | String  | ○  | `users.password`            | ログインパスワード            |
| role           | Integer | ○  | `users.role`                | 0：EMPLOYEE、1：ADMIN   |
| employeeNumber | String  | ○  | `employees.employee_number` | 社員番号                 |
| name           | String  | ○  | `employees.name`            | 氏名                   |
| employeeEmail  | String  | ○  | `employees.email`           | 社員用メールアドレス           |
| departmentId   | Long    | ○  | `employees.department_id`   | 部署ID                 |
| positionId     | Long    | ○  | `employees.position_id`     | 役職ID                 |
| hireDate       | String  | ○  | `employees.hire_date`       | 入社日。`YYYY-MM-DD`形式   |
| status         | String  | ○  | `employees.status`          | `ACTIVE`または`RETIRED` |

### 登録処理

1. `loginEmail`の重複をUsersテーブルで確認する。
2. `employeeNumber`の重複をEmployeesテーブルで確認する。
3. `employeeEmail`の重複をEmployeesテーブルで確認する。
4. `departmentId`に対応する部署が存在することを確認する。
5. `positionId`に対応する役職が存在することを確認する。
6. パスワードをハッシュ化する。
7. Usersテーブルへログイン情報を登録する。
8. 登録された`Users.id`を取得する。
9. 取得した`Users.id`を`Employees.user_id`に設定する。
10. Employeesテーブルへ社員情報を登録する。
11. JWTを発行する。

### レスポンス

```json
{
  "status": "SUCCESS",
  "data": {
    "token": "jwt-token",
    "role": 0
  },
  "message": "ユーザー登録が完了しました。"
}
```

### ステータスコード

| コード | 条件                       |
| --- | ------------------------ |
| 201 | 登録成功                     |
| 400 | 入力値または値制約が不正             |
| 404 | 指定した部署または役職が存在しない        |
| 409 | ログインメール、社員用メールまたは社員番号が重複 |
| 500 | 登録処理に失敗                  |

---

## 3.2 POST /api/auth/login

Usersテーブルのメールアドレスとパスワードを使用してログイン認証を行う。

### 認証・権限

| 認証 | 権限 |
| -- | -- |
| 不要 | 全員 |

### リクエスト

```json
{
  "loginEmail": "login@example.com",
  "password": "password123"
}
```

### リクエスト項目

| 項目名        | 型      | 必須 | 説明                             |
| ---------- | ------ | -- | ------------------------------ |
| loginEmail | String | ○  | `Users.email`に登録されたログインメールアドレス |
| password   | String | ○  | ログインパスワード                      |

### 処理内容

1. `loginEmail`を検索キーとしてUsersテーブルを検索する。
2. 入力されたパスワードとハッシュ化済みパスワードを照合する。
3. 認証に成功した場合はJWTを発行する。
4. レスポンスとしてJWTと権限を返す。

### レスポンス

```json
{
  "status": "SUCCESS",
  "data": {
    "token": "jwt-token",
    "role": 0
  },
  "message": "ログインが完了しました。"
}
```

### ステータスコード

| コード | 条件                    |
| --- | --------------------- |
| 200 | ログイン成功                |
| 400 | 入力値が不正                |
| 401 | メールアドレスまたはパスワードが一致しない |
| 500 | 認証処理に失敗               |

---

# 4. 社員管理API

## 4.1 GET /api/employees

社員一覧を取得する。

権限によって返却する項目を制御する。

DepartmentsテーブルおよびPositionsテーブルをJOINまたはJPAの関連付けによって参照する。

### リクエストヘッダー

```http
Authorization: Bearer {token}
```

### 管理者向けレスポンス

ADMINには社員情報の全項目を返す。

```json
{
  "status": "SUCCESS",
  "data": [
    {
      "id": 1,
      "userId": 10,
      "employeeNumber": "11111",
      "name": "山田太郎",
      "employeeEmail": "employee@example.com",
      "departmentId": 4,
      "departmentName": "SALES",
      "positionId": 3,
      "positionName": "MANAGER",
      "hireDate": "2026-04-01",
      "status": "ACTIVE"
    }
  ],
  "message": "社員一覧の取得が完了しました。"
}
```

### 社員向けレスポンス

EMPLOYEEには氏名、社員用メールアドレス、部署および役職を返す。

```json
{
  "status": "SUCCESS",
  "data": [
    {
      "name": "山田太郎",
      "employeeEmail": "employee@example.com",
      "departmentName": "SALES",
      "positionName": "MANAGER"
    }
  ],
  "message": "社員一覧の取得が完了しました。"
}
```

### ステータスコード

| コード | 条件            |
| --- | ------------- |
| 200 | 取得成功          |
| 401 | JWTが未指定または無効  |
| 403 | APIを利用する権限がない |
| 500 | 取得処理に失敗       |

---

## 4.2 POST /api/employees

管理者が、既存のUsersデータに対応する社員情報をEmployeesテーブルへ登録する。

このAPIではUsersテーブルのログイン情報は新規登録しない。

Employeesテーブルの`user_id`はNOT NULLであるため、登録済みの`Users.id`を必ず指定する。

### 認証・権限

| 認証 | 権限    |
| -- | ----- |
| 必要 | ADMIN |

### リクエストヘッダー

```http
Authorization: Bearer {token}
```

### リクエスト

```json
{
  "userId": 10,
  "employeeNumber": "11111",
  "name": "山田太郎",
  "employeeEmail": "employee@example.com",
  "departmentId": 4,
  "positionId": 3,
  "hireDate": "2026-04-01",
  "status": "ACTIVE"
}
```

### リクエスト項目

| 項目名            | 型      | 必須 | 説明                    |
| -------------- | ------ | -- | --------------------- |
| userId         | Long   | ○  | 関連付けるUsersテーブルのユーザーID |
| employeeNumber | String | ○  | 社員番号                  |
| name           | String | ○  | 氏名                    |
| employeeEmail  | String | ○  | 社員用メールアドレス            |
| departmentId   | Long   | ○  | 部署ID                  |
| positionId     | Long   | ○  | 役職ID                  |
| hireDate       | String | ○  | 入社日。`YYYY-MM-DD`形式    |
| status         | String | ○  | `ACTIVE`または`RETIRED`  |

### 登録ルール

* `userId`に対応するUsersデータが存在しない場合はエラーとする。
* すでにEmployeesテーブルと関連付けられている`userId`は使用できない。
* `employeeNumber`が重複する場合はエラーとする。
* `employeeEmail`が重複する場合はエラーとする。
* `departmentId`に対応する部署が存在しない場合はエラーとする。
* `positionId`に対応する役職が存在しない場合はエラーとする。

### レスポンス

```json
{
  "status": "SUCCESS",
  "data": {
    "id": 1,
    "userId": 10,
    "employeeNumber": "11111",
    "name": "山田太郎",
    "employeeEmail": "employee@example.com",
    "departmentId": 4,
    "departmentName": "SALES",
    "positionId": 3,
    "positionName": "MANAGER",
    "hireDate": "2026-04-01",
    "status": "ACTIVE"
  },
  "message": "社員情報を登録しました。"
}
```

### ステータスコード

| コード | 条件                      |
| --- | ----------------------- |
| 201 | 登録成功                    |
| 400 | 入力値が不正                  |
| 401 | JWTが未指定または無効            |
| 403 | ADMIN権限がない              |
| 404 | ユーザー、部署または役職が存在しない      |
| 409 | ユーザーID、社員番号または社員用メールが重複 |
| 500 | 登録処理に失敗                 |

---

## 4.3 PUT /api/employees/{id}

管理者が社員情報を更新する。

URLの`id`には、Employeesテーブルの`id`を指定する。

### 認証・権限

| 認証 | 権限    |
| -- | ----- |
| 必要 | ADMIN |

### リクエストヘッダー

```http
Authorization: Bearer {token}
```

### パスパラメータ

| 項目名 | 型    | 必須 | 説明                    |
| --- | ---- | -- | --------------------- |
| id  | Long | ○  | 更新対象となる`Employees.id` |

### リクエスト

```json
{
  "employeeNumber": "11111",
  "name": "山田太郎",
  "employeeEmail": "employee@example.com",
  "departmentId": 4,
  "positionId": 3,
  "hireDate": "2026-04-01",
  "status": "ACTIVE"
}
```

### 更新対象

* 社員番号
* 氏名
* 社員用メールアドレス
* 部署
* 役職
* 入社日
* 在籍状態

### 更新対象外

* `Employees.id`
* `Employees.user_id`
* `Users.email`
* `Users.password`
* `Users.role`

### 更新ルール

* 社員用メールアドレスを更新しても、Usersテーブルのログインメールアドレスは更新しない。
* `departmentId`に対応する部署が存在しない場合はエラーとする。
* `positionId`に対応する役職が存在しない場合はエラーとする。
* 他の社員が使用している社員番号または社員用メールアドレスは設定できない。

### レスポンス

```json
{
  "status": "SUCCESS",
  "data": {
    "id": 1,
    "userId": 10,
    "employeeNumber": "11111",
    "name": "山田太郎",
    "employeeEmail": "employee@example.com",
    "departmentId": 4,
    "departmentName": "SALES",
    "positionId": 3,
    "positionName": "MANAGER",
    "hireDate": "2026-04-01",
    "status": "ACTIVE"
  },
  "message": "社員情報を更新しました。"
}
```

### ステータスコード

| コード | 条件                   |
| --- | -------------------- |
| 200 | 更新成功                 |
| 400 | 入力値が不正               |
| 401 | JWTが未指定または無効         |
| 403 | ADMIN権限がない           |
| 404 | 社員、部署または役職が存在しない     |
| 409 | 社員番号または社員用メールアドレスが重複 |
| 500 | 更新処理に失敗              |

---

## 4.4 DELETE /api/employees/{id}

管理者が社員情報を物理削除する。

Employeesテーブルのデータを削除するとともに、`Employees.user_id`に対応するUsersテーブルのデータも削除する。

### 認証・権限

| 認証 | 権限    |
| -- | ----- |
| 必要 | ADMIN |

### リクエストヘッダー

```http
Authorization: Bearer {token}
```

### パスパラメータ

| 項目名 | 型    | 必須 | 説明                    |
| --- | ---- | -- | --------------------- |
| id  | Long | ○  | 削除対象となる`Employees.id` |

### 削除処理

1. `Employees.id`を検索キーとして社員情報を取得する。
2. 取得した社員情報の`user_id`を保持する。
3. Employeesテーブルから社員情報を削除する。
4. Usersテーブルから対応するユーザー情報を削除する。
5. 一連の削除処理は同一トランザクション内で実行する。

### レスポンス

```json
{
  "status": "SUCCESS",
  "data": null,
  "message": "社員情報およびユーザー情報を削除しました。"
}
```

### ステータスコード

| コード | 条件           |
| --- | ------------ |
| 200 | 削除成功         |
| 401 | JWTが未指定または無効 |
| 403 | ADMIN権限がない   |
| 404 | 対象社員が存在しない   |
| 500 | 削除処理に失敗      |

---

## 4.5 GET /api/employees/me

JWTに含まれるUsersテーブルの`id`を使用して、ログインユーザー自身の社員情報を取得する。

`Users.id`と`Employees.user_id`を関連付けて社員情報を検索する。

### 認証・権限

| 認証 | 権限       |
| -- | -------- |
| 必要 | EMPLOYEE |

### リクエストヘッダー

```http
Authorization: Bearer {token}
```

### レスポンス

```json
{
  "status": "SUCCESS",
  "data": {
    "id": 1,
    "employeeNumber": "11111",
    "name": "山田太郎",
    "employeeEmail": "employee@example.com",
    "departmentId": 4,
    "departmentName": "SALES",
    "positionId": 3,
    "positionName": "MANAGER",
    "hireDate": "2026-04-01",
    "status": "ACTIVE"
  },
  "message": "自身の社員情報を取得しました。"
}
```

### ステータスコード

| コード | 条件             |
| --- | -------------- |
| 200 | 取得成功           |
| 401 | JWTが未指定または無効   |
| 403 | EMPLOYEE権限がない  |
| 404 | 対応する社員情報が存在しない |
| 500 | 取得処理に失敗        |

---

## 4.6 PUT /api/employees/me

ログインユーザー自身の氏名および社員用メールアドレスを更新する。

JWTに含まれるUsersテーブルの`id`と、Employeesテーブルの`user_id`を使用して更新対象を特定する。

### 認証・権限

| 認証 | 権限       |
| -- | -------- |
| 必要 | EMPLOYEE |

### リクエストヘッダー

```http
Authorization: Bearer {token}
```

### リクエスト

```json
{
  "name": "山田太郎",
  "employeeEmail": "employee-new@example.com"
}
```

### 更新可能項目

| 項目名           | 型      | 必須 | 説明                       |
| ------------- | ------ | -- | ------------------------ |
| name          | String | ○  | 氏名                       |
| employeeEmail | String | ○  | Employeesテーブルの社員用メールアドレス |

### 更新対象外

* ログインメールアドレス
* パスワード
* 権限
* 社員番号
* 部署
* 役職
* 入社日
* 在籍状態

### 更新ルール

* 更新するメールアドレスは`Employees.email`である。
* `Users.email`は更新しない。
* 他の社員が使用している社員用メールアドレスは設定できない。

### レスポンス

```json
{
  "status": "SUCCESS",
  "data": {
    "name": "山田太郎",
    "employeeEmail": "employee-new@example.com"
  },
  "message": "自身の社員情報を更新しました。"
}
```

### ステータスコード

| コード | 条件             |
| --- | -------------- |
| 200 | 更新成功           |
| 400 | 入力値が不正         |
| 401 | JWTが未指定または無効   |
| 403 | EMPLOYEE権限がない  |
| 404 | 対応する社員情報が存在しない |
| 409 | 社員用メールアドレスが重複  |
| 500 | 更新処理に失敗        |

---

## 4.7 POST /api/employees/import

CSVファイルを使用して、社員情報を一括登録または更新する。

社員番号を基準として登録または更新を判定する。

### 認証・権限

| 認証 | 権限    |
| -- | ----- |
| 必要 | ADMIN |

### リクエストヘッダー

```http
Authorization: Bearer {token}
Content-Type: multipart/form-data
```

### リクエスト

| 項目名  | 型       | 必須 | 説明               |
| ---- | ------- | -- | ---------------- |
| file | CSVファイル | ○  | 社員情報を記載したCSVファイル |

### CSV項目例

```csv
userId,employeeNumber,name,employeeEmail,departmentName,positionName,hireDate,status
10,11111,山田太郎,employee@example.com,SALES,MANAGER,2026-04-01,ACTIVE
```

### CSV処理ルール

* `employeeNumber`を検索キーとして新規登録または更新を判定する。
* 社員番号が存在しない場合は、新規登録する。
* 社員番号が存在する場合は、既存社員情報を更新する。
* 新規登録時は、対応する`userId`がUsersテーブルに存在する必要がある。
* 新規登録時は、すでに他のEmployeesデータと関連付けられている`userId`を使用できない。
* CSVインポートではUsersテーブルのログイン情報を新規登録しない。
* `departmentName`からDepartmentsテーブルを検索し、取得したIDを`department_id`へ設定する。
* `positionName`からPositionsテーブルを検索し、取得したIDを`position_id`へ設定する。
* 部署名がDepartmentsテーブルに存在しない場合はエラーとする。
* 役職名がPositionsテーブルに存在しない場合はエラーとする。
* `Employees.id`はデータベースで自動採番する。
* CSVの登録・更新判定には`Employees.id`を使用しない。

### レスポンス

```json
{
  "status": "SUCCESS",
  "data": {
    "registeredCount": 5,
    "updatedCount": 3
  },
  "message": "CSVの取り込みが完了しました。"
}
```

### ステータスコード

| コード | 条件                     |
| --- | ---------------------- |
| 200 | CSV取り込み成功              |
| 400 | CSV形式または入力値が不正         |
| 401 | JWTが未指定または無効           |
| 403 | ADMIN権限がない             |
| 404 | ユーザー、部署または役職が存在しない     |
| 409 | ユーザーIDまたは社員用メールアドレスが重複 |
| 500 | CSV取り込み処理に失敗           |

---

# 5. 部署マスターAPI

## 5.1 GET /api/departments

Departmentsテーブルに登録されている部署一覧を取得する。

### リクエストヘッダー

```http
Authorization: Bearer {token}
```

### レスポンス

```json
{
  "status": "SUCCESS",
  "data": [
    {
      "id": 1,
      "name": "GENERAL"
    },
    {
      "id": 2,
      "name": "HR"
    },
    {
      "id": 3,
      "name": "PLANNING"
    },
    {
      "id": 4,
      "name": "SALES"
    }
  ],
  "message": "部署一覧を取得しました。"
}
```

---

## 5.2 POST /api/departments

管理者が部署情報を新規登録する。

### リクエスト

```json
{
  "name": "DEVELOPMENT"
}
```

### レスポンス

```json
{
  "status": "SUCCESS",
  "data": {
    "id": 5,
    "name": "DEVELOPMENT"
  },
  "message": "部署情報を登録しました。"
}
```

### 登録ルール

* すでに登録されている部署名は登録できない。
* 部署名は50文字以内とする。

---

## 5.3 PUT /api/departments/{id}

管理者が部署名を更新する。

URLの`id`には、Departmentsテーブルの`id`を指定する。

### リクエスト

```json
{
  "name": "SYSTEM_DEVELOPMENT"
}
```

### レスポンス

```json
{
  "status": "SUCCESS",
  "data": {
    "id": 5,
    "name": "SYSTEM_DEVELOPMENT"
  },
  "message": "部署情報を更新しました。"
}
```

### 更新ルール

* 対象となる部署が存在しない場合はエラーとする。
* 他の部署と同じ部署名には更新できない。
* 部署IDは変更しない。

---

# 6. 役職マスターAPI

## 6.1 GET /api/positions

Positionsテーブルに登録されている役職一覧を取得する。

### リクエストヘッダー

```http
Authorization: Bearer {token}
```

### レスポンス

```json
{
  "status": "SUCCESS",
  "data": [
    {
      "id": 1,
      "name": "STAFF"
    },
    {
      "id": 2,
      "name": "LEADER"
    },
    {
      "id": 3,
      "name": "MANAGER"
    },
    {
      "id": 4,
      "name": "DIRECTOR"
    },
    {
      "id": 5,
      "name": "EXECUTIVE"
    },
    {
      "id": 6,
      "name": "CEO"
    }
  ],
  "message": "役職一覧を取得しました。"
}
```

---

## 6.2 POST /api/positions

管理者が役職情報を新規登録する。

### リクエスト

```json
{
  "name": "SUB_MANAGER"
}
```

### レスポンス

```json
{
  "status": "SUCCESS",
  "data": {
    "id": 7,
    "name": "SUB_MANAGER"
  },
  "message": "役職情報を登録しました。"
}
```

### 登録ルール

* すでに登録されている役職名は登録できない。
* 役職名は50文字以内とする。

---

## 6.3 PUT /api/positions/{id}

管理者が役職名を更新する。

URLの`id`には、Positionsテーブルの`id`を指定する。

### リクエスト

```json
{
  "name": "ASSISTANT_MANAGER"
}
```

### レスポンス

```json
{
  "status": "SUCCESS",
  "data": {
    "id": 7,
    "name": "ASSISTANT_MANAGER"
  },
  "message": "役職情報を更新しました。"
}
```

### 更新ルール

* 対象となる役職が存在しない場合はエラーとする。
* 他の役職と同じ役職名には更新できない。
* 役職IDは変更しない。

---

# 7. テーブルとの対応

## 7.1 Usersテーブルへの登録

| API項目      | テーブルカラム          |
| ---------- | ---------------- |
| loginEmail | `users.email`    |
| password   | `users.password` |
| role       | `users.role`     |

---

## 7.2 Employeesテーブルへの登録

| API項目          | テーブルカラム                     |
| -------------- | --------------------------- |
| userId         | `employees.user_id`         |
| employeeNumber | `employees.employee_number` |
| name           | `employees.name`            |
| employeeEmail  | `employees.email`           |
| departmentId   | `employees.department_id`   |
| positionId     | `employees.position_id`     |
| hireDate       | `employees.hire_date`       |
| status         | `employees.status`          |

---

## 7.3 テーブル間の関連

```text
users.id
   ↑
employees.user_id
```

```text
departments.id
   ↑
employees.department_id
```

```text
positions.id
   ↑
employees.position_id
```

* UsersテーブルとEmployeesテーブルは、`Users.id`と`Employees.user_id`で関連付ける。
* DepartmentsテーブルとEmployeesテーブルは、`Departments.id`と`Employees.department_id`で関連付ける。
* PositionsテーブルとEmployeesテーブルは、`Positions.id`と`Employees.position_id`で関連付ける。
* `Users.id`と`Employees.id`は直接関連付けない。
* 社員情報、部署情報および役職情報をまとめて取得する場合は、JOINまたはJPAの関連付けを使用する。
