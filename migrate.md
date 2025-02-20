# PHP接口迁移至Golang流程

### 1. 接口鉴权分析

需要分析源接口和目标接口的以下特性:
- 是否需要加解密
- 是否需要登录验证
- 是否需要特殊权限

#### 源接口(PHP)分析
1. 检查`routes`文件中的中间件配置
2. 在`app/Http/kernel.php`和`app/Http/Middleware`中确认中间件具体实现

示例路由配置:
```php
Route::group(['middleware' => ['crypt.basic', 'auth.token', 'vip.route']], function () {
    Route::group(["prefix" => "v1/warning", "namespace" => "V1"], function () {
        Route::post("list", "WarningController@getWarningList");
    });
});
```
此示例中使用了三个中间件:
- `crypt.basic`: 加解密
- `auth.token`: 登录验证
- `vip.route`: VIP权限验证

#### 目标接口(Golang)配置
1. 检查`main.go`是否配置全局加解密中间件
2. 在对应`routes`文件中配置所需中间件

示例路由配置:
```go
warningV1 := app.Party("/api/v1/warning")
warningV1.Use(middleware.CheckLogin)
warningV1.Post("/token/get", controller.GetWarningToken)
warningV1.Post("/list", controller.GetWarningList)
```
`warningV1.Use(middleware.CheckLogin)`表示该路由需要登录
### 2. 请求参数解析

#### 获取加密请求参数
1. 打开pc端开发者工具
```bash:terminal
DEBUG=APP* OPEN_DEVTOOLS_IN_PRODUCTION=1 /Applications/AICoin.app/Contents/MacOS/AICoin
```
2. 在网络面板中找到目标接口
3. 查看"载荷"标签页
4. 右键复制加密的请求参数
#### 解密请求参数
使用项目提供的解密工具:
```go
// go-pc-api/test/decrypt_test.go，修改以下参数值
json := `{
    "p": "4Po7jd5ukqqyBfJZfyXdpx...",
    "k": "DXE1KS2QgZff3rhySqbI8r...",
    "v": "1535898394",
    "iv": "V1wcaPjzDw+a0MM09X7Wz..."
}`

// 运行测试
go test -v test/decrypt_test.go
```
确认请求参数的类型,解密后的数据如下(token此处省略)，此处type为int类型,last_id为string类型等等
```
=== RUN   TestDecryptRequestBody
    decrypt_test.go:34: {"db_key":"","type":2,"last_id":"54986020","size":20,"last_time":"1739842355","lan":"cn","pc_client":"Mac arm64","pc_client_version":"2.12.3","token":"..."}
map[db_key: lan:cn last_id:54986020 last_time:1739842355 pc_client:Mac arm64 pc_client_version:2.12.3 size:20 token:... type:2]
map[classify:[] coin: lan:cn market: pc_client:Mac x64 pc_client_version:2.3.1-alpha.7 token:...]
--- PASS: TestDecryptRequestBody (0.00s)
PASS
ok      command-line-arguments  0.400s
```
### 3.golang项目使用iris框架实现controller流程
1.

### 3. 接口返回值对比
- 对于需要登录的接口，需要先参考解密请求参数的步骤，获取到token，添加到请求体中
- 可使用以下python脚本，将源接口的返回数据和迁移接口的返回数据保存为json文件，然后进行对比
```python
import json
def compare_json_responses(response1, response2):
    """比较两个 JSON 响应的键和值差异"""
    
    def compare_dicts(dict1, dict2, path=""):
        """递归比较两个字典的键和值"""

        all_keys = set(dict1.keys()) | set(dict2.keys())

        
        for key in all_keys:
            current_path = f"{path}.{key}" if path else key
            
            # 检查键是否存在
            if key not in dict1:
                print(f"键只存在于响应2中: {current_path}")
                continue
            if key not in dict2:
                print(f"键只存在于响应1中: {current_path}")
                continue
            
            # 获取值
            value1 = dict1[key]
            value2 = dict2[key]
            
            # 如果值是字典，递归比较
            if isinstance(value1, dict) and isinstance(value2, dict):
                compare_dicts(value1, value2, current_path)
            # 如果值是列表，比较列表内容
            elif isinstance(value1, list) and isinstance(value2, list):
                if len(value1) != len(value2):
                    print(f"列表长度不同 {current_path}: {len(value1)} vs {len(value2)}")
                for i, (item1, item2) in enumerate(zip(value1, value2)):
                    if isinstance(item1, dict) and isinstance(item2, dict):
                        compare_dicts(item1, item2, f"{current_path}[{i}]")
                    elif item1 != item2:
                        print(f"不一致 {current_path}[{i}]: {item1} vs {item2}")
            # 比较普通值
            elif value1 != value2:
                print(f"不一致 {current_path}: {value1} vs {value2}")
    
    compare_dicts(response1, response2)


if __name__ == "__main__":
   
    with open('/Users/anndian/Desktop/go-pc/go-pc-api/response1.json', 'r') as f:
        response1 = json.load(f)
    
    with open('/Users/anndian/Desktop/go-pc/go-pc-api/response2.json', 'r') as f:
        response2 = json.load(f)

    compare_json_responses(response1, response2)
        
```

