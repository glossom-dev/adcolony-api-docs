# API
AdColony配信実績取得用 API仕様書

## Version
1.0

## 1. API一覧

### Publisher様向けAPI

- [アプリ一覧取得 ※1](#fetch_app)
- [枠一覧取得 ※1](#fetch_zones)
- [メディアレポート取得 ※1](#fetch_zone_report)

### 広告代理店/広告主様向けAPI

- [キャンペーン一覧取得 ※1](#fetch_campaigns)
- [グループ一覧取得 ※1](#fetch_groups)
- [キャンペーンレポート取得 ※1](#fetch_campaign_report)

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

### ログイン
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

### <a id="fetch_app"></a>アプリ一覧取得
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

### <a id="fetch_zones"></a>枠一覧取得
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


### <a id="fetch_zone_report"></a>メディアレポート取得
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
  "2014-12-31": {             # 発生した日付 yyyy-mm-dd のDate型
    "earnings_yen": "800.23", # 収益(円) 小数点第三位以下を四捨五入
    "requests": 172,          # 広告リクエスト数 INT型
    "impressions": 175,       # 広告表示回数 INT型
    "cvvs": 147,              # 動画視聴完了数 INT型
    "clicks": 1,              # クリック数 INT型
    "fill_rate": "101.74",    # Fill Rate(%) Float型
    "ecpm": "55.38"           # eCPM(円) Float型
  },
  "2014-12-30": {
    "earnings_yen": "1370.23",
    "requests": 257,
    "impressions": 264,
    "cvvs": 238,
    "clicks": 1,
    "fill_rate": "102.72",
    "ecpm": "57.56"
  },
  "total": {
    "earnings_yen": "630.90", # 収益合算値(円) 小数点第三位以下を四捨五入)
    "requests": 4616,
    "impressions": 4710,
    "cvvs": 4264,
    "clicks": 33,
    "fill_rate": "102.04",
    "ecpm": "147.63"
  }
}
```

### <a id="fetch_campaigns"></a>キャンペーン一覧取得

URL: https://adcolony.glossom.jp/api/v1/campaigns

#### リクエストパラメータ

| 名前 | 意味 | 必須 | サンプル |
| ---- | ---- | ---- | -------- |
| page | ページ番号(デフォルト: 1) | NO | 10 |
| per_page | 1ページ当たりの件数(デフォルト: 100) | NO | 50 |

##### リクエスト例

curl -b cookie.txt https://adcolony.glossom.jp/api/v1/campaigns

#### レスポンス

広告主の持つキャンペーン一覧をjson形式で返します。

jsonのsampleとデータ型に対する説明です。

```
{
  {
    "id": 100,                                   # キャンペーンID(ユニーク) INT型
    "account_id": 1,                             # アカウントID INT型
    "name": "test",                              # キャンペーン名(adcolony側)  String型(255文字まで)
    "created_at": "2015-03-05T05:00:19.000Z",    # 作成日時(UTC) DateTime型
    "updated_at": "2015-03-05T05:20:19.000Z"     # 更新日時(UTC) DateTime型
  },
  {
    "id": 101,
    "account_id": 1,
    "name": "test2",
    "created_at": "2015-03-05T05:00:19.000Z",
    "updated_at": "2015-03-05T05:20:19.000Z"
  }
}
```

### <a id="fetch_groups"></a>グループ一覧取得

URL: https://adcolony.glossom.jp/api/v1/groups

#### リクエストパラメータ

| 名前 | 意味 | 必須 | サンプル |
| ---- | ---- | ---- | -------- |
| page | ページ番号(デフォルト: 1) | NO | 10 |
| per_page | 1ページ当たりの件数(デフォルト: 100) | NO | 50 |

##### リクエスト例

curl -b cookie.txt https://adcolony.glossom.jp/api/v1/groups

#### レスポンス

広告主の持つグループ一覧をjson形式で返します。

jsonのsampleとデータ型に対する説明です。

```
{
  {
    "id": 100,                                   # グループID(ユニーク) INT型
    "campaign_id": 1054,                         # キャンペーンID INT型
    "name": "test",                              # グループ名(adcolony側)  String型(255文字まで)
    "created_at": "2015-03-05T05:00:19.000Z",    # 作成日時(UTC) DateTime型
    "updated_at": "2015-03-05T05:20:19.000Z"     # 更新日時(UTC) DateTime型
  },
  {
    "id": 101,                                   # グループID(ユニーク) INT型
    "campaign_id": 1090,                         # キャンペーンID INT型
    "name": "test",                              # グループ名(adcolony側)  String型(255文字まで)
    "created_at": "2015-03-05T05:00:19.000Z",    # 作成日時(UTC) DateTime型
    "updated_at": "2015-03-05T05:20:19.000Z"     # 更新日時(UTC) DateTime型
  }
}
```

### <a id="fetch_campaign_report"></a>キャンペーンレポート取得

URL: https://adcolony.glossom.jp/api/v1/advertiser/reports

広告主向け日別レポート情報が取得できます。

#### リクエストパラメータ

すべてGETで指定してください。

なお、`campaign_id` と `group_id` を同時に指定した場合、`group_id` が検索条件として採用されます。  
また、`per_page` に`1`を指定した場合、該当日の結果に、`total`の結果を付与した形で返却されます。  
`month` を省略した場合、リクエスト当月のレポート結果を返却します。

| 名前 | 意味 | 必須 | サンプル |
| ---- | ---- | ---- | -------- |
| month | 取得する日付 | NO | <ul><li>yyyy/mmの日付形式 例 2015/05</li><li>yyyy/mm/ddの日付形式 例 2015/05/01 ※ ddは解釈されません</li><li>yyyy-mm-ddの日付形式 例 2015-05-01 ※ ddは解釈されません</li></ul>|
| campaign_id | キャンペーンID | NO |<ul><li>取得したい対象のキャンペーンIDが入ります。</li><li>キャンペーンIDは「キャンペーン一覧取得」から取得してください。</li></ul>|
| group_id | グループID | NO |<ul><li>取得したい対象のグループIDが入ります。</li><li>グループIDは「グループ一覧取得」から取得してください。</li></ul>|
| page | ページ番号(デフォルト: 1) | NO | 10 |
| per_page | 1ページ当たりの件数(デフォルト: 100) | NO | 50 |

##### リクエスト例

curl -b cookie.txt -XGET -d month=2014/12 https://adcolony.glossom.jp/api/v1/advertiser/reports

#### レスポンス

広告主向け日別レポート情報をjson形式で返します。

jsonのsampleとデータ型に対する説明です。

```
{
  "2014-12-31": {            # 発生した日付 yyyy-mm-dd のDate型
    "spend_yen": "15395.12", # 消化金額(円) 小数点第三位以下を四捨五入
    "impressions": 100,      # 広告表示回数 INT型
    "cvvs": 147,             # 動画視聴完了数 INT型
    "clicks": 1000,          # クリック数 INT型
    "installs": 100,         # インストール数 INT型
    "ctr": "0.85",           # CTR Float型
    "cvr": "26.85",          # CVR Float型
    "cpi": "332.88"          # CPI(円) Float型
  },
  "2014-12-30": {
    "spend_yen": "20000.20",
    "impressions": 100,
    "cvvs": 147,
    "clicks": 1000,
    "installs": 100,
    "ctr": "0.85",
    "cvr": "26.85",
    "cpi": "332.88"
  },
  "total": {
    "spend_yen": "35395.42", # 消化金額合算値(円) 小数点第三位以下を四捨五入)
    "impressions": 100,
    "cvvs": 147,
    "clicks": 1000,
    "installs": 100,
    "ctr": "0.85",
    "cvr": "26.85",
    "cpi": "332.88"
  }
}
```

配信結果が存在しない場合、以下のレスポンスを返却します。

```
{
  "total": {
    "spend_yen": "0",
    "impressions": 0,
    "cvvs": 0,
    "clicks": 0,
    "installs": 0,
    "ctr": "0.0",
    "cvr": "0.0",
    "cpi": "0.0"
  }
}
```

## 改定履歴

[CHANGELOG](CHANGELOG.md)を参照
