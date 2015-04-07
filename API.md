# API
AdColony配信実績取得用 API仕様書

## Version
0.9

## 1. API一覧

* アプリ一覧取得 ※1
* 枠一覧取得 ※1
* メディアレポート取得 ※1

※1: ログインが必要なAPIです

それぞれのAPIの詳細仕様は3.以降をご参照ください

## 2. 全API共通 リクエスト・レスポンス

### 2.1. リクエスト
- パラメータの日本語はUTF-8でエンコーディングする必要があります。
- ログインセッションはCookieにより管理されているので、APIを叩く際はクライアント側でCookieを保持してください
- ログインが必要なAPIについて、セッションの有効期限切れもしくはサーバ側のセッションデータが削除されたあとに各APIにアクセスした場合は、認証エラー(HTTP 403)が返答されます。
- 認証エラーが起きた場合はログインしなおしてください

### 2.2. レスポンス
HTTP Response BodyはJSON形式となります。

HTTP Status Codeは以下を意味します

<table>
  <tr>
    <td>Status Code</td>
    <td>意味</td>
  </tr>
  <tr>
    <td>200 ok</td>
    <td>Requestは全て正常に処理された</td>
  </tr>
  <tr>
    <td>400 Bad Request</td>
    <td>リクエストの必須項目が不足している等、リクエスト側の問題</td>
  </tr>
  <tr>
    <td>403 Forbidden</td>
    <td>API利用認可コードが不正</td>
  </tr>
  <tr>
    <td>404 Not Found</td>
    <td>該当のデータやリソースがない</td>
  </tr>
  <tr>
    <td>500 Internal Server Error</td>
    <td>APIサーバー側エラーが発生した</td>
  </tr>
</table>

エラーの場合JSONのerrorsパラメータの配列にエラー要因の文字列が返されます。

例
```
{
  "errors": [
    { "message": "error_messageの内容" },
    { "message": "error_messageの内容" },
    { "message": "error_messageの内容" }
  }
}
```

## 3. API 詳細仕様

### 3.1. ログイン
URL: https://adcolony.glossom.jp/api/v1/users/sign_in

ログインユーザーの情報とクッキーが取得できます。

#### リクエストパラメータ

##### user[email]
ユーザーのメールアドレス

##### user[password]
ユーザーのパスワード

##### リクエスト例

https://adcolony.glossom.jp/api/v1/users/sign_in?user[email]=hoge@example.com&user[password]=PASS

#### レスポンス

アカウント情報がJSONで返ります。

```
{
  "id":1,
  "name":"hoge",
  "account_id":1,
  "status":1,
  "created_at":"2015-03-05T08:11:57.000Z",
  "updated_at":"2015-04-07T10:29:02.990Z",
  "email":"hoge@example.com",
  "password_set_by_admin":null
}
```

### 3.2. ログアウト
URL: https://adcolony.glossom.jp/api/v1/users/sign_out

DELETEでリクエストしてください。

#### リクエストパラメータ

特になし

##### リクエスト例

https://adcolony.glossom.jp/api/v1/users/sign_out

#### レスポンス

特になし

### 3.3. アプリ一覧取得
URL: https://adcolony.glossom.jp/api/v1/apps

メディアの持つアプリ一覧が取得できます。

#### リクエストパラメータ

特に無し

##### リクエスト例

https://adcolony.glossom.jp/api/v1/apps

#### レスポンス

メディアの持つアプリ一覧をjson形式で返します。

jsonのsampleとデータ型に対する説明です。

```
{
  {
    "id": 1,                                   # アプリID(ユニーク) INT型
    "account_id": 1,                           # アカウントID INT型
    "name": "test_GREE_Android",               # アプリ名(adcolony側)  String型(255文字まで)              
    "created_at": "2015-03-05T05:00:19.000Z",  # 作成日時(UTC) DateTime型
    "updated_at": "2015-03-05T05:00:19.000Z"   # 更新日時(UTC) DateTime型
  },
  {
    "id": 2,
    "account_id": 1,
    "name": "test_GREE_iOS",
    "created_at": "2015-03-05T05:00:19.000Z",
    "updated_at": "2015-03-05T05:00:19.000Z"
  }
}
```

### 3.4. 枠一覧取得
URL: https://adcolony.glossom.jp/api/v1/zones

#### リクエストパラメータ

特に無し

##### リクエスト例

https://adcolony.glossom.jp/api/v1/zones

#### レスポンス

メディアの持つアプリ一覧をjson形式で返します。

jsonのsampleとデータ型に対する説明です。

```
{
  {
    "id": 137,                                   # 枠ID(ユニーク) INT型
    "app_id": 105,                               # アプリID INT型
    "name": "wirecat01",                         # 枠名(adcolony側)  String型(255文字まで)
    "created_at": "2015-03-05T05:00:19.000Z",    # 作成日時(UTC) DateTime型
    "updated_at": "2015-03-05T05:20:19.000Z"     # 更新日時(UTC) DateTime型
  },
  {
    "id": 135,
    "app_id": 103,
    "name": "wirecat01",
    "created_at": "2015-03-05T05:00:19.000Z",
    "updated_at": "2015-03-05T05:20:19.000Z"
  }
}
```

### 3.5. メディアレポート取得
URL: https://adcolony.glossom.jp/api/v1/publisher/reports

メディア向け日別レポート情報が取得できます。

#### リクエストパラメータ

すべてGETで指定してください。

##### month
* yyyy/mmの日付形式 例 2015/05
* yyyy/mm/ddの日付形式 例 2015/05/01 ※ ddは解釈されません
* yyyy-mm-ddの日付形式 例 2015-05-01 ※ ddは解釈されません

取得したい対象月を指定します。

##### app_id
INT型

取得したい対象のアプリIDが入ります。

アプリIDは「アプリ一覧取得」から取得してください。

##### zone_id
INT型

取得したい対象の枠IDが入ります。

枠IDは「枠一覧取得」から取得してください。

##### リクエスト例

https://adcolony.glossom.jp/api/v1/publisher/reports?month=2015-05-01&zone_id=1

#### レスポンス

メディア向け日別レポート情報をjson形式で返します。

jsonのsampleとデータ型に対する説明です。

```
{
  "2015-03-03": {         # 発生した日付 yyyy-mm-dd のDate型
    "earnings_yen": 0,    # 収益(円) Float型
    "requests": 0,        # 広告リクエスト数 INT型
    "impressions": 0,     # 広告表示回数 INT型
    "cvvs": 0,            # 動画視聴完了数 INT型
    "clicks": 0,          # クリック数 INT型
    "fill_rate": 0,       # Fill Rate(%) Float型
    "ecpm": 0             # eCPM(円) Float型
  },
  "2015-03-02": {
    "earnings_yen": 406.80468,
    "requests": 441,
    "impressions": 449,
    "cvvs": 408,
    "clicks": 4,
    "fill_rate": 101.81405895691611,
    "ecpm": 997.0702941176471
  },
  "2015-03-01": {
    "earnings_yen": 796.97898,
    "requests": 700,
    "impressions": 735,
    "cvvs": 643,
    "clicks": 14,
    "fill_rate": 105,
    "ecpm": 1239.4696423017106
  },
  "total": {                           # 合算
    "earnings_yen": 1203.78366,        
    "requests": 1141,                  
    "impressions": 1184,               
    "cvvs": 1051,                      
    "clicks": 18,                      
    "fill_rate": 103.76862401402278,   
    "ecpm": 1145.3698001902949         
  }
}
```