#### 本地测试
1. 在测试数据库中构造与线上相同的测试数据
2. 使用YAPI对比本地接口与线上接口的返回结果
#### 测试环境测试
1. golang项目切到develop分支，配置接口转发规则:
   - 修改`kubernetes/ingress_dev.yaml`文件(有两处需要修改)
    例如转发api/v1/warning/list接口：
    ```yaml
    # ingress_dev.yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
    annotations:
        ingress.cloud.tencent.com/client-token: 01fe5a89-4e13-4ed3-a01e-7ffb85734aa9
        ingress.cloud.tencent.com/direct-access: "true"
        kubernetes.io/ingress.class: qcloud
        kubernetes.io/ingress.https-rules: "null"
        kubernetes.io/ingress.qcloud-loadbalance-id: lb-1bms6auc
        kubernetes.io/ingress.rule-mix: "false"
        kubernetes.io/ingress.subnetId: subnet-29xkf2g2
    name: pc-dev-login-ingress
    namespace: bg-api
    spec:
    rules:
    # 其余规则不变，添加以下规则
    - http:
        paths:
        - backend:
            service:
                name: api-pc
                port:
                number: 80
            path: /api/v1/warning/list
            pathType: Prefix
    ```

   - 修改`kubernetes/ingress_dev_login.yaml`文件

    ```yaml
    # ingress_dev.yaml
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
    annotations:
        kubernetes.io/ingress.rule-mix: "false"
        nginx.ingress.kubernetes.io/use-regex: "true"
        nginx.ingress.kubernetes.io/ssl-redirect: "false"
    name: apipc-dev-nginx-route
    namespace: bg-api
    spec:
    ingressClassName: nginx
    rules:
    - host: apipc-dev.aicoin.com
        http:
        paths:
        # 其余规则不变，添加以下规则
        - backend:
            service:
                name: go-api-pc
                port:
                number: 8000
            path: /api/v1/warning/list
            pathType: Prefix
    ---
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
    annotations:
        alb.ingress.kubernetes.io/group.name: apipc-dev-aicoin-inc-com-alb
        alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}]'
        alb.ingress.kubernetes.io/scheme: internal
        alb.ingress.kubernetes.io/target-type: ip
        alb.ingress.kubernetes.io/tags: map-migrated=migEZLOF6DH6W,YongTu-Eks=Sp-Dev,YongTu-Environment=Development,YongTu-Department=AI技术事业部,YongTu-Application=AICoin,YongTu-Owner=运维部
        alb.ingress.kubernetes.io/subnets: subnet-09b45e917bcc4c54d,subnet-0739142881e764427
        alb.ingress.kubernetes.io/success-codes: 404,200
        kubernetes.io/ingress.class: alb
    name: apipc-dev-aicoin-inc-com-route
    namespace: bg-api
    spec:
    ingressClassName: alb
    rules:
    - host: 
        http:
        paths:
        # 其余规则不变，添加以下规则
        - backend:
            service:
                name: go-api-pc
                port:
                number: 8000
            path: /api/v1/warning/list
            pathType: Prefix

    ```
2. 将本地代码合并到develop分支,并推送到远程，推送前格式化go代码
示例格式化命令：
```bash
gofmt -s -w ./internal/controller/custom_indicator_controller.go
```
3. 验证转发是否成功:
   - 打开AICoinTest应用开发者工具
   ```bash
   DEBUG=APP* OPEN_DEVTOOLS_IN_PRODUCTION=1 /Applications/AICoinTest.app/Contents/MacOS/AICoinTest
   ```
   - 检查响应头中是否包含`x-upgrade=true`标记

### 4. 应用端功能正常，达到发版要求
1. golang 项目转到master分支，新开一个分支，参考原文件写法配置接口转发规则
- 修改`kubernetes/ingress.yaml`文件
- 修改`kubernetes/ingress_anycast.yaml`文件    
- 修改`kubernetes/ingress_vip.yaml`文件
- 修改`kubernetes/ingress_anycast.yaml`文件
2. 验证转发是否成功

