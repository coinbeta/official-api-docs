# WebSocket API簡介

WebSocket協議是基於TCP的壹種新的網絡協議。它實現了客戶端與服務器之間在單個 tcp 連接上的全雙工通信，由服務器主動發送信息給客戶端，減少了頻繁的身份驗證等不必要的開銷。其最大優點有兩個：


- 兩方請求的 header 數據很小，大概只有2 Bytes。

- 服務器不再是被動的接到客戶端的請求後才返回數據，而是有了新數據後主動推送給客戶端。

以上 WebSocket 協議帶來的優點使得其十分適用於數字貨幣行情和交易這種實時性強的接口。

# 請求與訂閱說明


### 1. 訪問地址

- 行情請求地址為：wss://{HOST}/ws

### 2. 數據壓縮
WebSocket API 返回的所有數據都進行了 GZIP 壓縮，需要 client 在收到數據之後解壓，推薦使用pako。（[【pako】](https://github.com/nodeca/pako) 是壹個支持壓縮和解壓 GZIP 的庫）

### 3. WebSocket庫
[【ws】](https://github.com/websockets/ws) 是 Node.js 下的 WebSocket 庫。

### 4. 心跳
WebSocket API 支持雙向心跳，無論是 Server 還是 Client 都可以發起 `ping` message，對方返回 `pong` message。

WebSocket Server 發送心跳：
```
{"ping": 18212558000}
```
 WebSocket Client 應該返回：
```
 {"pong": 18212558000}
```
註：返回的數據裏面的 `"pong"` 的值為收到的 `"ping"` 的值
註：WebSocket Client 和 WebSocket Server 建立連接之後，WebSocket Server 每隔 `5s`（這個頻率可能會變化） 會向 WebSocket Client 發起壹次心跳，WebSocket Client 忽略心跳2次後，WebSocket Server 將會主動斷開連接；WebSocket Client發送最近2次心跳message中的其中壹個`ping`的值，WebSocket Server都會保持WebSocket連接。
```
┌────────┐                         ┌────────┐ 
│ Client │                         │ Server │
└───┬────┘                         └───┬────┘
    │         {"ping": 18212558000}  │
    │<─────────────────────────────────┤
    │                                  │ wait 5s
    │                                  ├───┐
    │                                  │<──┘
    │         {"ping": 18212558000}  │
    │<─────────────────────────────────┤
    │                                  │
    │ {"pong": 18212558000}          │
    ├┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄>│
    │                                  │
```
註：WebSocket Client 發送最近2次心跳 message 中的其中壹個 `"ping"` 的值，WebSocket Server 都會保持 WebSocket 連接
```
┌────────┐                         ┌────────┐ 
│ Client │                         │ Server │
└───┬────┘                         └───┬────┘
    │         {"ping": 1523778470416}  │
    │<─────────────────────────────────┤
    │                                  │ wait 5s
    │                                  ├───┐
    │                                  │<──┘
    │         {"ping": 1523778475416}  │
    │<─────────────────────────────────┤
    │                                  │ wait 5s
    │                                  ├───┐
    │                                  │<──┘
    │                                  │
    │                                  │ close WebSocket connection
    │                                  ├───┐
    │                                  │<──┘
    │                                  │

```
註：WebSocket Client 忽略心跳2次後，WebSocket Server 將會主動斷開連接。
WebSocket Client 發送心跳：
```json
{"ping": 18212553000}
```
註：發送的 message 裏面，"ping" 的值必須為 Long 類型，否則返回錯誤信息：
```json
{
  "ts": 1492420473027,
  "status": "error",
  "err-code": "bad-request",
  "err-msg": "invalid ping"
}
```

WebSocket Server 會返回：
```
{"pong": 18212553000}
```
註：返回的數據裏面的 `"pong"` 的值為收到的 `"ping"` 的值

錯誤信息返回格式
```
{
  "id": "id generate by client",
  "status": "error",
  "err-code": "err-code",
  "err-msg": "err-message",
  "ts": 1487152091345
}
```
註：`ts`為錯誤信息生成的時間戳，單位：毫秒
### 5. topic格式

訂閱數據和請求數據都要使用 `topic`，`topic` 的語法如下：

| topic 類型      | topic 語法                    |sub/req     | 描述                                       |
| ------------- | ---------------------------- | ---------------|------------------------- |
| KLine         | market.$symbol.kline.$period | sub/req |K線 數據，包含單位時間區間的開盤價、收盤價、最高價、最低價、成交量、成交額、成交筆數等數據 $period 可選值：{ 1min, 5min, 15min, 30min, 60min, 4hour,1day, 1mon, 1week, 1year } |
| Market Depth  | market.$symbol.depth.$type   | sub/req |盤口深度，按照不同 step 聚合的買壹、買二、買三等和賣壹、賣二、賣三等數據 $type 可選值：{ step0, step1, step2, step3, step4, step5, percent10 } （合並深度0-5）；step0時，不合並深度|
| Trade Detail  | market.$symbol.trade.detail  |  sub/req |  成交記錄，包含成交價格、成交量、成交方向等信息                                      |
| Market Detail | market.$symbol.detail        |  sub/req |   最近24小時成交量、成交額、開盤價、收盤價、最高價、最低價、成交筆數等                                     |
| Market Tickers | market.tickers  |  sub|   所有對外公開交易對的 日K線、最近24小時成交量等信息                            |

* `$symbol` 為交易對，可選值： { ethbtc, ltcbtc, etcbtc, bchbtc...... }
* 用戶選擇“合並深度”時，壹定報價精度內的市場掛單將予以合並顯示。合並深度僅改變顯示方式，不改變實際成交價格。

### 6. 請求數據(req/rep)

請求數據，僅返回壹次數據

#### 請求數據的格式

```json
{
  "req": "topic to req",
  "id": "id generate by client"
}
```
* `"req"` 的值為 **topic** ，請參考 `"5. topic格式"` 的 **topic 格式**

**正確請求數據的例子**

```json
{
  "req": "market.btcusdt.kline.1min",
  "id": "id10"
}
```

返回數據的例子：
```json
{
  "status": "ok",
  "rep": "market.btcusdt.kline.1min",
  "tick": [
    {
      "amount": 1.6206,
      "count":  3,
      "id":     1494465840,
      "open":   9887.00,
      "close":  9885.00,
      "low":    9885.00,
      "high":   9887.00,
      "vol":    16021.632026
    },
    {
      "amount": 2.2124,
      "count":  6,
      "id":     1494465900,
      "open":   9885.00,
      "close":  9880.00,
      "low":    9880.00,
      "high":   9885.00,
      "vol":    21859.023500
    }
  ]
}
```

**錯誤請求數據的例子**

```json
{
  "req": "market.invalidsymbo.kline.1min",
  "id": "id10"
}
```

返回的錯誤信息的例子：
```json
{
  "status": "error",
  "id": "id10",
  "err-code": "bad-request",
  "err-msg": "invalid topic market.invalidsymbol.trade.detail",
  "ts": 1494483996521
}
```


### 7. 訂閱數據(sub)

#### 訂閱數據(sub)以及接收訂閱數據的大致流程
```
┌────────┐                         ┌────────┐ 
│ Client │                         │ Server │
└───┬────┘                         └───┬────┘
    │ {"sub": "topic"}                 │
    ├─────────────────────────────────>│
    │                                  │
    │              {"subbed": "topic"} │
    │<┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┤
    │                                  │
    │        {"tick": "data of topic"} │
    │<┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┤
    │                                  │
    │        {"tick": "data of topic"} │
    │<┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┄┤
    │                                  │
```
註：訂閱 topic 成功之後，當 topic 對應的數據有更新時，Server 按壹定的頻率把 topic 對應的新數據推送給 Client
#### 訂閱數據的格式

成功建立和 WebSocket API 的連接之後，向 Server 發送如下格式的數據來訂閱數據：
```json
{
   "id": "id generate by client",
  "sub": "topic to sub",
  "freq-ms": 1000
}
```
註：`id` 參數是可選的
註：`freq-ms` 參數是可選的，可選值為 { `1000`, `2000`, `3000`, `4000`, `5000` }，不傳 `freq-ms` 時默認為` 0 `也就是有新的數據就馬上推送到 Client，`freq-ms` 決定 Server 推送的頻率
**正確訂閱的例子**

正確訂閱：
```json
{
  "sub": "market.btcusdt.kline.1min",
  "id": "id1"
}
```
* `"sub"` 的值為 **topic** ，請參考 `"5. topic格式"` 的 **topic 格式**

訂閱成功返回數據的例子：
```json
{
  "id": "id1",
  "status": "ok",
  "subbed": "market.btcusdt.kline.1min",
  "ts": 1489474081631
}
```

之後每當 KLine 有更新時，client 會收到數據，例子：
```json
{
  "ch": "market.btcusdt.kline.1min",
  "ts": 1489474082831,
  "tick": {
    "id": 1489464480,
    "amount": 0.0,
    "count": 0,
    "open": 7962.62,
    "close": 7962.62,
    "low": 7962.62,
    "high": 7962.62,
    "vol": 0.0
  }
}
```
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
  }

```


**錯誤訂閱的例子**

錯誤訂閱（錯誤的 symbol）：
```
{
  "sub": "market.invalidsymbol.kline.1min",
  "id": "id2"
}
```

訂閱失敗返回數據的例子：
```json
{
  "id": "id2",
  "status": "error",
  "err-code": "bad-request",
  "err-msg": "invalid topic market.invalidsymbol.kline.1min",
  "ts": 1494301904959
}
```

錯誤訂閱（錯誤的 topic）：
```json
{
  "sub": "market.btcusdt.kline.3min",
  "id": "id3"
}
```
訂閱失敗返回數據的例子：
```json
{
  "id": "id3",
  "status": "error",
  "err-code": "bad-request",
  "err-msg": "invalid topic market.btcusdt.kline.3min",
  "ts": 1494310283622
}
```

### 8. 取消訂閱(unsub)

#### 取消訂閱的格式

WebSocket Client 訂閱數據之後，可以取消訂閱，取消訂閱之後 WebSocket Server 將不會再發送該 topic 的數據，取消訂閱的格式如下：

```json
{
  "unsub": "topic to unsub",
  "id": "id generate by client"
}
```

**正確取消訂閱的例子**

正確取消訂閱的例子：
```json
{
  "unsub": "market.btcusdt.trade.detail",
  "id": "id4"
}
```

取消訂閱成功返回信息的例子：
```json
{
  "id": "id4",
  "status": "ok",
  "unsubbed": "market.btcusdt.trade.detail",
  "ts": 1494326028889
}
```

**錯誤取消訂閱的例子**

錯誤取消訂閱的例子（取消訂閱壹個尚未訂閱的 topic）：
```json
{
  "unsub": "market.btcusdt.trade.detail",
  "id": "id5"
}
```

返回的錯誤信息的例子
```json
{
  "id": "id5",
  "status": "error",
  "err-code": "bad-request",
  "err-msg": "unsub with not subbed topic market.btcusdt.trade.detail",
  "ts": 1494326217428
}
```

錯誤取消訂閱的例子（取消訂閱壹個不存在的 topic）：
```json
{
  "unsub": "not-exists-topic",
  "id": "id5"
}
```
返回的錯誤信息的例子：
```json
{
  "id": "id5",
  "status": "error",
  "err-code": "bad-request",
  "err-msg": "unsub with not subbed topic not-exists-topic",
  "ts": 1494326318809
}
```

# WebSocket API Reference

## 訂閱 KLine 數據 market.$symbol.kline.$period

成功建立和 WebSocket API 的連接之後，向 Server 發送如下格式的數據來訂閱數據：
```json
{
  "sub": "market.$symbol.kline.$period",
  "id": "id generate by client"
}
```

| 參數名稱         | 是否必須  | 類型     | 描述                | 默認值   | 取值範圍                                     |
| ------------ | ----- | ------ | ----------------- | ----- | ---------------------------------------- |
| symbol       | true  | string | 交易對                |       | ethbtc,btusdt... |
| period       | true  | string | K線周期          |       | 1min, 5min, 15min, 30min, 60min, 1day, 1mon, 1week, 1year |

**正確訂閱的例子**

正確訂閱
```
{
  "sub": "market.btcusdt.kline.1min",
  "id": "id1"
}
```

訂閱成功返回數據的例子
```
{
  "id": "id1",
  "status": "ok",
  "subbed": "market.btcusdt.kline.1min",
  "ts": 1489474081631
}
```

之後每當 KLine 有更新時，client 會收到數據，例子
```json
{
  "ch": "market.btcusdt.kline.1min",
  "ts": 1489474082831,
  "tick": {
    "id": 1489464480,
    "amount": 0.0,
    "count": 0,
    "open": 7962.62,
    "close": 7962.62,
    "low": 7962.62,
    "high": 7962.62,
    "vol": 0.0
  }
}
```

tick 說明
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
  }

```

**錯誤訂閱的例子**

錯誤訂閱(錯誤的 symbol)
```json
{
  "sub": "market.invalidsymbol.kline.1min",
  "id": "id2"
}
```

訂閱失敗返回數據的例子
```json
{
  "id": "id2",
  "status": "error",
  "err-code": "bad-request",
  "err-msg": "invalid topic market.invalidsymbol.kline.1min",
  "ts": 1494301904959
}
```

錯誤訂閱(錯誤的 topic)
```
{
  "sub": "market.btcusdt.kline.3min",
  "id": "id3"
}
```

訂閱失敗返回數據的例子
```json
{
  "id": "id3",
  "status": "error",
  "err-code": "bad-request",
  "err-msg": "invalid topic market.btcusdt.kline.3min",
  "ts": 1494310283622
}
```

## 請求 KLine 數據 market.$symbol.kline.$period  

```
{
  "req": "market.$symbol.kline.$period",
  "id": "id generated by client",
  "from": 1533536947, //optional, type: long, 2017-07-28T00:00:00+08:00 至 2050-01-01T00:00:00+08:00 之間的時間點，單位：秒
  "to": 1533536947 //optional, type: long, 2017-07-28T00:00:00+08:00 至 2050-01-01T00:00:00+08:00 之間的時間點，單位：秒，必須比 from 大
}
```

| 參數名稱         | 是否必須  | 類型     | 描述                | 默認值   | 取值範圍                                     |
| ------------ | ----- | ------ | ----------------- | ----- | ---------------------------------------- |
| symbol       | true  | string | 交易對                |       | btcusdt, ethusdt, ltcusdt...|
| period       | true  | string | K線周期          |       | 1min, 5min, 15min, 30min, 60min, 1day, 1mon, 1week, 1year |

#### 請求 KLine 數據的例子

```
{
  "req": "market.btcusdt.kline.1min",
  "id": "id10"
}
```

返回數據的例子
```
{
  "rep": "market.btcusdt.kline.1min",
  "status": "ok",
  "id": "id10",
  "tick": [
    {
      "amount": 17.4805,
      "count":  27,
      "id":     1494478080,
      "open":   10050.00,
      "close":  10058.00,
      "low":    10050.00,
      "high":   10058.00,
      "vol":    175798.757708
    },
    {
      "amount": 15.7389,
      "count":  28,
      "id":     1494478140,
      "open":   10058.00,
      "close":  10060.00,
      "low":    10056.00,
      "high":   10065.00,
      "vol":    158331.348600
    },
    // more KLine data here
  ]
}
```

## 訂閱 Market Depth 數據 market.$symbol.depth.$type 

成功建立和 WebSocket API 的連接之後，向 Server 發送如下格式的數據來訂閱數據：
```
{
  "sub": "market.$symbol.depth.$type",
  "id": "id generated by client"
}
```

| 參數名稱         | 是否必須  | 類型     | 描述                | 默認值   | 取值範圍                                     |
| ------------ | ----- | ------ | ----------------- | ----- | ---------------------------------------- |
| symbol       | true  | string | 交易對                |       | btcusdt, ethusdt, ltcusdt, etcusdt, bchusdt, ethbtc, ltcbtc, etcbtc, bchbtc... |
| type         | true  | string | Depth 類型          |       | step0, step1, step2, step3, step4, step5（合並深度0-5）；step0時，不合並深度 |

* 用戶選擇“合並深度”時，壹定報價精度內的市場掛單將予以合並顯示。合並深度僅改變顯示方式，不改變實際成交價格。

正確訂閱例子
```
{
  "sub": "market.btcusdt.depth.step0",
  "id": "id1"
}
```

訂閱成功返回數據的例子
```
{
  "id": "id1",
  "status": "ok",
  "subbed": "market.btcusdt.depth.step0",
  "ts": 1489474081631
}
```

之後每當 depth 有更新時，client 會收到數據，例子
```
{
  "ch": "market.btcusdt.depth.step0",
  "ts": 1489474082831,
  "tick": {
    "bids": [
    [9999.3900,0.0098], // [price, amount]
    [9992.5947,0.0560],
    // more Market Depth data here
    ]
    "asks": [
    [10010.9800,0.0099]
    [10011.3900,2.0000]
    //more data here
    ]
  }
}
```

tick 說明
```
  "tick": {
    "bids": [
    [買1價,買1量]
    [買2價,買2量]
    //more data here
    ]
    "asks": [
    [賣1價,賣1量]
    [賣2價,賣2量]
    //more data here
    ]
  }
```

## 請求 Market Depth 數據 market.$symbol.depth.$type 

```
{
  "req": "market.$symbol.depth.$type",
  "id": "id generated by client"
}
```

#### 請求 Market Depth 數據的例子
```
{
  "req": "market.btcusdt.depth.step0",
  "id": "id10"
}
```

返回數據的例子：
```
{
  "rep": "market.btcusdt.depth.step0",
  "status": "ok",
  "id": "id10",
  "tick": {
    "bids": [
    [9999.3900,0.0098], // [price, amount]
    [9992.5947,0.0560],
    // more Market Depth data here
    ]
    "asks": [
    [10010.9800,0.0099]
    [10011.3900,2.0000]
    //more data here
    ]
  }
}
```

## 訂閱 Trade Detail 數據 market.$symbol.trade.detail  

```
{
  "sub": "market.$symbol.trade.detail",
  "id": "id generated by client"
}
```

| 參數名稱         | 是否必須  | 類型     | 描述                | 默認值   | 取值範圍                                     |
| ------------ | ----- | ------ | ----------------- | ----- | ---------------------------------------- |
| symbol       | true  | string | 交易對                |       |btcusdt, ethusdt, ltcusdt, etcusdt, bchusdt, ethbtc, ltcbtc, etcbtc, bchbtc... |

正確訂閱例子：

```
{
  "sub": "market.btcusdt.trade.detail",
  "id": "id1"
}
```

訂閱成功返回數據的例子：
```
{
  "id": "id1",
  "status": "ok",
  "subbed": "market.btcusdt.trade.detail",
  "ts": 1489474081631
}
```

之後每當 Trade Detail 有更新時，client 會收到數據，例子：

```
{
  "ch": "market.btcusdt.trade.detail",
  "ts": 1489474082831,
  "tick": {
        "id": 14650745135,
        "ts": 1533265950234,
        "data": [
            {
                "amount": 0.0099,
                "ts": 1533265950234,
                "id": 146507451359183894799,
                "price": 401.74,
                "direction": "buy"
            },
            // more Trade Detail data here
        ]
    }
  
  ]
  }
}
```

data 說明：
```
  "data": [
    {
      "id":        消息ID,
      "price":     成交價,
      "time":      成交時間,
      "amount":    成交量,
      "direction": 成交方向,
      "tradeId":   成交ID,
      "ts":        時間戳
    }
  ]
```

## 請求 Trade Detail 數據 market.$symbol.trade.detail 

```
{
  "req": "market.$symbol.trade.detail",
  "id": "id generated by client"
}
```

* 僅能獲取最近 300 個 Trade Detail 數據

#### 請求 Trade Detail 數據的例子
```
{
  "req": "market.btcusdt.trade.detail",
  "id": "id11"
}
```

返回數據的例子：
```
{
  "rep": "market.btcusdt.trade.detail",
  "status": "ok",
  "id": "id11",
  "data": [
    {
      "id":        601595424,
      "price":     10195.64,
      "time":      1494495766,
      "amount":    0.2943,
      "direction": "buy",
      "tradeId":   601595424,
      "ts":        1494495766000
    },
    {
      "id":        601595423,
      "price":     10195.64,
      "time":      1494495711,
      "amount":    0.2430,
      "direction": "buy",
      "tradeId":   601595423,
      "ts":        1494495711000
    },
    // more Trade Detail data here
  ]
}
```

## 請求 Market Detail 數據 market.$symbol.detail 
```
{
  "req": "market.$symbol.detail",
  "id": "id generated by client"
}
```

* 僅返回當前 Market Detail

#### 請求 Market Detail 數據的例子

```
{
  "req": "market.btcusdt.detail",
  "id": "id12"
}
```

返回數據的例子：
```json
{
  "rep": "market.btcusdt.detail",
  "status": "ok",
  "id": "id12",
  "tick": {
    "amount": 12224.2922,
    "open":   9790.52,
    "close":  10195.00,
    "high":   10300.00,
    "ts":     1494496390000,
    "id":     1494496390,
    "count":  15195,
    "low":    9657.00,
    "vol":    121906001.754751
  }
}
```
