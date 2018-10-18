# REST API 簡介

API實現程序化交易。

通過API可以實現以下功能：

- 市場行情信息查詢（K線、深度、實時成交、24小時行情）
- 賬戶資產信息查詢
- 下單、撤單操作 
- 訂單信息查詢


# 安全認證
目前關於apikey申請和修改，請在“賬戶 - API管理”頁面進行相關操作。其中AccessKey為API 訪問密鑰，SecretKey為用戶對請求進行簽名的密鑰（僅申請時可見）。

**重要提示：這兩個密鑰與賬號安全緊密相關，無論何時都請勿向其它人透露。**

## 合法請求結構
基於安全考慮，除行情API 外的 API 請求都必須進行簽名運算。壹個合法的請求由以下幾部分組成：

* 方法請求地址 即訪問服務器地址：HOST後面跟上方法名，比如{HOST}/v1/order/orders。

* API 訪問密鑰（AccessKeyId） 您申請的 APIKEY 中的AccessKey。

* 簽名方法（SignatureMethod） 用戶計算簽名的基於哈希的協議，此處使用 HmacSHA256。

* 簽名版本（SignatureVersion） 簽名協議的版本，此處使用2。

* 時間戳（Timestamp） 您發出請求的時間 (UTC 時區) (UTC 時區) (UTC 時區) 。在查詢請求中包含此值有助於防止第三方截取您的請求。如：2017-05-11T16:22:06。再次強調是 (UTC 時區) 。

* 必選和可選參數 每個方法都有壹組用於定義 API 調用的必需參數和可選參數。可以在每個方法的說明中查看這些參數及其含義。 請壹定註意：對於GET請求，每個方法自帶的參數都需要進行簽名運算； 對於POST請求，每個方法自帶的參數不進行簽名認證，即POST請求中需要進行簽名運算的只有AccessKeyId、SignatureMethod、SignatureVersion、Timestamp四個參數，其它參數放在body中。

* 簽名 簽名計算得出的值，用於確保簽名有效和未被篡改。

例：
```
https://{host}/v1/order/orders?
AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx
&SignatureMethod=HmacSHA256
&SignatureVersion=2
&Timestamp=2017-05-11T15%3A19%3A30
&order-id=1234567890
&Signature=calculated value
```
# 簽名運算
API 請求在通過 Internet 發送的過程中極有可能被篡改。為了確保請求未被更改，我們會要求用戶在每個請求中帶上簽名（行情 API 除外），來校驗參數或參數值在傳輸途中是否發生了更改。

### 計算簽名所需的步驟：

1. 規範要計算簽名的請求
因為使用 HMAC 進行簽名計算時，使用不同內容計算得到的結果會完全不同。所以在進行簽名計算前，請先對請求進行規範化處理。下面以查詢某訂單詳情請求為例進行說明

```
https://{HOST}/v1/order/orders?
AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx
&SignatureMethod=HmacSHA256
&SignatureVersion=2
&Timestamp=2017-05-11T15:19:30
&order-id=1234567890
```

2. 請求方法（GET 或 POST），後面添加換行符\n。
```
GET\n
```
3. 添加小寫的訪問地址，後面添加換行符\n。
```
{host}\n
```
4. 訪問方法的路徑，後面添加換行符\n。
```
/v1/order/orders\n
```
5. 按照**ASCII碼的順序對參數名進行排序(使用 UTF-8 編碼，且進行了 URI 編碼，十六進制字符必須大寫**，如‘:’會被編碼為'%3A'，空格被編碼為'%20')。
例如，下面是請求參數的原始順序，進行過編碼後。
```
AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx
order-id=1234567890
SignatureMethod=HmacSHA256
SignatureVersion=2
Timestamp=2017-05-11T15%3A19%3A30
```

這些參數會被排序為：
```
AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx
SignatureMethod=HmacSHA256
SignatureVersion=2
Timestamp=2017-05-11T15%3A19%3A30
order-id=1234567890

```
按照以上順序，將各參數使用字符’&’連接。
```
AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx&SignatureMethod=HmacSHA256&SignatureVersion=2&Timestamp=2017-05-11T15%3A19%3A30&order-id=1234567890
````
組成最終的要進行簽名計算的字符串如下：
```
GET\n
{host}\n
/v1/order/orders\n
AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx&SignatureMethod=HmacSHA256&SignatureVersion=2&Timestamp=2017-05-11T15%3A19%3A30&order-id=1234567890
````
計算簽名，將以下兩個參數傳入加密哈希函數：
要進行簽名計算的字符串
```
GET\n
{host}\n
/v1/order/orders\n
AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx&SignatureMethod=HmacSHA256&SignatureVersion=2&Timestamp=2017-05-11T15%3A19%3A30&order-id=1234567890
```
進行簽名的密鑰（SecretKey）
```
b0xxxxxx-c6xxxxxx-94xxxxxx-dxxxx
```
得到簽名計算結果並進行 Base64編碼
```
4F65x5A2bLyMWVQj3Aqp+B4w+ivaA7n5Oi2SuYtCJ9o=
```
將上述值作為參數Signature的取值添加到 API 請求中。 將此參數添加到請求時，必須將該值進行 URI 編碼。

最終，發送到服務器的 API 請求應該為：
```
https://{host}/v1/order/orders?AccessKeyId=e2xxxxxx-99xxxxxx-84xxxxxx-7xxxx&order-id=1234567890&SignatureMethod=HmacSHA256&SignatureVersion=2&Timestamp=2017-05-11T15%3A19%3A30&Signature=4F65x5A2bLyMWVQj3Aqp%2BB4w%2BivaA7n5Oi2SuYtCJ9o%3D
```

#  請求說明

1. 訪問地址：將文檔中的{HOST}替換為服務商的host
2. POST請求頭信息中必須聲明 Content-Type:application/json;GET請求頭信息中必須聲明 Content-Type:application/x-www-form-urlencoded。(漢語用戶建議設置 Accept-Language:zh-cn)
3. 所有請求參數請按照 API 說明進行參數封裝。
4. 將封裝好參數的 API 請求通過 POST 或 GET 的方式提交到服務器。
5. 服務端處理請求，並返回相應的 JSON 格式結果。
6. 請使用 https 請求。
7. 限制頻率（每個接口，只針對交易api，行情api不限制）為10秒100次。
8. 查詢資產詳情方法調用順序：查詢當前用戶的所有賬戶->查詢指定賬戶的余額 



# API Reference

``` 請務必在header中設置user agent為 'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/39.0.2171.71 Safari/537.36' ```