# Go接口开发核心函数详解(iris框架)

## 1. 请求参数处理

### 1.1 GetRequestParams 函数
这个函数用于解析和验证请求参数:

```go
// httputil/request.go
func GetRequestParams(ctx iris.Context, params interface{}) error {
    // 获取请求体
    bodyBytes := GetRequestBodyFromValues(ctx)
    
    // JSON解析
    if err := jsoniter.Unmarshal(bodyBytes, params); err != nil {
        return err
    }
    
    // 参数验证
    if err := GetValidator().Struct(params); err != nil {
        return err
    }
    
    return nil
}
```

使用示例:

```go
// 添加validate标签，用于验证请求参数的值是否符合要求,如没有添加则只验证参数类型
type AlertParams struct {
    Symbol    string  `json:"symbol" validate:"required"`
    Mode      int     `json:"mode" validate:"required,oneof=1 2 3"`
    Price     float64 `json:"price" validate:"required,gt=0"`
}

func AddAlert(ctx iris.Context) {
    // 定义请求参数结构体
    var params AlertParams
    // 获取请求参数并验证类型
    if err := httputil.GetRequestParams(ctx, &params); err != nil {
        httputil.NewResponse().JSONFailedNoData(ctx, err.Error(), 400)
        return
    }
    // 参数验证通过,继续处理...
}
```
### 1.2 GetRequestBodyFromValues 函数

```go
// 从已解析的结果中，提取请求体内容并转换为字节数组返回
func GetRequestBodyFromValues(ctx iris.Context) []byte {
	bodyBytes := []byte{}

	val, status := ctx.Values().GetEntry("bodyBytes")
	if !status {
		return bodyBytes
	}

	if val.ValueRaw != nil {
		rType := reflect.TypeOf(val.ValueRaw)
		if rType == reflect.TypeOf([]byte{}) {
			bodyBytes = reflect.ValueOf(val.ValueRaw).Bytes()
		}
	}

	return bodyBytes
}
```
使用示例:
```go
// 从已解析的结果中，提取请求体内容并转换为字节数组返回
bodyBytes := httputil.GetRequestBodyFromValues(ctx)
```
### 1.3 ParseGolbalParams 函数
用于从请求体中解析或者添加全局通用参数: Currency, OpenTime, Lan

```go
// tools/global_params.go
func ParseGolbalParams(bodyBytes []byte) config.QuoteParamConfig {
	logger := logutil.GetLogger()
	defer logger.Sync()
	sugar := logger.Sugar()

	type param struct {
		Currency string      `json:"currency"`
		OpenTime interface{} `json:"open_time"`
		Lan      string      `json:"lan"`
	}

	var params param

	jsonErr := json.Unmarshal(bodyBytes, &params)
	if jsonErr != nil {
		sugar.Warn(jsonErr)
	}

	var quoteParam config.QuoteParamConfig
	var formatOpenTime string
	openTime := params.OpenTime
	rV := reflect.ValueOf(openTime)
	if rV.Kind() == reflect.Float64 {
		formatOpenTime = fmt.Sprintf("%.0f", rV.Float())
	} else if rV.Kind() == reflect.String {
		formatOpenTime = rV.String()
	}

	switch formatOpenTime {
	case "0":
		quoteParam.OpenTime = config.OPEN_TIME_E8

	case "8":
		quoteParam.OpenTime = config.OPEN_TIME_E0

	case "24":
		quoteParam.OpenTime = config.OPEN_TIME_24H

	default:
		quoteParam.OpenTime = config.OPEN_TIME_24H
	}
	switch params.Currency {
	case "cny":
		quoteParam.Currency = config.CURRENCY_CNY

	case "usd":
		quoteParam.Currency = config.CURRENCY_USD

	case "raw":
		quoteParam.Currency = config.CURRENCY_RAW

	default:
		quoteParam.Currency = config.CURRENCY_CNY
	}
	switch params.Lan {
	case "en":
		quoteParam.Lan = config.LAN_EN

	case "cn":
		quoteParam.Lan = config.LAN_CN

	case "tc":
		quoteParam.Lan = config.LAN_TC

	default:
		quoteParam.Lan = config.LAN_CN
	}

	return quoteParam
}
```

使用示例:

