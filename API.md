# API
AdColony配信実績取得用 API仕様書

## Version
1.0

## 1. API一覧

* アプリ一覧取得 ※1
* 枠一覧取得 ※1
* メディアレポート取得 ※1
* キャンペーン一覧取得
* キャンペーン作成
* キャンペーン変更
* キャンペーン取消

※1: ログインが必要なAPI、アカウントについては、2. ログインアカウントをご参照ください

それぞれのAPIの詳細仕様は4.以降をご参照ください

## 2. ログインアカウント

- 弊社レポートツール(https://adcolony.glossom.jp/tool/partner) のログインアカウントと同じ物がAPIでも利用可能です。
- 現状、ログインアカウントをお持ちではないが、レポートツールのご利用をご希望される場合は、http://video-ad.glossom.jp/#contact からお問い合わせください。

## 3. 全API共通 リクエスト・レスポンス

### 3.1. リクエスト
- パラメータの日本語はUTF-8でエンコーディングする必要があります。
- ログインセッションはCookieにより管理されているので、APIを叩く際はクライアント側でCookieを保持してください
- ログインが必要なAPIについて、セッションの有効期限切れもしくはサーバ側のセッションデータが削除されたあとに各APIにアクセスした場合は、認証エラー(HTTP 403)が返答されます。
- セッションの有効期限は一日です
- 認証エラーが起きた場合はログインしなおしてください
- リクエストは20分に1回まで可能です

### 3.2. レスポンス
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
    <td>401 Unauthorized</td>
    <td>APIログイン認証に関するエラー</td>
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

## 4. API 詳細仕様

### 4.1. ログイン
URL: https://adcolony.glossom.jp/api/v1/users/sign_in

ログインユーザーの情報とクッキーが取得できます。

#### リクエストパラメータ

##### user[email]
ユーザーのメールアドレス

##### user[password]
ユーザーのパスワード

##### リクエスト例

curl -c cookie.txt -d user[email]=EMAIL -d user[password]=PASS https://adcolony.glossom.jp/api/v1/users/sign_in

#### レスポンス

```
# ログインに成功した場合
{"success":"ログインしました"}
# ログインに失敗した場合
{"errors":[{"message":"メールアドレスまたはパスワードが違います。"}]}
# セッションが切れた場合
{"errors":[{"message":"セッションがタイムアウトしました。もう一度ログインしてください。"}]}
```

### 4.2. アプリ一覧取得
URL: https://adcolony.glossom.jp/api/v1/apps

メディアの持つアプリ一覧が取得できます。

#### リクエストパラメータ

特に無し

##### リクエスト例

curl -b cookie.txt https://adcolony.glossom.jp/api/v1/apps

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

### 4.3. 枠一覧取得
URL: https://adcolony.glossom.jp/api/v1/zones

#### リクエストパラメータ

特に無し

##### リクエスト例

curl -b cookie.txt https://adcolony.glossom.jp/api/v1/zones

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

### 4.4. メディアレポート取得
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

curl -b cookie.txt -XGET -d month=2014/12 https://adcolony.glossom.jp/api/v1/publisher/reports

#### レスポンス

メディア向け日別レポート情報をjson形式で返します。

jsonのsampleとデータ型に対する説明です。

```
{
  "2014-12-31": {            # 発生した日付 yyyy-mm-dd のDate型
    "earnings_yen": "8.14",  # 収益(円) Float型
    "requests": 172,         # 広告リクエスト数 INT型
    "impressions": 175,      # 広告表示回数 INT型
    "cvvs": 147,             # 動画視聴完了数 INT型
    "clicks": 1,             # クリック数 INT型
    "fill_rate": "101.74",   # Fill Rate(%) Float型
    "ecpm": "55.38"          # eCPM(円) Float型
  },
  "2014-12-30": {
    "earnings_yen": "13.70",
    "requests": 257,
    "impressions": 264,
    "cvvs": 238,
    "clicks": 1,
    "fill_rate": "102.72",
    "ecpm": "57.56"
  },
  "total": {
    "earnings_yen": "630.00",
    "requests": 4616,
    "impressions": 4710,
    "cvvs": 4264,
    "clicks": 33,
    "fill_rate": "102.04",
    "ecpm": "147.63"
  }
}
```

### 4.5. キャンペーン一覧取得

URL: https://adcolony.glossom.jp/api/v1/campaigns

#### リクエストパラメータ

| 名前 | 意味 | 必須 | サンプル |
| ---- | ---- | ---- | -------- |
| id | キャンペーンID | NO | 100 |
| status | ステータス | NO | [キャンペーン状態シンボル](STATE.md) を参照 |
| platform | プラットフォーム | NO | iOS or Android |
| bid_type | 単価種別 | NO | cpi or cpcv |

##### リクエスト例

curl -b cookie.txt https://adcolony.glossom.jp/api/v1/campaigns?status=delivery&platform=ios

#### レスポンス

代理店/クライアントの持つキャンペーン一覧をjson形式で返します。

jsonのsampleとデータ型に対する説明です。

```
{
  {
    "id": 137,                                   # キャンペーンID(ユニーク) INT型
    "name": "キャンペーン1",                     # キャンペーン名 String型(255文字まで)
	"status": "delivery",                        # ステータス
	"start_date": "2015-03-05",                  # 配信開始日
    "end_date": "2015-03-15",                    # 配信終了日
    "bid": 100,                                  # Bid(円)
    "bid_type": "cpi",                           # Bid type
    "total_budget": 10000000,                    # 全体予算上限(円)
    "daily_budget": 1000000,                     # 日予算上限(円)
    "platform": "Android",                       # プラットフォーム
    "created_at": "2015-03-05T05:00:19.000Z",    # 作成日時(UTC) DateTime型
    "updated_at": "2015-03-05T05:20:19.000Z"     # 更新日時(UTC) DateTime型
  },
  {
    "id": 139,
    "name": "キャンペーン2",
	"status": "staged",
	"start_date": "2015-03-05",
    "end_date": "2015-03-25",
    "bid": 200,
    "bid_type": "cpcv",
    "total_budget": 20000000,
    "daily_budget": 2000000,
    "platform": "iOS",
    "created_at": "2015-03-05T05:00:19.000Z",
    "updated_at": "2015-03-05T05:20:19.000Z"
  }
}
```

### 4.6. キャンペーン登録

URL: https://adcolony.glossom.jp/api/v1/campaigns

#### HTTP メソッド

POST

#### リクエストパラメータ

| 名前 | 意味 | 必須 | 仕様 | サンプル |
| ---- | ---- | ---- | ---- | -------- |
| name | キャンペーン名 | YES | x文字以内 | キャンペーン |
| start_date | キャンペーン開始日時 | YES | YYYY/MM/DD | 2015/04/01 |
| end_date | キャンペーン終了日時 | YES | YYYY/MM/DD | 2015/12/31 |
| bid | 単価 | YES | 単位は円、少数点x位以下は切り捨て。| 100 |
| bid_type | 単価種別 | YES | "cpi" or "cpcv" | cpi |
| total_budget | キャンペーン総予算(円) | YES | INT | 1000000 |
| daily_budget | キャンペーン日予算(円) | YES | INT | 100000 |
| platform | プラットフォーム | YES | "iOS" or "Android" | iOS |

##### リクエスト例

```
curl https://adcolony.glossom.jp/api/v1/campaigns \
-b cookie.txt \
--request POST \
--header 'Content-type: application/json' \
-d camapign[name]=キャンペーン \
-d campaign[start_date]=2015/09/01 \
-d campaign[end_date]=2015/12/31 \
-d campaign[bid]=100 \
-d campaign[bid_type]=CPM \
-d campaign[total_budget]=100000000 \
-d campaign[daily_budget]=1000000 \
-d campaign[platform]=iOS
```

#### レスポンス

* 成功の場合、レスポンスは無し。
* 失敗の場合、エラーレスポンスを返却。

### 4.7. キャンペーン確定

URL: https://adcolony.glossom.jp/api/v1/campaigns/{camapign_id}/fix

#### HTTP メソッド

POST

#### リクエストパラメータ

特に無し

##### リクエスト例

```
curl https://adcolony.glossom.jp/api/v1/campaigns/100/fix \
-b cookie.txt \
--request POST
```

#### レスポンス

* 成功の場合、レスポンスは無し。
* 失敗の場合、エラーレスポンスを返却。

### 4.8. キャンペーン更新

URL: https://adcolony.glossom.jp/api/v1/campaigns/{camapign_id}

指定したキャンペーン情報を更新することが出来ます。

#### 前提条件

キャンペーンの状態が以下の場合、エラーレスポンスを返却します。

* 作成中
* 否認済み
* 変更中

更新後の状態遷移は、[キャンペーン状態遷移図](STATE.md) を参照ください。

#### HTTP メソッド

PUT

#### リクエストパラメータ

| 名前 | 意味 | 必須 | サンプル |
| ---- | ---- | ---- | -------- |
| name | キャンペーン名 | x文字以内 | キャンペーン |
| start_date | キャンペーン開始日時 | YYYY/MM/DD | 2015/04/01 |
| end_date | キャンペーン終了日時 | YYYY/MM/DD | 2015/12/31 |
| bid | 単価 | 単位は円、少数点x位以下は切り捨て。| 100 |
| bid_type | 単価種別 | "cpi" or "cpcv" | cpi |
| total_budget | キャンペーン総予算(円) | INT | 1000000 |
| daily_budget | キャンペーン日予算(円) | INT | 100000 |
| platform | プラットフォーム | "iOS" or "Android" | iOS |

##### リクエスト例

```
curl https://adcolony.glossom.jp/api/v1/campaigns/100 \
-b cookie.txt \
--request PUT \
--header 'Content-type: application/json' \
-d campaign[end_date]=2015/12/31 \
-d campaign[total_budget]=1000000000 \
```

#### レスポンス

```
# 更新に成功した場合
* HTTP Status Code: 200
* HTTP Response Body: {"success":"ログインしました"}
# 更新の前提条件を満たさなかった場合
* HTTP Status Code: 409
* HTTP Response Body: `{"errors":[{"message":"更新可能な状態ではありません。"}]}`
```

### 4.9. キャンペーン取消

URL: https://adcolony.glossom.jp/api/v1/campaigns/{camapign_id}/cancel

指定したキャンペーンを取り消すことが出来ます。  

取消後の状態遷移は、[キャンペーン状態遷移図](STATE.md) を参照ください。

#### 前提条件

キャンペーンの状態が以下の場合、エラーレスポンスを返却します。

* 作成中
* 変更中

該当キャンペーンの状態が上記の条件を満たさない場合、HTTPステータスコード `409` を返します。

#### HTTP メソッド

POST

#### リクエストパラメータ

無し

##### リクエスト例

```
curl https://adcolony.glossom.jp/api/v1/campaigns/100/cancel \
-b cookie.txt \
--request POST
```

#### レスポンス

```
# 取消に成功した場合
* HTTP Status Code: 200
* HTTP Response Body: {"success":"ログインしました"}
# 取消の前提条件を満たさなかった場合
* HTTP Status Code: 409
* HTTP Response Body: `{"errors":[{"message":"取消可能な状態ではありません。"}]}`
```
