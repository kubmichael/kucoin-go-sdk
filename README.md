# GO SDK for KuCoin API
> The detailed document [https://docs.kucoin.com](https://docs.kucoin.com).

[![GoDoc](https://godoc.org/github.com/Kucoin/kucoin-go-sdk?status.svg)](https://godoc.org/github.com/Kucoin/kucoin-go-sdk)
[![Build Status](https://travis-ci.org/Kucoin/kucoin-go-sdk.svg?branch=master)](https://travis-ci.org/Kucoin/kucoin-go-sdk)
[![Go Report Card](https://goreportcard.com/badge/github.com/Kucoin/kucoin-go-sdk)](https://goreportcard.com/report/github.com/Kucoin/kucoin-go-sdk)

## Install

```bash
go get github.com/Kucoin/kucoin-go-sdk
```

## Usage

- The available environments

| Environment | BaseUri |
| -------- | -------- |
| *Production* `DEFAULT` | https://openapi-v2.kucoin.com |
| *Sandbox* | https://openapi-sandbox.kucoin.com |

- Create ApiService

```go
s := kucoin.NewApiService(
	// kucoin.ApiBaseURIOption("https://openapi-sandbox.kucoin.com"),
    kucoin.ApiKeyOption("key"),
    kucoin.ApiSecretOption("secret"),
    kucoin.ApiPassPhraseOption("passphrase"),
)

// Or add these options into the environmental variable
// Bash: 
// export API_BASE_URI=https://openapi-sandbox.kucoin.com
// export API_KEY=key
// export API_SECRET=secret
// export API_PASSPHRASE=passphrase
// s := NewApiServiceFromEnv()
```

- Example of API `without` authentication

```go
rsp, err := s.ServerTime()
if err != nil {
    // Handle error
}

var ts int64
if err := rsp.ReadData(&ts); err != nil {
    // Handle error
}
log.Printf("The server time: %d", ts)
```

- Example of API `with` authentication

```go
// Without pagination
rsp, err := s.Accounts("", "")
if err != nil {
    // Handle error
}

as := kucoin.AccountsModel{}
if err := rsp.ReadData(&as); err != nil {
    // Handle error
}

for _, a := range as {
    log.Printf("Available balance: %s %s => %s", a.Type, a.Currency, a.Available)
}
```

```go
// Handle pagination
rsp, err := s.Orders(map[string]string{}, &kucoin.PaginationParam{CurrentPage: 1, PageSize: 10})
if err != nil {
    // Handle error
}

os := kucoin.OrdersModel{}
pa, err := rsp.ReadPaginationData(&os)
if err != nil {
    // Handle error
}
log.Printf("Total num: %d, total page: %d", pa.TotalNum, pa.TotalPage)
for _, o := range os {
    log.Printf("Order: %s, %s, %s", o.Id, o.Type, o.Price)
}
```

- Example of WebSocket feed
> Require package [gorilla/websocket](https://github.com/gorilla/websocket)

```bash
go get github.com/gorilla/websocket
```

```go
mc, done, ec := s.WebSocketSubscribePublicChannel("/market/ticker:KCS-BTC", true)
var i = 0
type Ticker struct {
    Sequence    string `json:"sequence"`
    BestAsk     string `json:"bestAsk"`
    Size        string `json:"size"`
    BestBidSize string `json:"bestBidSize"`
    Price       string `json:"price"`
    BestAskSize string `json:"bestAskSize"`
    BestBid     string `json:"bestBid"`
}
for {
    select {
    case m := <-mc:
        // log.Printf("Received: %s", kucoin.ToJsonString(m))
        t := &Ticker{}
        if err := m.ReadData(t); err != nil {
            log.Printf("Failure to read: %s", err.Error())
            return
        }
        log.Printf("Ticker: %s, %s, %s", t.Sequence, t.Price, t.Size)
        i++
        if i == 3 {
            log.Println("Exit subscription")
            close(done)
            return
        }
    case err := <-ec:
        log.Printf("Error: %s", err.Error())
        close(done)
        return
    }
}
```

- More methods

![More methods](https://user-images.githubusercontent.com/7278743/51730854-fef1b900-20b3-11e9-8430-571b70443e4e.png)

## Run tests

```shell
# Add your API configuration items into the environmental variable first
export API_BASE_URI=https://openapi-sandbox.kucoin.com
export API_KEY=key
export API_SECRET=secret
export API_PASSPHRASE=passphrase

# Run tests
go test -v
```

## License

[MIT](LICENSE)