```go
func GetAlertList(ctx iris.Context) {
    bodyBytes := httputil.GetRequestBodyFromValues(ctx)
    quoteParam := tools.ParseGolbalParams(bodyBytes)
    
    // 根据语言返回对应的提示信息
    if quoteParam.Lan == config.LAN_EN {
        msg = "Alert added successfully"
    } else {
        msg = "预警添加成功"
    }
}
```
### 1.4 GetSessionByCtx 函数
获取session信息
```go
// httputil/session.go
func GetSessionByCtx(ctx iris.Context) *SessionData {
	entry, ok := ctx.Values().GetEntry("session")
	if !ok {
		return nil
	}

	session, ok := entry.ValueRaw.(*SessionData)
	if !ok || session == nil {
		return nil
	}

	return session
}
```
session结构体定义:
```go
// httputil/session.go
type SessionData struct {
	UserId                     string
	SessionId                  string
	ProEndTimestamp            int64
	ArbiEndTimestamp           int64
	SignalEndTimestamp         int64 // 指标胜率会员
	IndicatorAlertEndTimestamp int64 // 信号预警会员
}
```
使用示例:
```go
// 获取session信息
session := httputil.GetSessionByCtx(ctx)
// 取具体字段
session.UserID
session.Token
```
## 2. 响应处理

### 2.1 Response 结构体
标准化的响应结构:

```go
// httputil/response.go
type apiAgent int

const (
	// PCAPIAgent 标识响应的客户端类型为pc端
	PCAPIAgent = apiAgent(0)
	// AppNewAPIAgent 标识响应的客户端类型为新版app
	AppNewAPIAgent = apiAgent(1)
	// WebNeAPIwAgent 标识响应的客户端类型为新版网页
	WebNeAPIwAgent = apiAgent(2)
	// BackAPIAgent 标识响应的客户端类型为管理后台
	BackAPIAgent = apiAgent(3)
)

type Response struct {
    Agent       apiAgent
    HTTPCode    int
    HTTPHeader  map[string]string
    BodySuccess bool
    BodyData    interface{}
    BodyMsg     string  
    BodyCode    int
}

// 成功响应
func (r *Response) JSONSuccess(ctx iris.Context, data interface{}, msg string, code int) {
    r.HTTPCode = 200
    r.BodySuccess = true
    r.BodyData = data
    r.BodyMsg = msg
    r.BodyCode = code
    
    r.BuildJSONResponse(ctx)
}

// 错误响应
func (r *Response) JSONFailedNoData(ctx iris.Context, msg string, code int) {
    r.HTTPCode = 200
    r.BodySuccess = false
    r.BodyMsg = msg
    r.BodyCode = code
    
    r.BuildJSONResponse(ctx)
}
```

使用示例:

```go
// 指定对应agent
var res httputil.Response
res.Agent = httputil.PCAPIAgent
func AddAlert(ctx iris.Context) {
    // 成功响应
    data := map[string]interface{}{
        "alert_id": 123,
    }
    res.JSONSuccess(ctx, data, "添加成功", 200)
    
    // 错误响应
    res.JSONFailedNoData(ctx, "参数错误", 400)
}
```
## 3.builder构建器使用
### 3.1 TpBuilder 构建器
获取交易对数据示例代码,具体使用时，需要根据实际情况，添加对应的配置
```go
import (
	"team.aicoin.net.cn/lab/sosobtc-backend/go-core-lib/pkg/quote"
)
// 获取请求体
bodyBytes := httputil.GetRequestBodyFromValues(ctx)
// 解析请求体,得到quoteParam
quoteParam := tools.ParseGolbalParams(bodyBytes)
// 获取交易对详情

// 使用quote.NewTPBuilder(tradingList)获取交易对详情,其中参数为切片类型，表示dbkey
tpBuilder := quote.NewTPBuilder(tradingList)
// 设置是否过滤下线交易对
tpBuilder.BaseBuilder.IsFilterOffline = false
// 添加配置，addconfig的第一个参数为字符串类型，表示要获取的配置字段名,第二个参数为字符串类型，表示输出结果中的字段名,第三个参数为quoteParam类型，包含默认值等信息
// 获取货币
tpBuilder.AddConfig("currency_str", "currency", quoteParam)
// 获取是否展示
tpBuilder.AddConfig("coin_show", "show", quoteParam)
// 获取货币符号
tpBuilder.AddCurrencySymbol("symbol", quoteParam)
// 获取市场名称，后面匹配语言来获取
tpBuilder.AddConfig("market_cn", "market_cn", quoteParam)
tpBuilder.AddConfig("market_en", "market_en", quoteParam)
// 使用GetResult获取交易对详情
tradingDetail := tpBuilder.BaseBuilder.GetResult()

// 取具体字段
tradingDetail["market_cn"]
tradingDetail["symbol"]
tradingDetail["currency"]

```