``` symbol 規則： 基礎幣種+計價幣種。如BTC/USDT，symbol為btcusdt；ETH/BTC， symbol為ethbtc。以此類推```

## 接口列表
| 接口數據類型 | 請求方法 | 類型     | 描述  | 需要驗簽  |子賬號可用|
| ------------ | ----- | ------ | ----- | ----- | ----|
| 市場行情       | [GET /market/history/kline](#get-markethistorykline-%E8%8E%B7%E5%8F%96k%E7%BA%BF%E6%95%B0%E6%8D%AE)  | GET | K線  | N |Y|
| 市場行情       | [GET /market/detail/merged](#get-marketdetailmerged-%E8%8E%B7%E5%8F%96%E8%81%9A%E5%90%88%E8%A1%8C%E6%83%85ticker)  | GET | 滾動24小時交易和最優報價聚合行情(單個symbol)  | N |Y|
| 市場行情       | [GET /market/tickers](#get-markettickers)  | GET | 全部symbol的交易行情 | N |Y|
| 市場行情       | [GET /market/depth](#get-marketdepth-獲取-market-depth-數據)  | GET | 市場深度行情（單個symbol） | N |Y|
| 市場行情       | [GET /market/trade](#get-markettrade-獲取-trade-detail-數據)  | GET | 單個symbol最新成交記錄 | N |Y|
| 市場行情       | [GET /market/history/trade](#get-markethistorytrade-批量獲取最近的交易記錄)  | GET | 單個symbol批量成交記錄 | N |Y|
| 交易品種信息       | [GET /v1/common/symbols](#get-v1commonsymbols-查詢支持的所有交易對及精度)  | GET | 交易品種的計價貨幣和報價精度  |N |Y|
| 交易品種信息       | [GET /v1/common/currencys](#get-v1commoncurrencys-查詢支持的所有幣種)  | GET | 交易幣種列表  |N |Y|
| 系統信息       | [GET /v1/common/timestamp](#get-v1commontimestamp-查詢系統當前時間)  | GET | 查詢當前系統時間  |N |Y|
|賬戶信息	|[GET /v1/account/accounts](#get-v1accountaccounts)|	GET| 查詢用戶的所有賬戶狀態| Y|Y|
|賬戶信息	|[GET /v1/account/accounts/{account-id}/balance](#get-v1accountaccountsaccount-idbalance-查詢指定賬戶的余額)|GET|	查詢指定賬戶余額	|Y|Y|
|交易	|[POST/v1/order/orders/place](#post-v1orderordersplace-下單)|POST|	下單	|Y|Y|
|交易	|[POST/v1/order/orders/{order-id}/submitcancel](#post-v1orderordersorder-idsubmitcancel--申請撤銷壹個訂單請求)|POST|	按order-id撤銷壹個訂單	|Y|Y|
|交易	|[POST /v1/order/orders/batchcancel](#post-v1orderordersbatchcancel-批量撤銷訂單)|POST|	按order_id, 批量撤銷訂單（up to 50)	|Y|Y|
|交易	|[POST /v1/order/orders/batchCancelOpenOrders](#post-v1orderbatchcancelopenorders-批量取消符合條件的訂單)|POST|	按訂單條件批量撤銷訂單（up to 100)	|Y|Y|
|用戶訂單信息	|[GET /v1/order/orders/{order-id}](#get-v1orderordersorder-id-查詢某個訂單詳情)|GET|根據order-id查詢訂單詳情|Y|Y|
|用戶訂單信息	|[GET /v1/order/orders/{order-id}/matchresults](#get-v1orderordersorder-idmatchresults--查詢某個訂單的成交明細)	|GET| 根據order-id查詢訂單的成交明細	|Y|Y|
|用戶訂單信息	|[GET /v1/order/orders](#get-v1orderorders-查詢當前委托歷史委托)	|GET|查詢用戶當前委托、或歷史委托訂單 (up to 100)	|Y|Y|
|用戶訂單信息	|[GET /v1/order/matchresults](#get-v1ordermatchresults-查詢當前成交歷史成交)	|GET|查詢用戶當前成交、歷史成交|Y|Y|
|用戶訂單信息	|[GET /v1/order/openOrders](#get-v1orderopenorders-獲取所有當前帳號下未成交訂單)	|GET|查詢用戶當前未成交訂單 (up to 500)|Y|	Y|
|充提幣	|[POST /v1/dw/withdraw/api/create](#post-v1dwwithdrawapicreate-申請提現虛擬幣)	|POST|申請提幣|	Y|N|
|充提幣	|[POST /v1/dw/withdraw-virtual/{withdraw-id}/cancel](#post-v1dwwithdraw-virtualwithdraw-idcancel-申請取消提現虛擬幣)|POST| 	撤銷提幣申請|Y|N|
|充提幣	|[GET /v1/query/deposit-withdraw](#get-v1querydeposit-withdraw-查詢虛擬幣充提記錄)	|GET|查詢充提記錄|Y|N|


## 市場行情

在調用行情接口時，請添加get參數，key為AccessKeyId ，value為網頁上申請的apikey的accesskey 。

例：

```
https://{HOST}/market/history/kline?period=1day&size=200&symbol=btcusdt&AccessKeyId=fff-xxx-ssss-kkk

```

#### GET /market/history/kline 獲取K線數據

請求參數: 

| 參數名稱 | 是否必須  | 類型     | 描述  | 默認值   | 取值範圍  |
| ------------ | ----- | ------ | ----- | ----- | ------- |
| symbol       | true  | string | 交易對  |  | btcusdt, bchbtc, rcneth ...   |
| period       | true  | string | K線類型 |    | 1min, 5min, 15min, 30min, 60min, 1day, 1mon, 1week, 1year |
| size | false | integer | 獲取數量 | 150 | [1,2000] |

響應數據: 

| 參數名稱   | 是否必須 | 數據類型   | 描述   | 取值範圍   |
| ------ | ---- | ------ | ----------- | ------ |
| status | true | string | 請求處理結果    | "ok" , "error" |
| ts     | true | number | 響應生成時間點，單位：毫秒  |    |
| tick   | true | object | KLine 數據   |      |
| ch     | true | string | 數據所屬的 channel，格式： market.$symbol.kline.$period |    |

data 說明: 

```
  "data": [
{
    "id": K線id,
    "amount": 成交量,
    "count": 成交筆數,
    "open": 開盤價,
    "close": 收盤價,當K線為最晚的壹根時，是最新成交價
    "low": 最低價,
    "high": 最高價,
    "vol": 成交額, 即 sum(每壹筆成交價 * 該筆的成交量)
  }
]
```

請求響應示例: 

```
/* GET /market/history/kline?period=1day&size=200&symbol=btcusdt */
{
  "status": "ok",
  "ch": "market.btcusdt.kline.1day",
  "ts": 1499223904680,
  "data": [
{
    "id": 1499184000,
    "amount": 37593.0266,
    "count": 0,
    "open": 1935.2000,
    "close": 1879.0000,
    "low": 1856.0000,
    "high": 1940.0000,
    "vol": 71031537.97866500
  },
// more data here
]
}

/* GET /market/history/kline?period=not-exist&size=200&symbol=ethusdt */
{
  "ts": 1490758171271,
  "status": "error",
  "err-code": "invalid-parameter",
  "err-msg": "invalid period"
}

/* GET /market/history/kline?period=1day&size=not-exist&symbol=ethusdt */
{
  "ts": 1490758221221,
  "status": "error",
  "err-code": "bad-request",
  "err-msg": "invalid size, valid range: [1,2000]"
}

/* GET /market/history/kline?period=1day&size=200&symbol=not-exist */
{
  "ts": 1490758171271,
  "status": "error",
  "err-code": "invalid-parameter",
  "err-msg": "invalid symbol"
}
```

#### GET /market/detail/merged 獲取聚合行情(Ticker)

請求參數: 

| 參數名稱   | 是否必須  | 類型     | 描述  | 默認值   | 取值範圍   |
| ------------ | ----- | ------ | -----  | ---  |  ----------- |
| symbol    | true  | string | 交易對   |   | btcusdt, bchbtc, rcneth ...|

響應數據: 

| 參數名稱   | 是否必須 | 數據類型   | 描述   | 取值範圍   |
| ------ | ---- | ------ | -------  | ----  |
| status | true | string | 請求處理結果  | "ok" , "error" |
| ts     | true | number | 響應生成時間點，單位：毫秒    |     |
| tick   | true | object | K線數據    |      |
| ch     | true | string | 數據所屬的 channel，格式： market.$symbol.detail.merged |     |

tick 說明: 

```
  "tick": {
    "id": K線id,
    "amount": 成交量,
    "count": 成交筆數,
    "open": 開盤價,
    "close": 收盤價,當K線為最晚的壹根時，是最新成交價
    "low": 最低價,
    "high": 最高價,
    "vol": 成交額, 即 sum(每壹筆成交價 * 該筆的成交量)
    "bid": [買1價,買1量],
    "ask": [賣1價,賣1量]
  }

```

請求響應示例: 

```
/* GET /market/detail/merged?symbol=ethusdt */
{
"status":"ok",
"ch":"market.ethusdt.detail.merged",
"ts":1499225276950,
"tick":{
  "id":1499225271,
  "ts":1499225271000,
  "close":1885.0000,
  "open":1960.0000,
  "high":1985.0000,
  "low":1856.0000,
  "amount":81486.2926,
  "count":42122,
  "vol":157052744.85708200,
  "ask":[1885.0000,21.8804],
  "bid":[1884.0000,1.6702]
  }
}

/* GET /market/detail/merged?symbol=not-exist */
{
  "ts": 1490758171271,
  "status": "error",
  "err-code": "invalid-parameter",
  "err-msg": "invalid symbol”
}

```
#### GET /market/tickers 
```json
{  
    "status":"ok",
    "ts":1510885463001,
    "data":[  
        {  
            "open":0.044297,      // 日K線 開盤價
            "close":0.042178,     // 日K線 收盤價
            "low":0.040110,       // 日K線 最低價
            "high":0.045255,      // 日K線 最高價
            "amount":12880.8510,  // 24小時成交量
            "count":12838,        // 24小時成交筆數
            "vol":563.0388715740, // 24小時成交額
            "symbol":"ethbtc"     // 交易對
        },
        {  
            "open":0.008545,
            "close":0.008656,
            "low":0.008088,
            "high":0.009388,
            "amount":88056.1860,
            "count":16077,
            "vol":771.7975953754,
            "symbol":"ltcbtc"
        }
    ]
}
```
註：當交易對尚未產生成交時，返回的數據裏面 `open` `close` `high` `low` `amount` `count` `vol` 的值都為 `null`
#### GET /market/depth 獲取 Market Depth 數據

請求參數:

| 參數名稱   | 是否必須  | 類型     | 描述    | 默認值   | 取值範圍   |
| ------  | ----- | ------ | ------  | ----- | -------  |
| symbol     | true  | string | 交易對    |       | btcusdt, bchbtc, rcneth ... |
| type    | true  | string | Depth 類型     |       | step0, step1, step2, step3, step4, step5（合並深度0-5）；step0時，不合並深度 |

* 用戶選擇“合並深度”時，壹定報價精度內的市場掛單將予以合並顯示。合並深度僅改變顯示方式，不改變實際成交價格。

響應數據:

| 參數名稱   | 是否必須 | 數據類型   | 描述    | 取值範圍    |
| ------ | ---- | ------ | -------  | ---  |
| status | true | string |       | "ok" 或者 "error" |
| ts     | true | number | 響應生成時間點，單位：毫秒    |     |
| tick   | true | object | Depth 數據    |     |
| ch     | true | string | 數據所屬的 channel，格式： market.$symbol.depth.$type |  |

tick 說明:

```
  "tick": {
    "id": 消息id,
    "ts": 消息生成時間，單位：毫秒,
    "bids": 買盤,[price(成交價), amount(成交量)], 按price降序,
    "asks": 賣盤,[price(成交價), amount(成交量)], 按price升序
  }
```

請求響應示例:

```json
/* GET /market/depth?symbol=ethusdt&type=step1 */
{
  "status": "ok",
  "ch": "market.btcusdt.depth.step1",
  "ts": 1489472598812,
  "tick": {
    "id": 1489464585407,
    "ts": 1489464585407,
    "bids": [
      [7964, 0.0678], // [price, amount]
      [7963, 0.9162],
      [7961, 0.1],
      [7960, 12.8898],
      [7958, 1.2],
      [7955, 2.1009],
      [7954, 0.4708],
      [7953, 0.0564],
      [7951, 2.8031],
      [7950, 13.7785],
      [7949, 0.125],
      [7948, 4],
      [7942, 0.4337],
      [7940, 6.1612],
      [7936, 0.02],
      [7935, 1.3575],
      [7933, 2.002],
      [7932, 1.3449],
      [7930, 10.2974],
      [7929, 3.2226]
    ],
    "asks": [
      [7979, 0.0736],
      [7980, 1.0292],
      [7981, 5.5652],
      [7986, 0.2416],
      [7990, 1.9970],
      [7995, 0.88],
      [7996, 0.0212],
      [8000, 9.2609],
      [8002, 0.02],
      [8008, 1],
      [8010, 0.8735],
      [8011, 2.36],
      [8012, 0.02],
      [8014, 0.1067],
      [8015, 12.9118],
      [8016, 2.5206],
      [8017, 0.0166],
      [8018, 1.3218],
      [8019, 0.01],
      [8020, 13.6584]
    ]
  }
}

/* GET /market/depth?symbol=ethusdt&type=not-exist */
{
  "ts": 1490759358099,
  "status": "error",
  "err-code": "invalid-parameter",
  "err-msg": "invalid type"
}
```


#### GET /market/trade 獲取 Trade Detail 數據

請求參數:

| 參數名稱   | 是否必須  | 類型  | 描述   | 默認值   | 取值範圍  |
| -------  | ----- | ------ | ------ | ----- | ---- |
| symbol   | true  | string | 交易對   |   | btcusdt, bchbtc, rcneth ... |

響應數據:

| 參數名稱   | 是否必須 | 數據類型   | 描述   | 取值範圍    |
| ------ | ---- | ------ | ----------| --------------- |
| status | true | string |    | "ok" 或者 "error" |
| ts     | true | number | 響應生成時間點，單位：毫秒    |      |
| tick   | true | object | Trade 數據      |     |
| ch     | true | string | 數據所屬的 channel，格式： market.$symbol.trade.detail |     |

tick 說明：

```
  "tick": {
    "id": 消息id,
    "ts": 最新成交時間,
    "data": [
      {
        "id": 成交id,
        "price": 成交價錢,
        "amount": 成交量,
        "direction": 主動成交方向,
        "ts": 成交時間
      }
    ]
  }
```

請求響應例子:

```json
/* GET /market/trade?symbol=ethusdt */
{
  "status": "ok",
  "ch": "market.btcusdt.trade.detail",
  "ts": 1489473346905,
  "tick": {
    "id": 600848670,
    "ts": 1489464451000,
    "data": [
      {
        "id": 600848670,
        "price": 7962.62,
        "amount": 0.0122,
        "direction": "buy",
        "ts": 1489464451000
      }
    ]
  }
}

/* GET /market/trade?symbol=not-exist */
{
  "ts": 1490759506429,
  "status": "error",
  "err-code": "invalid-parameter",
  "err-msg": "invalid symbol"
}
```
#### GET /market/history/trade 批量獲取最近的交易記錄

請求參數:

| 參數名稱   | 是否必須  | 類型   | 描述   | 默認值   | 取值範圍    |
| ------- | ----- | ------ | ---- | ----- | ---  |
| symbol   | true  | string | 交易對   |       | btcusdt, bchbtc, rcneth ... |
| size  | false  |integer| 獲取交易記錄的數量    |  1   | [1, 2000]    |

響應數據:

| 參數名稱   | 是否必須  | 類型  | 描述  | 默認值   | 取值範圍   |
| -------- | ----- | ------ | --------  | ----- | ----  |
| status   | true  | string |    |    | ok, error   |
| ch     | true | string | 數據所屬的 channel，格式： market.$symbol.trade.detail  |    |
| ts    | true  |integer| 發送時間  |    |      |
| data  | true  | object | 成交記錄    |    |    |

data 說明：

```
  "data": {
    "id": 消息id,
    "ts": 最新成交時間,
    "data": [
      {
        "id": 成交id,
        "price": 成交價,
        "amount": 成交量,
        "direction": 主動成交方向,
        "ts": 成交時間
      }
    ]
  }
```

請求響應例子:

```
/* GET /market/history/trade?symbol=ethusdt */
{
    "status": "ok",
    "ch": "market.ethusdt.trade.detail",
    "ts": 1502448925216,
    "data": [
        {
            "id": 31459998,
            "ts": 1502448920106,
            "data": [
                {
                    "id": 17592256642623,
                    "amount": 0.04,
                    "price": 1997,
                    "direction": "buy",
                    "ts": 1502448920106
                }
            ]
        }
    ]
}
```

### 公共API


####  GET /v1/common/symbols 查詢支持的所有交易對及精度

 請求參數:
(無)

響應數據:

| 參數名稱    | 是否必須 | 數據類型   | 描述    | 取值範圍 |
| -------------- | ---- | ------ | ----- | ---- |
| base-currency  | true | string | 基礎幣種  |      |
| quote-currency | true | string | 計價幣種  |      |
| price-precision | true | string | 價格精度位數（0為個位） |      |
| amount-precision | true | string | 數量精度位數（0為個位) |      |
| symbol-partition | true | string | 交易區 | main主區，innovation創新區，bifurcation分叉區  |

 請求響應例子:

```
/* GET /v1/common/symbols */
{
  "status": "ok",
  "data": [
    {
      "base-currency": "eth",
      "quote-currency": "usdt",
      "symbol": "ethusdt"
    }
    {
      "base-currency": "etc",
      "quote-currency": "usdt",
      "symbol": "etcusdt"
    }
  ]
}
```

####  GET /v1/common/currencys 查詢支持的所有幣種

 請求參數:

(無)

 響應數據:

```
currency list
```

請求響應例子:

```json
/* GET /v1/common/currencys */
{
  "status": "ok",
  "data": [
    "usdt",
    "eth",
    "etc"
  ]
}
```

####  GET /v1/common/timestamp 查詢系統當前時間

請求參數:

(無)

 響應數據:

```
系統時間戳
```

請求響應例子

```
/* GET /v1/common/timestamp */
{
  "status": "ok",
  "data": 1494900087029
}
```

### 用戶資產API

####  GET /v1/account/accounts 

請求參數:

無

響應數據:

| 參數名稱  | 是否必須 | 數據類型 | 描述 | 取值範圍 |
| ----- | ---- | ------ | ----- | ----  |
| id    | true | long   | account-id |    |
| state | true | string | 賬戶狀態  | working：正常, lock：賬戶被鎖定 |
| type  | true | string | 賬戶類型  | spot：現貨賬戶    |

請求響應例子:

```json
/* GET /v1/account/accounts */
{
  "status": "ok",
  "data": [
    {
      "id": 100009,
      "type": "spot",
      "state": "working",
      "user-id": 1000
    }
  ]
}
```

####  GET /v1/account/accounts/{account-id}/balance 查詢指定賬戶的余額

請求參數

| 參數名稱   | 是否必須 | 類型   | 描述   | 默認值  | 取值範圍 |
| ---------- | ---- | ------ | --------------- | ---- | ---- |
| account-id | true | string | account-id，填在 path 中，可用 GET /v1/account/accounts 獲取 |      |      |

* 如果不知道自己的賬戶ID，請使用 ```GET /v1/account/accounts``` 查詢

響應數據:

| 參數名稱  | 是否必須  | 數據類型   | 描述    | 取值範圍   |
| ----- | ----- | ------ | ----- | ----- |
| id    | true  | long   | 賬戶 ID |      |
| state | true  | string | 賬戶狀態  | working：正常  lock：賬戶被鎖定 |
| type  | true  | string | 賬戶類型  | spot：現貨賬戶              |
| list  | false | Array  | 子賬戶數組 |     |

list字段說明

| 參數名稱   | 是否必須 | 數據類型   | 描述   | 取值範圍    |
| -------- | ---- | ------ | ---- |  ------ |
| balance  | true | string | 余額   |    |
| currency | true | string | 幣種   |    |
| type     | true | string | 類型   | trade: 交易余額，frozen: 凍結余額 |

請求響應例子:

```json
/* GET /v1/account/accounts/'account-id'/balance */
{
  "status": "ok",
  "data": {
    "id": 100009,
    "type": "spot",
    "state": "working",
    "list": [
      {
        "currency": "usdt",
        "type": "trade",
        "balance": "500009195917.4362872650"
      },
      {
        "currency": "usdt",
        "type": "frozen",
        "balance": "328048.1199920000"
      },
     {
        "currency": "etc",
        "type": "trade",
        "balance": "499999894616.1302471000"
      },
      {
        "currency": "etc",
        "type": "frozen",
        "balance": "9786.6783000000"
      }
     {
        "currency": "eth",
        "type": "trade",
        "balance": "499999894616.1302471000"
      },
      {
        "currency": "eth",
        "type": "frozen",
        "balance": "9786.6783000000"
      }
    ],
    "user-id": 1000
  }
}
```

## 交易API

#### POST /v1/order/orders/place 下單

#### 請求參數

| 參數名稱   | 是否必須  | 類型     | 描述    | 默認值  | 取值範圍    |
| ----- | ----- | ------ | --------  | ---- | -------  |
| account-id | true  | string | 賬戶 ID，使用accounts方法獲得。幣幣交易使用‘spot’賬戶的accountid |      |     |
| amount     | true  | string | 限價單表示下單數量，市價買單時表示買多少錢，市價賣單時表示賣多少幣 |   |   |
| price      | false | string | 下單價格，市價單不傳該參數   |      |       |
| source     | false | string | 訂單來源    | api |    |
| symbol     | true  | string | 交易對    |      | btcusdt, bchbtc, rcneth ...   |
| type       | true  | string | 訂單類型    |    | buy-market：市價買, sell-market：市價賣, buy-limit：限價買, sell-limit：限價賣, buy-ioc：IOC買單, sell-ioc：IOC賣單, buy-limit-maker, sell-limit-maker(詳細說明見下)|

**buy-limit-maker**

當“下單價格”>=“市場最低賣出價”，訂單提交後，系統將拒絕接受此訂單；

當“下單價格”<“市場最低賣出價”，提交成功後，此訂單將被系統接受。

**sell-limit-maker**

當“下單價格”<=“市場最高買入價”，訂單提交後，系統將拒絕接受此訂單；

當“下單價格”>“市場最高買入價”，提交成功後，此訂單將被系統接受。


#### 響應數據:

| 參數名稱 | 是否必須 | 數據類型 | 描述   | 取值範圍 |
| ---- | ---- | ---- | ---- | ---- |
| data | false | string | 訂單ID  |      |

#### 請求響應例子:

```json
/* POST /v1/order/orders/place */
{
   "account-id": "100009",
   "amount": "10.1",
   "price": "100.1",
   "source": "api",
   "symbol": "ethusdt",
   "type": "buy-limit"
}
{
  "status": "ok",
  "data": "59378"
}
```

####  GET /v1/order/openOrders 獲取所有當前帳號下未成交訂單

####  請求參數: 

`“account-id” 和 “symbol” 需同時指定或者二者都不指定。如果二者都不指定，返回最多500條尚未成交訂單，按訂單號降序排列。`

| 參數名稱     | 是否必須 | 類型     | 描述           | 默認值  | 取值範圍 |
| -------- | ---- | ------ | ------------ | ---- | ---- |
| account-id | true | string | 賬號ID |      |      |
| symbol | true | string | 交易對 |      |   單個交易對字符串，缺省將返回所有符合條件尚未成交訂單   |
| side | false | string | 主動交易方向 |      |   “buy”或者“sell”，缺省將返回所有符合條件尚未成交訂單   |
| size | false | int | 所需返回記錄數 |   10   |    [0,500]  |

####  響應數據: 

| 參數名稱 | 是否必須 | 數據類型   | 描述    | 取值範圍 |
| ---- | ---- | ------ | ----- | ---- |
| id | true | long | 訂單號 |  |
| symbol| true | string | 交易對 |     |
| price | true | string | 下單價格 |    |
| created-at | true | int | 下單時間（毫秒） |  Unix時間戳  |
| type | true | string | 訂單類型 |  buy-market, sell-market, buy-limit, sell-limit, buy-ioc, sell-ioc  |
| filled-amount | true | string | 下單時間（毫秒） |  對於非“部分成交”訂單，此字段為 0  |
| filled-cash-amount | true | string | 已成交部分的訂單價格(=已成交單量x下單價格) |  對於非“部分成交”訂單，此字段為 0  |
| filled-fees | true | string | 已成交部分所收取手續費 |  對於非“部分成交”訂單，此字段為 0  |
| source | true | string | 訂單來源 |  sys, web, api, app  |
| state | true | string | 此訂單狀態 |  submitted（已提交）, partial-filled（部分成交）, cancelling（正在取消）  |

####  響應示例:

```json
/* GET /v1/orders/openOrders */
{
  "status": "ok",
  "data": [
    {
      "id": 5454937,
      "symbol": "ethusdt",
      "account-id": 30925,
      "amount": "1.000000000000000000",
      "price": "0.453000000000000000",
      "created-at": 1530604762277,
      "type": "sell-limit",
      "filled-amount": "0.0",
      "filled-cash-amount": "0.0",
      "filled-fees": "0.0",
      "source": "web",
      "state": "submitted"
    }
  ]
}
```

####  POST /v1/order/orders/{order-id}/submitcancel  申請撤銷壹個訂單請求

請求參數: 

| 參數名稱     | 是否必須 | 類型     | 描述           | 默認值  | 取值範圍 |
| -------- | ---- | ------ | ------------ | ---- | ---- |
| order-id | true | string | 訂單ID，填在path中 |      |      |

響應數據: 

| 參數名稱 | 是否必須 | 數據類型   | 描述    | 取值範圍 |
| ---- | ---- | ------ | ----- | ---- |
| data | true | string | 訂單 ID |      |

請求響應例子:

```
/* POST /v1/order/orders/{order-id}/submitcancel */
{
  "status": "ok",//註意，返回OK表示撤單請求成功。訂單是否撤銷成功請調用訂單查詢接口查詢該訂單狀態
  "data": "59378"
}
```

#### POST /v1/order/orders/batchcancel 批量撤銷訂單

請求參數:

| 參數名稱  | 是否必須 | 類型   | 描述   | 默認值  | 取值範圍 |
| ---- | ---- | ---- | ----  | ---- | ---- |
| order-ids | true | list | 撤銷訂單ID列表 |  |單次不超過50個訂單id|

響應數據:

| 參數名稱 | 是否必須  | 數據類型 | 描述    | 取值範圍 |
| ---- | ----- | ---- | ----- | ---- |
| data | false | map | 撤單結果 |      |

請求響應例子:

```json
/* POST /v1/order/orders/batchcancel */
{
  "order-ids": [
    "1", "2", "3"
  ]
}
```
---
```json
{
  "status": "ok",
  "data": {
    "success": [
      "1",
      "3"
    ],
    "failed": [
      {
        "err-msg": "記錄無效",
        "order-id": "2",
        "err-code": "base-record-invalid"
      }
    ]
  }
}
```

####  POST  /v1/order/orders/batchCancelOpenOrders  批量取消符合條件的訂單

請求參數: 

| 參數名稱     | 是否必須 | 類型     | 描述           | 默認值  | 取值範圍 |
| -------- | ---- | ------ | ------------ | ---- | ---- |
| account-id | true  | string | 賬戶ID     |     |      |
| symbol     | false | string | 交易對     |      |   單個交易對字符串，缺省將返回所有符合條件尚未成交訂單  |
| side | false | string | 主動交易方向 |      |   “buy”或“sell”，缺省將返回所有符合條件尚未成交訂單   |
| size | false | int | 所需返回記錄數  |  100 |   [0,100]   |

響應數據: 

| 參數名稱 | 是否必須 | 數據類型   | 描述    | 取值範圍 |
| ---- | ---- | ------ | ----- | ---- |
| success-count | true | int | 成功取消的訂單數 |     |
| failed-count | true | int | 取消失敗的訂單數 |     |
| next-id | true | long | 下壹個符合取消條件的訂單號 |    |

####  響應示例:

```json
/* POST /v1/order/orders/batchCancelOpenOrders */
{
  "status": "ok",
  "data": {
    "success-count": 2,
    "failed-count": 0,
    "next-id": 5454600
  }
}
```

####  GET /v1/order/orders/{order-id} 查詢某個訂單詳情

請求參數: 

| 參數名稱     | 是否必須 | 類型  | 描述   | 默認值  | 取值範圍 |
| -------- | ---- | ------ | -----  | ---- | ---- |
| order-id | true | string | 訂單ID，填在path中 |      |      |

響應數據: 

| 參數名稱     | 是否必須  | 數據類型   | 描述   | 取值範圍     |
| ----------------- | ----- | ------ | -------  | ----  |
| account-id        | true  | long   | 賬戶 ID    |       |
| amount            | true  | string | 訂單數量              |    |
| canceled-at       | false | long   | 訂單撤銷時間    |     |
| created-at        | true  | long   | 訂單創建時間    |   |
| field-amount      | true  | string | 已成交數量    |     |
| field-cash-amount | true  | string | 已成交總金額     |      |
| field-fees        | true  | string | 已成交手續費（買入為幣，賣出為錢） |     |
| finished-at       | false | long   | 訂單變為終結態的時間，不是成交時間，包含“已撤單”狀態    |     |
| id                | true  | long   | 訂單ID    |     |
| price             | true  | string | 訂單價格       |     |
| source            | true  | string | 訂單來源   | api |
| state             | true  | string | 訂單狀態   | submitting , submitted 已提交, partial-filled 部分成交, partial-canceled 部分成交撤銷, filled 完全成交, canceled 已撤銷 |
| symbol            | true  | string | 交易對   | btcusdt, bchbtc, rcneth ... |
| type              | true  | string | 訂單類型   | buy-market：市價買, sell-market：市價賣, buy-limit：限價買, sell-limit：限價賣, buy-ioc：IOC買單, sell-ioc：IOC賣單 |


請求響應例子:

```json
/* GET /v1/order/orders/{order-id} */
{
  "status": "ok",
  "data": {
    "id": 59378,
    "symbol": "ethusdt",
    "account-id": 100009,
    "amount": "10.1000000000",
    "price": "100.1000000000",
    "created-at": 1494901162595,
    "type": "buy-limit",
    "field-amount": "10.1000000000",
    "field-cash-amount": "1011.0100000000",
    "field-fees": "0.0202000000",
    "finished-at": 1494901400468,
    "user-id": 1000,
    "source": "api",
    "state": "filled",
    "canceled-at": 0,
    "exchange": "xxx",
    "batch": ""
  }
}
```

####  GET /v1/order/orders/{order-id}/matchresults  查詢某個訂單的成交明細

請求參數:

| 參數名稱  | 是否必須 | 類型  | 描述  | 默認值  | 取值範圍 |
| -------- | ---- | ------ | -----  | ---- | ---- |
| order-id | true | string | 訂單ID，填在path中 |      |      |

響應數據:

| 參數名稱    | 是否必須 | 數據類型   | 描述   | 取值範圍     |
| ------------- | ---- | ------ | -------- | -------- |
| created-at    | true | long   | 成交時間     |    |
| filled-amount | true | string | 成交數量     |    |
| filled-fees   | true | string | 成交手續費    |     |
| id            | true | long   | 訂單成交記錄ID |     |
| match-id      | true | long   | 撮合ID     |     |
| order-id      | true | long   | 訂單 ID    |      |
| price         | true | string | 成交價格  |    |
| source        | true | string | 訂單來源  | api      |
| symbol        | true | string | 交易對   | btcusdt, bchbtc, rcneth ...  |
| type          | true | string | 訂單類型   | buy-market：市價買, sell-market：市價賣, buy-limit：限價買, sell-limit：限價賣, buy-ioc：IOC買單, sell-ioc：IOC賣單 |

請求響應例子:

```json
/* GET /v1/order/orders/{order-id}/matchresults */
{
  "status": "ok",
  "data": [
    {
      "id": 29553,
      "order-id": 59378,
      "match-id": 59335,
      "symbol": "ethusdt",
      "type": "buy-limit",
      "source": "api",
      "price": "100.1000000000",
      "filled-amount": "9.1155000000",
      "filled-fees": "0.0182310000",
      "created-at": 1494901400435
    }
  ]
}
```

####  GET /v1/order/orders 查詢當前委托、歷史委托

請求參數:

| 參數名稱   | 是否必須  | 類型     | 描述   | 默認值  | 取值範圍   |
| ---------- | ----- | ------ | ------  | ---- | ----  |
| symbol     | true  | string | 交易對      |      |btcusdt, bchbtc, rcneth ...  |
| types      | false | string | 查詢的訂單類型組合，使用','分割  |      | buy-market：市價買, sell-market：市價賣, buy-limit：限價買, sell-limit：限價賣, buy-ioc：IOC買單, sell-ioc：IOC賣單 |
| start-date | false | string | 查詢開始日期, 日期格式yyyy-mm-dd |      |      |
| end-date   | false | string | 查詢結束日期, 日期格式yyyy-mm-dd |      |    |
| states     | true  | string | 查詢的訂單狀態組合，使用','分割  |      | submitted 已提交, partial-filled 部分成交, partial-canceled 部分成交撤銷, filled 完全成交, canceled 已撤銷 |
| from       | false | string | 查詢起始 ID   |      |    |
| direct     | false | string | 查詢方向   |      | prev 向前，next 向後    |
| size       | false | string | 查詢記錄大小      |      |         |

響應數據: 

| 參數名稱    | 是否必須  | 數據類型   | 描述   | 取值範圍   |
| ----------------- | ----- | ------ | ----------------- | ----  |
| account-id        | true  | long   | 賬戶 ID    |     |
| amount            | true  | string | 訂單數量    |   |
| canceled-at       | false | long   | 接到撤單申請的時間   |    |
| created-at        | true  | long   | 訂單創建時間   |    |
| field-amount      | true  | string | 已成交數量   |    |
| field-cash-amount | true  | string | 已成交總金額    |    |
| field-fees        | true  | string | 已成交手續費（買入為幣，賣出為錢） |       |
| finished-at       | false | long   | 最後成交時間    |   |
| id                | true  | long   | 訂單ID    |    |
| price             | true  | string | 訂單價格  |    |
| source            | true  | string | 訂單來源   | api  |
| state             | true  | string | 訂單狀態    | submitting , submitted 已提交, partial-filled 部分成交, partial-canceled 部分成交撤銷, filled 完全成交, canceled 已撤銷 |
| symbol            | true  | string | 交易對    | btcusdt, bchbtc, rcneth ... |
| type              | true  | string | 訂單類型  | submit-cancel：已提交撤單申請  ,buy-market：市價買, sell-market：市價賣, buy-limit：限價買, sell-limit：限價賣, buy-ioc：IOC買單, sell-ioc：IOC賣單 |


請求響應例子:

```json
/* GET /v1/order/orders */
{
  "status": "ok",
  "data": [
    {
      "id": 59378,
      "symbol": "ethusdt",
      "account-id": 100009,
      "amount": "10.1000000000",
      "price": "100.1000000000",
      "created-at": 1494901162595,
      "type": "buy-limit",
      "field-amount": "10.1000000000",
      "field-cash-amount": "1011.0100000000",
      "field-fees": "0.0202000000",
      "finished-at": 1494901400468,
      "user-id": 1000,
      "source": "api",
      "state": "filled",
      "canceled-at": 0,
      "exchange": "xxx",
      "batch": ""
    }
  ]
}
```

####  GET /v1/order/matchresults 查詢當前成交、歷史成交

請求參數:

| 參數名稱   | 是否必須  | 類型  | 描述   | 默認值  | 取值範圍    |
| ---------- | ----- | ------ | ------ | ---- | ----------- |
| symbol     | true  | string | 交易對   | btcusdt, bchbtc, rcneth ... |    |
| types      | false | string | 查詢的訂單類型組合，使用','分割   |      | buy-market：市價買, sell-market：市價賣, buy-limit：限價買, sell-limit：限價賣, buy-ioc：IOC買單, sell-ioc：IOC賣單 |
| start-date | false | string | 查詢開始日期, 日期格式yyyy-mm-dd | -61 days     | [-61day, now] |
| end-date   | false | string | 查詢結束日期, 日期格式yyyy-mm-dd |   Now   |  [start-date, now]  |
| from       | false | string | 查詢起始 ID    |   訂單成交記錄ID（最大值）   |     |
| direct     | false | string | 查詢方向    |   默認next， 成交記錄ID由大到小排序   | prev 向前，next 向後   |
| size       | false | string | 查詢記錄大小    |   100   | <=100  |

響應數據: 

| 參數名稱   | 是否必須 | 數據類型   | 描述   | 取值範圍   |
| ------------- | ---- | ------ | -------- | ------- |
| created-at    | true | long   | 成交時間     |    |
| filled-amount | true | string | 成交數量     |    |
| filled-fees   | true | string | 成交手續費    |    |
| id            | true | long   | 訂單成交記錄ID |    |
| match-id      | true | long   | 撮合ID     |    |
| order-id      | true | long   | 訂單 ID    |    |
| price         | true | string | 成交價格     |    |
| source        | true | string | 訂單來源     | api   |
| symbol        | true | string | 交易對      | btcusdt, bchbtc, rcneth ...  |
| type          | true | string | 訂單類型     | buy-market：市價買, sell-market：市價賣, buy-limit：限價買, sell-limit：限價賣, buy-ioc：IOC買單, sell-ioc：IOC賣單 |

請求響應例子:

```json
/* GET /v1/orders/matchresults */
{
  "status": "ok",
  "data": [
    {
      "id": 29555,
      "order-id": 59378,
      "match-id": 59335,
      "symbol": "ethusdt",
      "type": "buy-limit",
      "source": "api",
      "price": "100.1000000000",
      "filled-amount": "0.9845000000",
      "filled-fees": "0.0019690000",
      "created-at": 1494901400487
    }
  ]
}
```


## 虛擬幣提現API

> **僅支持提現到【Pro站提幣地址列表中的提幣地址】**


####  POST /v1/dw/withdraw/api/create 申請提現虛擬幣

請求參數:

| 參數名稱       | 是否必須 | 類型     | 描述     | 默認值  | 取值範圍 |
| ---------- | ---- | ------ | ------ | ---- | ---- |
| address | true | string   | 提現地址 |      |      |
| amount     | true | string | 提幣數量   |      |      |
| currency | true | string | 資產類型   |   |  btc, ltc, bch, eth, etc ...(支持的幣種) |
| fee     | false | string | 轉賬手續費  |      |      |
| addr-tag|false | string | 虛擬幣共享地址tag，適用於xrp，xem，bts，steem，eos，xmr |  | 格式, "123"類的整數字符串|

響應數據: 

| 參數名稱 | 是否必須  | 數據類型 | 描述   | 取值範圍 |
| ---- | ----- | ---- | ---- | ---- |
| data | false | long | 提現ID |      |

請求響應例子:

```
/* POST /v1/dw/withdraw/api/create*/
{
  "address": "0xde709f2102306220921060314715629080e2fb77",
  "amount": "0.05",
  "currency": "eth",
  "fee": "0.01"
}
{
  "status": "ok",
  "data": 700
}
```

####  POST /v1/dw/withdraw-virtual/{withdraw-id}/cancel 申請取消提現虛擬幣

請求參數:

| 參數名稱        | 是否必須 | 類型   | 描述 | 默認值  | 取值範圍 |
| ----------- | ---- | ---- | ------------ | ---- | ---- |
| withdraw-id | true | long | 提現ID，填在path中 |      |      |

響應數據:

| 參數名稱 | 是否必須  | 數據類型 | 描述    | 取值範圍 |
| ---- | ----- | ---- | ----- | ---- |
| data | false | long | 提現 ID |      |

請求響應例子:

```
/* POST /v1/dw/withdraw-virtual/{withdraw-id}/cancel */
{
  "status": "ok",
  "data": 700
}
```

####  GET /v1/query/deposit-withdraw 查詢虛擬幣充提記錄

請求參數:

| 參數名稱        | 是否必須 | 類型   | 描述 | 默認值  | 取值範圍 |
| ----------- | ---- | ---- | ------------ | ---- | ---- |
| currency | true | string | 幣種  |  |  |
| type | true | string | 'deposit' or 'withdraw'  |     |    |
| from   | false | string | 查詢起始 ID  |    |     |
| size   | false | string | 查詢記錄大小  |    |     |

響應數據:

| 參數名稱 | 是否必須 | 數據類型 | 描述 | 取值範圍 |
|-----|-----|-----|-----|------|
|   id  |  true  |  long  |   | |
|   type  |  true  |  long  | 類型 | 'deposit' 'withdraw' |
|   currency  |  true  |  string  |  幣種 | |
| tx-hash | true |string | 交易哈希 | |
| amount | true | long | 個數 | |
| address | true | string | 地址 | |
| address-tag | true | string | 地址標簽 | |
| fee | true | long | 手續費 | |
| state | true | string | 狀態 | 狀態參見下表 |
| created-at | true | long | 發起時間 | |
| updated-at | true | long | 最後更新時間 | |

###### 虛擬幣提現狀態定義：

| 狀態 | 描述  |
|--|--|
| submitted | 已提交 |
| reexamine | 審核中 |
| canceled  | 已撤銷 |
| pass    | 審批通過 |
| reject  | 審批拒絕 |
| pre-transfer | 處理中 |
| wallet-transfer | 已匯出 |
| wallet-reject   | 錢包拒絕 |
| confirmed      | 區塊已確認 |
| confirm-error  | 區塊確認錯誤 |
| repealed       | 已撤銷 |

###### 虛擬幣充值狀態定義：

|狀態|描述|
|--|--|
|unknown|狀態未知|
|confirming|確認中|
|confirmed|確認中|
|safe|已完成|
|orphan| 待確認|

請求響應例子:

```
/* GET /v1/query/deposit-withdraw?currency=xrp&type=deposit&from=5&size=12 */

{
    
    "status": "ok",
    "data": [
      {
        "id": 1171,
        "type": "deposit",
        "currency": "xrp",
        "tx-hash": "ed03094b84eafbe4bc16e7ef766ee959885ee5bcb265872baaa9c64e1cf86c2b",
        "amount": 7.457467,
        "address": "rae93V8d2mdoUQHwBDBdM4NHCMehRJAsbm",
        "address-tag": "100040",
        "fee": 0,
        "state": "safe",
        "created-at": 1510912472199,
        "updated-at": 1511145876575
      },
     ...
    ]
}
```



# 錯誤碼

## 行情 API 錯誤碼

| 錯誤碼  |  描述 |
|-----|-----|
| bad-request | 錯誤請求 |
| invalid-parameter | 參數錯 |
| invalid-command | 指令錯 |
code 的具體解釋, 參考對應的 `err-msg`.

## 交易 API 錯誤碼

| 錯誤碼  |  描述 |
|-----|-----|
| base-symbol-error |  交易對不存在 |
| base-currency-error |  幣種不存在 |
| base-date-error | 錯誤的日期格式 |
| account-transfer-balance-insufficient-error | 余額不足無法凍結 |
| bad-argument | 無效參數 |
| api-signature-not-valid | API簽名錯誤 |
| gateway-internal-error | 系統繁忙，請稍後再試|
|security-require-assets-password|需要輸入資金密碼|
|audit-failed| 下單失敗|
|ad-ethereum-addresss| 請輸入有效的以太坊地址|
|order-accountbalance-error| 賬戶余額不足|
| order-limitorder-price-error|限價單下單價格超出限制 |
|order-limitorder-amount-error|限價單下單數量超出限制 |
|order-orderprice-precision-error|下單價格超出精度限制 |
|order-orderamount-precision-error|下單數量超過精度限制|
|order-marketorder-amount-error|下單數量超出限制|
|order-queryorder-invalid|查詢不到此條訂單|
|order-orderstate-error|訂單狀態錯誤|
|order-datelimit-error|查詢超出時間限制|
|order-update-error|訂單更新出錯|