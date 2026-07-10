# 社員管理システム

## 概要

社員情報を管理するWebアプリケーションです。
管理者と社員で利用できる機能を分けており、
JWT認証によるログイン機能を実装しています。

---

## 主な機能

### 共通
- ログイン
- JWT認証
- ログアウト

### 管理者
- 社員一覧取得
- 社員登録
- 社員更新
- 社員削除
- CSV一括登録

### 社員
- 自身の社員情報取得
- 自身の社員情報更新

---

## 使用技術

### Frontend
- React
- TypeScript
- React Router

### Backend
- Spring Boot
- Spring Security
- JWT
- Spring Data JPA

### Database
- PostgreSQL

---

## システム構成

Frontend
↓
Spring Boot API
↓
PostgreSQL

---

## 起動方法

### Backend

```bash
cd backend
./mvnw spring-boot:run
```

### Frontend

```bash
cd frontend
npm install
npm start
```

---

## API一覧

| メソッド | URL | 内容 |
|----------|-----|------|
| POST | /api/auth/login | ログイン |
| POST | /api/auth/register | ユーザー登録 |
| GET | /api/employees | 社員一覧取得 |
| GET | /api/employees/me | 自身の情報取得 |
| PUT | /api/employees/{employeeNumber} | 社員登録・更新 |
| DELETE | /api/employees/{employeeNumber} | 社員削除 |
| POST | /api/employees/import | CSV登録 |

---

## ディレクトリ構成

```
group-assignment
├── backend
├── frontend
└── 設計書
```

---

## 作成者

益山 哲太