### 3.2 MarketBuilder 构建器
获取市场数据示例代码,具体使用时，需要根据实际情况，添加对应的配置
```go
import (
	"team.aicoin.net.cn/lab/sosobtc-backend/go-core-lib/pkg/quote"
)
// 获取请求体
bodyBytes := httputil.GetRequestBodyFromValues(ctx)
// 解析请求体,得到quoteParam
quoteParam := tools.ParseGolbalParams(bodyBytes)
// 使用quote.NewMarketBuilder(platformList)获取市场数据,其中参数为切片类型，表示平台列表
marketBuilder := quote.NewMarketBuilder(platformList)
// 添加配置，addconfig的第一个参数为字符串类型，表示要获取的配置字段名,第二个参数为字符串类型，表示输出结果中的字段名,第三个参数为quoteParam类型，包含默认值等信息
marketBuilder.AddConfig("key", "market_key", quoteParam)
marketBuilder.AddConfig("logo_white", "market_logo", quoteParam)
if params.Lan == "en" {
    marketBuilder.AddConfig("en_name", "market_name", quoteParam)
} else {
    marketBuilder.AddConfig("cn_name", "market_name", quoteParam)
}
// 获取广告标题
marketBuilder.AddConfig(adTitleField, "ad_title", quoteParam)
// 获取广告标题(英文)
marketBuilder.AddConfig(adTitleEnField, "ad_title_en", quoteParam)
// 获取广告链接
marketBuilder.AddConfig(adUrlField, "ad_url", quoteParam)
// 获取广告标题(英文)
marketBuilder.AddConfig(adTitleEnField, "ad_title_en", quoteParam)
// 获取标签
marketBuilder.AddConfig(labels, "labels", quoteParam)
// 获取注册链接
marketBuilder.AddConfig("register_link", "registerLink", quoteParam)
// 获取url
marketBuilder.AddConfig("url", "url", quoteParam)
// 获取dex网络
marketBuilder.AddConfig("dex_network", "dex_network", quoteParam)
// 使用GetResult获取市场数据
marketConfig := marketBuilder.BaseBuilder.GetResult()
// 取具体字段
marketConfig["market_key"]
marketConfig["market_logo"]

```

### 3.3 CoinBuilder 构建器
获取货币数据示例代码,具体使用时，需要根据实际情况，添加对应的配置
```go
import (
	"team.aicoin.net.cn/lab/sosobtc-backend/go-core-lib/pkg/quote"
)
// 获取请求体
bodyBytes := httputil.GetRequestBodyFromValues(ctx)
// 解析请求体,得到quoteParam
quoteParam := tools.ParseGolbalParams(bodyBytes)
// 使用quote.NewCoinBuilder(coinList)获取货币数据,其中参数为切片类型，表示dbkeys
coinBuilder := quote.NewCoinBuilder(coinList)
// 添加配置，addconfig的第一个参数为字符串类型，表示要获取的配置字段名,第二个参数为字符串类型，表示输出结果中的字段名,第三个参数为quoteParam类型，包含默认值等信息
coinBuilder.AddConfig("coin_show", "coin_show", quoteParam)
// 获取货币logo
coinBuilder.AddConfig("logo_142", "logo", quoteParam)
// 获取货币唯一key
coinBuilder.AddConfig("unique_key", "unique_key", quoteParam)
// 获取货币24小时交易量
coinBuilder.AddDegree(config.TICKER_CYCCLE_24H, "degree24H", quoteParam)
// 获取货币24小时交易量
coinBuilder.AddTrade(config.TICKER_CYCCLE_24H, "trade24H", quoteParam)
// 设置数据过期时间
coinBuilder.BaseBuilder.DataExpireTime = int(time.Now().Unix())
// 获取货币最后一条数据
coinBuilder.AddLast("last", quoteParam)
// 使用GetResult获取货币数据
coinConfigMap := coinBuilder.BaseBuilder.GetResult()
// 取具体字段
coinConfigMap["logo"]
coinConfigMap["unique_key"]
coinConfigMap["degree24H"]
coinConfigMap["trade24H"]

```

