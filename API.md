# API
AdColony配信実績取得用 API仕様書

## Version
1.0

## 1. API一覧

* レポート系
  * [アプリ一覧取得※1](#fetch_app)
  * [枠一覧取得※1](#fetch_zones)
  * [メディアレポート取得※1](#fetch_zone_report)
  * [キャンペーン一覧取得※1](#fetch_campaigns)
  * [クリエイティブ一覧取得※1](#fetch_creatives)
* キャンペーン操作系
  * [クリエイティブアップロード※1](#upload_creative)
  * [掲載可否審査依頼※1](#judge_campaign)
  * [キャンペーン登録※1](#create_campaign)
  * [配信設定変更依頼※1](#update_campaign)
  * [配信設定変更取消※1](#cancel_campaign)

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
    <td>409 Conflict</td>
    <td>リクエストと、サーバに既に存在しているデータが競合している</td>
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

--

### <a id="fetch_app"></a>4.2. アプリ一覧取得
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

--

### <a id="fetch_zones"></a>4.3. 枠一覧取得
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

--

### <a id="fetch_zone_report"></a>4.4. メディアレポート取得
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

--

### <a id="fetch_campaigns"></a>4.5. キャンペーン一覧取得

URL: https://adcolony.glossom.jp/api/v1/campaigns

#### リクエストパラメータ

| 名前 | 意味 | 必須 | サンプル |
| ---- | ---- | ---- | -------- |
| id | キャンペーンID | NO | 100 |
| status | ステータス | NO | [キャンペーン状態一覧のシンボル](STATE.md) を参照 |
| platform | プラットフォーム | NO | iOS or Android |
| bid_type | 単価種別 | NO | cpi or cpcv |
| page | ページ番号(デフォルト: 1) | NO | 10 |
| per_page | 1ページ当たりの件数(デフォルト: 20、最高: 100) | NO | 50 |

##### リクエスト例

curl -b cookie.txt https://adcolony.glossom.jp/api/v1/campaigns?status=working&platform=ios

#### レスポンス

キャンペーン一覧をjson形式で返します。

jsonのsampleとデータ型に対する説明です。

```
HTTP/1.1 200
Content-Type: application/json

{
  {
    "id": 137,                                   # キャンペーンID(ユニーク) INT型
    "name": "キャンペーン1",                     # キャンペーン名 String型(255文字まで)
	"status": "working",                        # ステータス
	"start_date": "2015-03-05",                  # 配信開始日
    "end_date": "2015-03-15",                    # 配信終了日
    "bid": 100,                                  # Bid(円)
    "bid_type": "cpi",                           # Bid type
    "total_budget": 10000000,                    # 全体予算上限(円)
    "daily_budget": 1000000,                     # 日予算上限(円)
    "platform": "Android",                       # プラットフォーム
	"os": "Android",                             # 配信先OS
    "device": [                                  # 配信デバイス(リスト)
	  "Phone",
	  "Tablet"
	],
    "country": "JP",                             # 配信国
    "url": "https://play.google.com/store/apps/details?id=com.android.chrome", # ストアURL or クリック後遷移先URL
    "tracking": [                                # トラッキングツール(リスト)
	  "test",
	  "test2"
	],
    "note": "test message",                      # 備考
    "creative_id": [                             # クリエイティブID(リスト)
      1000,
      1001
    ],
    "created_at": "2015-03-05T05:00:19.000Z",    # 作成日時(UTC) DateTime型
    "updated_at": "2015-03-05T05:20:19.000Z"     # 更新日時(UTC) DateTime型
  },
  {
    "id": 139,
    "name": "キャンペーン2",
	"status": "passed",
	"start_date": "2015-03-05",
    "end_date": "2015-03-25",
    "bid": 200,
    "bid_type": "cpcv",
    "total_budget": 20000000,
    "daily_budget": 2000000,
    "platform": "iOS",
    "os": "iOS",
	"device": [
	  "iPad",
	  "iPhone",
	  "iPod"
    ],
	"country": "US",
    "url": "https://itunes.apple.com/jp/app/keynote/id361285480?mt=8"
	"tracking": [
	  "test",
	  "test2"
	],
    "creative_id": [
      1002,
      1003
    ],
	"note": "test message",
    "created_at": "2015-03-05T05:00:19.000Z",
    "updated_at": "2015-03-05T05:20:19.000Z"
  }
}
```

--

### <a id="fetch_creatives"></a>4.6. クリエイティブ一覧取得

URL: https://adcolony.glossom.jp/api/v1/creatives

#### リクエストパラメータ

| 名前 | 意味 | 必須 | サンプル |
| ---- | ---- | ---- | -------- |
| id | クリエイティブID | NO | 100 |
| campaign_id | キャンペーンID | NO | 100 |
| page | ページ番号(デフォルト: 1) | NO | 10 |
| per_page | 1ページ当たりの件数(デフォルト: 20、最高: 100) | NO | 50 |

##### リクエスト例

curl -b cookie.txt https://adcolony.glossom.jp/api/v1/creatives?campaign_id=1000

#### レスポンス

クリエイティブ一覧をjson形式で返します。

jsonのsampleとデータ型に対する説明です。

```
HTTP/1.1 200
Content-Type: application/json

{
  {
    "id": 137,                                   	 					 # クリエイティブID(ユニーク) INT型
    "name": "クリエイティブ1",                   						 # クリエイティブ名 String型(255文字まで)
    "campaign_id": 1000,                                                 # キャンペーンID
    "click_url": "http://example.com/click?action=xxx",                  # クリックURL
    "postback_url": "https://postback.example.com/click?campaign_id=xx", # ポストバックURL
	"md5": "3d24dbf8256dfa6b7eecb5e9ac21f148"                            # メディアファイルのmd5
    "created_at": "2015-03-05T05:00:19.000Z",    						 # 作成日時(UTC) DateTime型
    "updated_at": "2015-03-05T05:20:19.000Z"     						 # 更新日時(UTC) DateTime型
  },
  {
    "id": 138,
    "name": "クリエイティブ2",
    "campaign_id": 1000,
    "click_url": "http://example.com/click?action=yyy",
    "postback_url": "https://postback.example.com/click?campaign_id=yy",
	"md5": "f882cea8c124ad3f2ddf7b3b71f70538"
    "created_at": "2015-03-05T05:00:19.000Z",
    "updated_at": "2015-03-05T05:20:19.000Z"
  }
}
```

--

### <a id="upload_creative"></a>4.7. クリエイティブアップロード

URL: https://adcolony.glossom.jp/api/v1/camapigns/:campaign_id/creatives

クリエイティブをアップロードすることが出来ます。

#### HTTP メソッド

POST

#### リクエストパラメータ

| 名前 | 意味 | 必須 | 仕様 |
| -------- | ---- | ---- | ---- | ---- |
| campaign_id | キャンペーンID | YES | 掲載可否審査依頼APIで返したキャンペーンIDを、URLの`:campaign_id`に指定下さい。 |
| name | ファイル名 | YES | 128文字以内 |
| platform | プラットフォーム | YES | "iOS" or "Android" |
| media_file | メディアファイル | YES | sample.mp4 |
| md5 | メディアファイルのmd5 | YES | - |
| click_url | クリックURL | YES | 512文字以内 |
| postback_url | ポストバックURL | NO | 512文字以内 |

#### レスポンス

##### ステータスコード

* リクエストに成功した場合、ステータスコード`200`を返します。
* リクエストに失敗した場合、エラーレスポンス及び発生したエラーに該当するステータスコードを返却します。  
  
##### ボディ

| 名前 | 意味 |
| -------- | ---- |
| id | クリエイティブID |

#### サンプル

##### リクエスト

事前にメディアファイルを用意しておく必要があります。

```
curl https://adcolony.glossom.jp/api/v1/campaigns/100/creatives \
-b cookie.txt \
--request POST \
--form 'media_file=@sample.mp4'
--form 'name=テスト動画' \
--form 'platform=iOS' \
--form 'md5=3d24dbf8256dfa6b7eecb5e9ac21f148' \
--form 'click_url=http://example.com/click?action=xxx' \
--form 'postback_url=https://postback.example.com/click?campaign_id=xx'
```

##### レスポンス

```
HTTP/1.1 200
Content-Type: application/json

{
  "id": 10 # クリエイティブID
}
```

--

### <a id="judge_campaign"></a>4.8. 掲載可否審査依頼

URL: https://adcolony.glossom.jp/api/v1/campaigns

#### 前提条件

キャンペーンの状態が以下の場合、依頼することが出来ます。  
下記以外の状態だった場合、エラーレスポンスを返却します。

* 掲載OK

更新後の状態遷移は、[キャンペーン状態遷移図](STATE.md) を参照ください。

#### HTTP メソッド

POST

#### リクエストパラメータ

| カテゴリ | 名前 | 意味 | 必須 | 仕様 |
| -------- | ---- | ---- | ---- | ---- |
| campaign | | キャンペーン情報 | | |
| | name | アプリ/キャンペーン名称 | YES | 128文字以内 |
| | platform | プラットフォーム | YES | "iOS" or "Android" or "Web" |
| | client | クライアント名 | YES | 128文字以内 |
| | agency | 代理店名 | NO | 128文字以内 |
| | url | APPの場合はストアURL、WEBの場合は、クリック後遷移先URL | YES | |
| contact | | 担当者情報 | | |
| | company | 御社名 | YES | 128文字以内 |
| | name | 担当者名 | YES | 128文字以内 |
| | mail | 担当者メールアドレス | YES | |
| | tel | 担当者電話番号 | YES | |

#### レスポンス

##### ステータスコード

* リクエストに成功した場合、ステータスコード`200`を返します。
* リクエストに失敗した場合、エラーレスポンス及び発生したエラーに該当するステータスコードを返却します。  
  
##### ボディ

| 名前 | 意味 |
| -------- | ---- |
| id | キャンペーンID |

#### サンプル

##### リクエスト

sample.json

```
{
  "campaign": {
    "name": "テストキャンペーン",
    "client": "テストクライアント",
    "agency": "テスト代理店",
    "platform": "iOS",
    "url": "https://itunes.apple.com/jp/app/keynote/id361285480?mt=8"
  },
  "contact": {
    "company": "テスト会社",
    "name": "山田太郎",
    "mail": "test@example.com",
    "tel": "03-1234-5678"
  }
}
```

```
curl https://adcolony.glossom.jp/api/v1/campaigns \
-b cookie.txt \
--request POST \
--header 'Content-Type: application/json' \
-d @sample.json
```

##### レスポンス

```
HTTP/1.1 200
Content-Type: application/json

{
  "id": 1 # キャンペーンID
}
```

--

### <a id="create_campaign"></a>4.9. キャンペーン登録

URL: https://adcolony.glossom.jp/api/v1/campaigns/:campaign_id

※予めクリエイティブアップロードAPIでクリエイティブIDを取得してください。

#### HTTP メソッド

POST

#### リクエストパラメータ

| 名前 | 意味 | 必須 | 仕様 | 備考 |
| ---- | ---- | ---- | ---- | -------- |
| campaign_id | キャンペーンID | YES | - | 掲載可否審査依頼APIで返したキャンペーンIDを、URLの`:campaign_id`に指定下さい。 |
| creative_id | クリエイティブID | YES | - | 事前にクリエイティブアップロードAPIで返したクリエイティブIDを指定下さい。 |
| total_budget | キャンペーン総予算(円) | NO | 600,000円以上 | 1000000 |
| daily_budget | キャンペーン日予算(円) | NO | 20,000円以上 | 100000 |
| start_date | キャンペーン開始日時 | NO | YYYY/MM/DD | 2015/04/01 |
| end_date | キャンペーン終了日時 | NO | YYYY/MM/DD | 2015/12/31 |
| bid | 単価 | YES | 単位は円、少数点x位以下は切り捨て。| 100 |
| bid_type | 単価種別 | YES | "cpi" or "cpcv" | cpi |
| tracking | トラッキングツール | NO | xxx文字以下 | 複数指定可 |
| os | 配信OS指定 | YES | "iOS" or "Android" | |
| device | 配信デバイス指定 | NO | iOS: "iPad / iPhone / iPod, Andrid: Phone / Tablet" | 複数指定可 |
| country | 配信国指定 | YES | "JP" or "US" | |
| note | 備考 | NO | 255文字以下 | 希望等あればご利用ください |

#### レスポンス

##### ステータスコード

* リクエストに成功した場合、ステータスコード`200`を返します。
* リクエストに失敗した場合、エラーレスポンス及び発生したエラーに該当するステータスコードを返却します。  

#### サンプル
  
##### リクエスト

sample.json

```
{
  "creative_id": [
    100,
    101
  ],
  "total_budget": 100000000,
  "daily_budget": 1000000,
  "start_date": "2015/09/01",
  "end_date": "2015/12/31",
  "bid": 100,
  "bid_type": "CPM",
  "tracking": [
    "test",
    "test2"
  ],
  "os": "iOS",
  "device": [
    "iPad",
    "iPhone"
  ],
  "country": "JP",
  "note": "test"
}
```

```
curl https://adcolony.glossom.jp/api/v1/campaigns/100 \
-b cookie.txt \
--request POST \
--header 'Content-Type: application/json' \
-d @sample.json
```

##### レスポンス

```
HTTP/1.1 200
```

--

### <a id="update_campaign"></a>4.10. 配信設定変更依頼

URL: https://adcolony.glossom.jp/api/v1/campaigns/:camapign_id/change

指定したキャンペーンの配信設定変更を依頼することが出来ます。  
実際に変更が反映されるまでの間にはタイムラグが存在します。

#### 前提条件

キャンペーンの状態が以下の場合、依頼することが出来ます。  
下記以外の状態だった場合、エラーレスポンスを返却します。

* 配信中
* 配信終了

更新後の状態遷移は、[キャンペーン状態遷移図](STATE.md) を参照ください。

#### HTTP メソッド

PUT

#### リクエストパラメータ

| 名前 | 意味 | 必須 | サンプル |
| ---- | ---- | ---- | -------- |
| campaign_id | キャンペーンID | NO | - | URLの`:campaign_id`に指定下さい。 |
| cpi | 目標CPI(円) | NO | - | - |
| total_budget | キャンペーン総予算(円) | NO | 600,000円以上 | 1000000 |
| daily_budget | キャンペーン日予算(円) | NO | 20,000円以上 | 100000 |
| start_date | キャンペーン開始日時 | NO | YYYY/MM/DD | 2015/04/01 |
| end_date | キャンペーン終了日時 | NO | YYYY/MM/DD | 2015/12/31 |
| bid | 単価 | NO | 単位は円、少数点x位以下は切り捨て。| 100 |
| bid_type | 単価種別 | NO | "cpi" or "cpcv" | ※cpiこれ変更可能にする？ |
| tracking | トラッキングツール | NO | xxx文字以下 | 複数指定可 |
| os | 配信OS指定 | NO | "iOS" or "Android" | |
| device | 配信デバイス指定 | NO | iOS: "iPad / iPhone / iPod, Andrid: Phone / Tablet" | 複数指定可 |
| country | 配信国指定 | NO | "JP" or "US" | |
| note | 備考 | NO | 255文字以下 | 希望等あればご利用ください |

下記の値で登録情報を上書きするため、変更情報としては差分ではなく、変更後の値をリクエストして下さい。  
なお、変更しないデータについてはリクエストパラメータに含める必要はありません。

##### ステータスコード

* リクエストに成功した場合、ステータスコード`200`を返します。
* リクエストに失敗した場合、エラーレスポンス及び発生したエラーに該当するステータスコードを返却します。
* 前提条件を満たさなかった場合、ステータスコード`409`を返します。

##### ボディ

なし

#### サンプル

##### リクエスト例

sample.json

```
{
  "campaign": {
    "end_date": "2015/12/31",
    "total_budget": 1000000000
  }
}
```

```
curl https://adcolony.glossom.jp/api/v1/campaigns/100/change \
-b cookie.txt \
--request PUT \
--header 'Content-Type: application/json' \
-d @sample.json
```

##### レスポンス

```
HTTP/1.1 200
```

--

### <a id="cancel_campaign"></a>4.11. 配信設定変更取消

URL: https://adcolony.glossom.jp/api/v1/campaigns/{camapign_id}/cancel

キャンペーンの変更依頼を取り消すことが出来ます。  

取消後の状態遷移は、[キャンペーン状態遷移図](STATE.md) を参照ください。

#### 前提条件

キャンペーンの状態が以下の場合、依頼することが出来ます。  
下記以外の状態だった場合、エラーレスポンスを返却します。

* 設定変更中
* 設定再利用中

取消後の状態遷移は、[キャンペーン状態遷移図](STATE.md) を参照ください。

#### HTTP メソッド

POST

#### リクエストパラメータ

| 名前 | 意味 | 必須 | サンプル |
| ---- | ---- | ---- | -------- |
| campaign_id | キャンペーンID | NO | - | URLの`:campaign_id`に指定下さい。 |

##### ステータスコード

* リクエストに成功した場合、ステータスコード`200`を返します。
* リクエストに失敗した場合、エラーレスポンス及び発生したエラーに該当するステータスコードを返却します。
* 前提条件を満たさなかった場合、ステータスコード`409`を返します。

##### ボディ

なし

#### サンプル

##### リクエスト例

```
curl https://adcolony.glossom.jp/api/v1/campaigns/100/cancel \
-b cookie.txt \
--request POST
```

##### レスポンス

```
HTTP/1.1 200
```