### 3.4 IndexBuilder 构建器
获取指数数据示例代码,具体使用时，需要根据实际情况，添加对应的配置
```go
import (
	"team.aicoin.net.cn/lab/sosobtc-backend/go-core-lib/pkg/quote"
)
// 获取请求体
bodyBytes := httputil.GetRequestBodyFromValues(ctx)
// 解析请求体,得到quoteParam
quoteParam := tools.ParseGolbalParams(bodyBytes)
// 使用quote.NewIndexBuilder(indexKeys)获取指数数据,其中参数为切片类型，表示指数列表
indexData := quote.NewIndexBuilder(indexKeys).
    // 添加配置，addconfig的第一个参数为字符串类型，表示要获取的配置字段名,第二个参数为字符串类型，表示输出结果中的字段名,第三个参数为quoteParam类型，包含默认值等信息
    // 获取指数key
    AddConfig("key", "key", quoteParam).
    // 获取指数小数位
    AddConfig("decimal", "decimal", quoteParam).
    // 获取指数中文名称
    AddConfig("cn_name", "internal_cn_name", quoteParam).
    // 获取指数英文名称
    AddConfig("en_name", "internal_en_name", quoteParam).
    // 获取指数最新价格
    AddLast("priceCny", quoteParam).
    // 获取指数24小时交易量
    AddDegree(config.TICKER_CYCCLE_24H, "deg24HCny", quoteParam).
    // 获取指数日交易量
    AddDegree(config.TICKER_CYCCLE_DAY, "degree", quoteParam).
    // 获取指数24小时交易量变化
    AddChangeValue(config.TICKER_CYCCLE_24H, "chg24HCny", quoteParam).
    // 获取指数日交易量变化
    AddChangeValue(config.TICKER_CYCCLE_DAY, "change", quoteParam).
    // 使用GetResult获取指数数据
    BaseBuilder.GetResult()
// 取具体字段
indexData["key"]
indexData["decimal"]

```
## 4. 数据库操作

### 4.1 mysql 数据库相关用法

#### 4.1.1 mysql 查询示例
mysql数据库使用gorm的用法进行进行查询
```go
// 声明单条记录结构体，用于获取表名
var record model.KlineShareLikes
// 声明结果集切片，用于存储查询结果
var recordList []model.KlineShareLikes

// 获取MySQL数据库连接
mysqlConn := db.GetMysqlConn(db.MYSQL_DB_AICOIN)

// 构建基础查询
// mysqlConn.Table(record.TableName())获取具体表名
KlineShareQuery := mysqlConn.Table(record.TableName()).
    Where(model.KlineShareLikesColumns.ShareKey+" IN ?", shareKey).
    Where(model.KlineShareLikesColumns.State+" = ?", 1)

// 可选的时间过滤条件
// 如果filterTime不为0，添加创建时间大于filterTime的条件
if filterTime != 0 {
    KlineShareQuery = KlineShareQuery.Where(model.KlineShareLikesColumns.Createtime+" > ?", filterTime)
}

// 执行查询并获取结果
// Find(): 执行查询并将结果解析到recordList切片中
KlineShareQueryResult := KlineShareQuery.Find(&recordList)

// 错误处理
// 如果查询出错，记录警告日志并返回错误
if KlineShareQueryResult.Error != nil {
    logger.Warn(KlineShareQueryResult.Error.Error())
    return recordList, KlineShareQueryResult.Error
}
```

#### 4.1.2 mysql 定义model示例
```go
// KlineShareLikes [...]
type KlineShareLikes struct {
	ID         int    `gorm:"primaryKey;column:id" json:"-"`
	UserID     int    `gorm:"column:user_id" json:"userId"`        // 用户id
	ShareKey   string `gorm:"column:share_key" json:"shareKey"`    // 分享K线主键
	Createtime int    `gorm:"column:createtime" json:"createtime"` // 创建时间
	Updatetime int    `gorm:"column:updatetime" json:"updatetime"` // 更新时间
	State      int8   `gorm:"column:state" json:"state"`           // 记录状态
}

// TableName get sql table name.获取数据库表名
func (m *KlineShareLikes) TableName() string {
	if tools.EnvIsDev() {
		return "kline_share_likes_test"
	}
	return "kline_share_likes"
}
```
### 4.2 redis 数据库相关用法
#### 4.2.1 redis 查询示例
redis数据库使用redis的用法进行进行查询
```go
// 获取redis数据库连接
var masterRedis = db.GetRedisConn("result")
// 初始化关注者数量
twitterFollowers := 0

// 构建用户信息的Redis键
twitterKey := "flash_member_info:" + memberId

// 从用户信息hash表中获取关注者数量
twitterInfo, err := masterRedis.HGet(
    timeoutContext, 
    twitterKey, 
    "followersCount"
).Result()

if err != nil && err != redis.Nil {
    logger.Error(err.Error())
}
```
### 4.3 mongo 数据库相关用法
#### 4.3.1 mongo 查询示例
mongo数据库使用bson的用法进行进行查询
```go
// 获取MongoDB连接
mongoConn := db.GetMongoConn()

// 创建带超时的上下文
mongoTimeoutContext, cancelFunc := libTools.NewTimeoutContext(libTools.GetDefaultTimeout())
defer cancelFunc()  // 确保函数结束时取消上下文

// 构建查询条件
filter := bson.D{
    // 状态为1的记录
    {Key: mongoModel.NewsFlashColumns.State, Value: 1},
    // cmid在指定数组中的记录
    {Key: mongoModel.NewsFlashColumns.Cmid, Value: bson.M{"$in": cmids}},
}

// 处理字段投影（选择返回哪些字段）
var formatProjection interface{}
if len(fields) > 0 {
    // 如果指定了字段，构建投影
    projection := make(bson.D, len(fields))
    for index, field := range fields {
        projection[index] = bson.E{Key: field, Value: 1}  // 1表示返回该字段
    }
    formatProjection = projection
} else {
    // 没有指定字段，返回所有字段
    formatProjection = nil
}

flashDataList := make([]mongoModel.NewsFlash, 0, len(cmids))

// 获取集合引用
flashCollection := mongoConn.Database("News").Collection("HotNews")

// 执行查询
flashResult, flashResultErr := flashCollection.Find(
    mongoTimeoutContext, 
    filter, 
    &options.FindOptions{
        Projection: formatProjection,           // 字段投影
        Sort: bson.D{{Key: "crawled", Value: -1}},  // 按crawled字段降序排序
    },
)

// 处理查询错误
if flashResultErr != nil && flashResultErr != mongo.ErrNoDocuments {
    logger.Error("FindFlashByCmids Find", 
        zap.Any("err", flashResultErr.Error()))
    return flashDataList, flashResultErr
}

// 将查询结果解码到切片中
err := flashResult.All(mongoTimeoutContext, &flashDataList)
if err != nil {
    logger.Error("FindFlashByCmids All", 
        zap.Any("err", err.Error()))
    return flashDataList, err
}
```
#### 4.3.2 获取集合引用
获取集合引用使用有两种用法:
1. 通过数据库连接获取集合引用
```go
// 获取集合引用
flashCollection := mongoConn.Database("News").Collection("HotNews")

```
2. 通过项目定义的model结构体获取集合引用
```go
import (
    "aicoin/pc-api/internal/model"
)
// 获取集合引用
backtestStatus := model.BacktestStatus{}

collection := mongodbConn.Database(backtestStatus.DatabaseName()).Collection(backtestStatus.TableName())
```
#### 4.3.3 mongo 定义model示例
```go
type BacktestStatus struct {
	UserID            int64   `bson:"user_id" json:"user_id"`                         // 用户id
	UniqueID          string  `bson:"unique_id" json:"unique_id"`                     // 唯一id，必须
	Symbol            string  `bson:"symbol" json:"symbol"`                           // 交易对key，必须
	StartCash         float64 `bson:"start_cash" json:"start_cash"`                   // 初始金额，必须
	FeeRate           float64 `bson:"fee_rate" json:"fee_rate"`                       // 手续费率
    // 其他字段
}
// 获取数据库名称
func (t BacktestStatus) DatabaseName() string {
	return "quotation"
}
// 获取表名称
func (t BacktestStatus) TableName() string {
	if tools.EnvIsDev() {
		return "backtest_status_test"
	} else {
		return "backtest_status"
	}
}
```
## 5. zlog日志处理
#### 5.1 代码中获取zlog
```go
import (
    "aicoin/pc-api/internal/infrastructure/zlog"
    "go.uber.org/zap"
)
```
#### 5.2 zlog.Error 方法
```go
// Error 封装 zap log Error 方法
// 接收msg和fields，其中fields是可选的，可以传递多个字段，msg是必填的日志信息
func Error(msg string, fields ...zap.Field) {
	zLog.Error(msg, fields...)
}
```

#### 5.3 简单错误记录

1. 对于简单错误，优先使用 zap.Error 字段而不是格式化字符串：
```go
// 推荐
zlog.Error("operation failed", zap.Error(err))

// ⚠️ 不推荐
zlog.Error(fmt.Sprintf("operation failed: %v", err))
```

2. 记录错误时应包含足够的上下文信息：
```go
zlog.Error("database operation failed",
    zap.Error(err),
    zap.String("module", "cache"),
    zap.String("function", "GetMultiShareLikesCountByCache"),
    zap.Any("params", params))
```

3. 对于特定错误类型的处理：
```go
if err != nil && err != mongo.ErrNilDocument {
    zlog.Error("database query failed",
        zap.Error(err),
        zap.String("collection", collectionName),
        zap.Any("query", query))
}
```
#### 5.4 其他zlog方法,与zlog.Error一致
```go
// Warn 封装 zap log Warn 方法
func Warn(msg string, fields ...zap.Field) {
	zLog.Warn(msg, fields...)
}

// Info 封装 zap log Info 方法
func Info(msg string, fields ...zap.Field) {
	zLog.Info(msg, fields...)
}

// Debug 封装 zap log Debug 方法
func Debug(msg string, fields ...zap.Field) {
	zLog.Debug(msg, fields...)
}

// Panic 封装 zap log Panic 方法
func Panic(msg string, fields ...zap.Field) {
	zLog.Panic(msg, fields...)
}
```

使用示例:
```go
// 记录信息
zlog.Info("info message", zap.String("key", "value"))

// 记录警告
zlog.Warn("warning message", zap.String("key", "value"))

zlog.Warn("ews exceed limit",
    zap.Any("ewsAmount", ewsAmount),
    zap.Any("numUid", numUid),
    zap.Any("module", module),
    zap.Any("beta", param.Beta),
    zap.Any("limit", limit),
)

```

## 6. 完整接口示例

将以上所有组件组合使用的完整示例:

```go
// controller/alert_controller.go
func GetWarningList(ctx iris.Context) {
    // 初始化响应
	var res httputil.Response
	res.Agent = httputil.PCAPIAgent
	// 获取请求体
	bodyBytes := httputil.GetRequestBodyFromValues(ctx)
	// 解析全局参数
	quoteParam := tools.ParseGolbalParams(bodyBytes)
	// 获取会话数据
	sessionData := httputil.GetSessionByCtx(ctx)
	// 参数验证
	type Params struct {
		Type         int    `json:"type" `
		LastID       string `json:"last_id"`
		DBKey        string `json:"db_key"`
		LastTime     string `json:"last_time"`
		Size         int    `json:"size"`
		FilterUpDown int    `json:"filterUpDown"`
		Lan          string `json:"lan"`
	}

	var params Params

	// 获取请求参数
	if err := httputil.GetRequestParams(ctx, &params); err != nil {
		httputil.NewResponse().JSONFailedNoData(ctx, err.Error(), 304)
		ctx.Next()
		return
	}
	if params.Lan == "" {
		params.Lan = "cn"
	}
	if params.Size == 0 {
		params.Size = 20
	}

    
	// 根据会话数据获取用户ID
	uid, err := strconv.Atoi(sessionData.UserId)
	if err != nil {
		zlog.Error("GetWarningList", zap.Error(err))
	}
    // 使用模型结构体
	mod := &locModel.EwsRecord{}
	var list []locModel.EwsRecord

	// 获取数据库连接
	mysqlConn := db.GetMysqlConn(db.MYSQL_DB_AICOIN)
	beta := 0
	// 构建基础查询，强制使用索引
	query := mysqlConn.Table(mod.TableName(beta)+" FORCE INDEX(idx_uid_style_createtime)").
		Where("uid = ?", uid).
		Where("price_type IN ?", []string{"last", "buy", "sell"}).
		Where("style = ?", 1).
		Where("mode IN ?", []string{"1", "2", "3"})

	// 设置限制
	if params.FilterUpDown == 0 {
		query = query.Limit(params.Size)
	}

	// 执行查询
	err = query.Find(&list).Error
	if err != nil && err != gorm.ErrRecordNotFound {
        // 记录错误
		zlog.Error("GetWarningList", zap.Error(err))
		// 不返回错误，继续执行
	}

	// 收集交易对列表
	tradingList := make([]string, 0)
	for _, record := range list {
		if !tools.InArray(record.DbKey, tradingList) {
			tradingList = append(tradingList, record.DbKey)
		}
	}

	// 准备返回数据
	body := make([]map[string]interface{}, 0)
	var lastID, lastTime string

    // 使用builder模式组装交易对信息
	if len(tradingList) > 0 {
		// 获取交易对详情
		tpBuilder := quote.NewTPBuilder(tradingList)
		tpBuilder.BaseBuilder.IsFilterOffline = false
		tpBuilder.AddConfig("currency_str", "currency", quoteParam)
		tpBuilder.AddConfig("coin_show", "show", quoteParam)
		tpBuilder.AddCurrencySymbol("symbol", quoteParam)
		tpBuilder.AddConfig("market_cn", "market_cn", quoteParam)
		tpBuilder.AddConfig("market_en", "market_en", quoteParam)
		tradingDetail := tpBuilder.BaseBuilder.GetResult()
		// 组装返回数据
        // 省略
    }

	// 获取预警总数
	warningCount := strconv.FormatInt(service.GetAlertCount(uid, ""), 10)

	// 返回成功响应
	httputil.ResponseData(ctx, map[string]interface{}{
		"body":      body,
		"last_id":   lastID,
		"last_time": lastTime,
		"ews_count": warningCount,
	})

}
```

