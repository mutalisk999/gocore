<a href="https://echo.labstack.com"><img height="80" src="https://cdn.labstack.com/images/echo-logo.svg"></a>

# GoCore Integrated Development Framework


SUNMI Go language development core library, aggregation configuration center, configuration management, database, service grid, database,
All common components such as encryption and decryption are considered and packaged uniformly.
Form a complete development framework to improve R&D efficiency;

SUNMI Go语言开发核心库，聚合配置中心、配置管理、数据库、服务网格、数据库、
加解密等各种常用组件进行统一考虑、统一封装，
形成一套完整开发框架来提升研发效率；

---

## Installation

```go

go get github.com/sunmi-OS/gocore


import (
...
	"github.com/sunmi-OS/gocore/xxxxx"
...
)

```


### Supported Go versions

- Golang > 1.13
- [Go module](https://github.com/golang/go/wiki/Modules)

---

## Examples

### API Service 

```go
	// Echo instance
	e := echo.New()

	// Middleware
	e.Use(middleware.Logger())
	// panic custom returns error content
	// Echo default {"message":"Internal Server Error"}
	e.Use(coreMiddleware.Recover(`{"code":-1,"data":null,"msg":"Service is abnormal, please try again later"}`))

	// Route => handler
	e.POST("/", func(c echo.Context) error {

		request := api.NewRequest(c)
		response := api.NewResponse(c)

		err := request.InitRawJson()
		if err != nil {
			return response.RetError(err, 400)
		}

		msg := request.Param(`test`).GetString()

		return response.RetSuccess(msg)
	})

	// Start server
	e.Logger.Fatal(e.Start(":1323"))
```

### Type Conversion

```go

import (
	"fmt"
	"github.com/spf13/cast"
...
	var i64 int64
	i64 = 60

	toString(cast.ToString(i64))
```

### CronJob

```go

import (
	"fmt"
	"github.com/robfig/cron"

...

	c := cron.New()
	c.AddFunc("0 30 * * * *", func() { fmt.Println("Every hour on the half hour") })
	c.AddFunc("0 * * * * *", func() { fmt.Println("Every minutes") })
	c.AddFunc("@hourly", func() { fmt.Println("Every hour") })
	c.AddFunc("@every 1h30m", func() { fmt.Println("Every hour thirty") })

	// Synchronous blocking
	c.Run()

	// Run asynchronously
	//c.Start()
```

### Encryption

AES:

```go

import (
	"fmt"

	"github.com/sunmi-OS/gocore/encryption/aes"

...

	str, _ := aes.AesEncrypt("sunmi", "sunmiWorkOnesunmiWorkOne")
	fmt.Println(str)
	str2, _ := aes.AesDecrypt(str, "sunmiWorkOnesunmiWorkOne")
	fmt.Println(str2)

```

DES:

```go

import (
	"fmt"

	"github.com/sunmi-OS/gocore/encryption/des"
)

...
	str, _ := des.DesEncrypt("sunmi", "sunmi388", "12345678")
	fmt.Println(str)
	str2, _ := des.DesDecrypt(str, "sunmi388", "12345678")
	fmt.Println(str2)

	str, _ = des.DesEncryptECB("sunmi", "sunmi388")
	fmt.Println(str)
	str2, _ = des.DesDecryptECB(str, "sunmi388")
	fmt.Println(str2)

	str, _ = des.TripleDesEncrypt("sunmi", "sunmi388sunmi388sunmi388", "12345678")
	fmt.Println(str)
	str2, _ = des.TripleDesDecrypt(str, "sunmi388sunmi388sunmi388", "12345678")
	fmt.Println(str2)

```

RSA:

```go

import (
	"errors"
	"log"

	"github.com/sunmi-OS/gocore/encryption/gorsa"
)

var Pubkey = `-----BEGIN public-----
...
-----END public-----
`

var Pirvatekey = `-----BEGIN private-----
...
-----END private-----
`

func main() {
	// Public key encryption and private key decryption
	if err := applyPubEPriD(); err != nil {
		log.Println(err)
	}
	// Private key encryption, public key decryption
	if err := applyPriEPubD(); err != nil {
		log.Println(err)
	}
}

// Public key encryption and private key decryption
func applyPubEPriD() error {
	pubenctypt, err := gorsa.PublicEncrypt(`hello world`, Pubkey)
	if err != nil {
		return err
	}

	pridecrypt, err := gorsa.PriKeyDecrypt(pubenctypt, Pirvatekey)
	if err != nil {
		return err
	}
	if string(pridecrypt) != `hello world` {
		return errors.New(`Decryption failed`)
	}
	return nil
}

// Private key encryption, public key decryption
func applyPriEPubD() error {
	prienctypt, err := gorsa.PriKeyEncrypt(`hello world`, Pirvatekey)
	if err != nil {
		return err
	}

	pubdecrypt, err := gorsa.PublicDecrypt(prienctypt, Pubkey)
	if err != nil {
		return err
	}
	if string(pubdecrypt) != `hello world` {
		return errors.New(`Decryption failed`)
	}
	return nil
}


```

### GoMail


Config:

```toml
[email]
host="smtp.exmail.qq.com"
port=465
username="test@sunmi.com"
password="password"

```

```go

import (
	"github.com/sunmi-OS/gocore/gomail"
	"github.com/sunmi-OS/gocore/viper"
)

func main() {

	viper.NewConfig("config", "conf")

	gomail.SendEmail("wenzhenxi@vip.qq.com", "service@sunmi.com", "service", "title", "msg")
}


```


### Grpc


```go

var Rpc *rpcx.DirectClient

func Init(host string, timeout int64, opts ...rpcx.ClientOption) {

	var err error
	Rpc, err = rpcx.NewDirectClient(host, timeout, opts...)
	if err != nil {
		log.Fatal("rpc connect fail")
	}
}

func PrintOk(in *Request) (*Response, error) {
	conn, ok := Rpc.Next()
	if !ok || conn == nil {
		return nil, errors.New("rpc connect fail")
	}
	client := NewPrintServiceClient(conn)

	return client.PrintOK(context.Background(), in)
}

...


resp, err := printpb.PrintOk(req, trace)

```



### Istio

```go
func PrintOk(in *Request, trace istio.TraceHeader) (*Response, error) {
	conn, ok := Rpc.Next()
	if !ok || conn == nil {
		return nil, errors.New("rpc connect fail")
	}
	client := NewPrintServiceClient(conn)

	ctx := metadata.NewOutgoingContext(context.Background(), trace.Grpc_MD)
	return client.PrintOK(ctx, in)
}
...

    trace := istio.SetHttp(c.Request().Header)
    req := &printpb.Request{
		Message: "test",
	}
	resp, err := printpb.PrintOk(req, trace)
	if err != nil {
		log.Sugar.Errorf("请求失败: %s", err.Error())
		return err
	}
```


### HttpLib

```go

	b := httplib.Post("https://baidu.com/")
	b.Param("username", "astaxie")
	b.Param("password", "123456")
	b.PostFile("uploadfile1", "httplib.pdf")
	b.PostFile("uploadfile2", "httplib.txt")
	str, err := b.String()
	if err != nil {
		fmt.Println(err)
	}
	fmt.Println(str)

```

### Logs

```go

	log.InitLogger("example-log")

...

	log.Sugar.Debugw("example-log:debug")
	log.Sugar.Infow("example-log:info", zap.String("type", "log"))
	log.Sugar.Errorw("example-log:err", zap.Error(errors.New("IS ERROR")))

```



### Nacos

```go

func InitNacos(runtime string) {
	nacos.SetRunTime(runtime)
	nacos.ViperTomlHarder.SetviperBase(baseConfig)
	switch runtime {
	case "local":
		nacos.AddLocalConfig(runtime, localConfig)
	default:
		Endpoint := os.Getenv("ENDPOINT")
		NamespaceId := os.Getenv("NAMESPACE_ID")
		AccessKey := os.Getenv("ACCESS_KEY")
		SecretKey := os.Getenv("SECRET_KEY")
		if Endpoint == "" || NamespaceId == "" || AccessKey == "" || SecretKey == "" {
			panic("The configuration file cannot be empty.")
		}
		err := nacos.AddAcmConfig(runtime, constant.ClientConfig{
			Endpoint:    Endpoint,
			NamespaceId: NamespaceId,
			AccessKey:   AccessKey,
			SecretKey:   SecretKey,
		})
		if err != nil {
			panic(err)
		}
	}
}

...

	config.InitNacos("local")

	nacos.ViperTomlHarder.SetDataIds("DEFAULT_GROUP", "adb")
	nacos.ViperTomlHarder.SetDataIds("pay", "test")

	nacos.ViperTomlHarder.SetCallBackFunc("DEFAULT_GROUP", "adb", func(namespace, group, dataId, data string) {

		err := gorm.UpdateDB("remotemanageDB")
		if err != nil {
			fmt.Println(err.Error())
		}
	})

	nacos.ViperTomlHarder.NacosToViper()

```



### Redis


```toml
[redisServer]
host = "host"
port = ":6379"
auth = "password"
prefix = "tob_"
encryption = 0

[redisDB]
e_invoice = 34

[OtherRedisServer]
host = "host"
port = ":6379"
auth = "password"
prefix = "tob_"
encryption = 0

[OtherRedisServer.redisDB]
e_invoice = 34


```

```go

	viper.NewConfig("config", "conf")
	redis.GetRedisOptions("e_invoice")
	redis.GetRedisDB("e_invoice").Set("test", "sunmi", 0)
	fmt.Println(redis.GetRedisDB("e_invoice").Get("test").String())

	redis.GetRedisOptions("OtherRedisServer.e_invoice")
	redis.GetRedisDB("OtherRedisServer.e_invoice").Set("test", "sunmi_other", 0)
	fmt.Println(redis.GetRedisDB("OtherRedisServer.e_invoice").Get("test").String())
	fmt.Println(redis.GetRedisDB("e_invoice").Get("test").String())

```

### Utils


file:
```go
	fmt.Println("GetPath:%s", utils.GetPath())
	fmt.Println(utils.IsDirExists("/tmp/go-build803419530/command-line-arguments/_obj/exe"))
	fmt.Println(utils.MkdirFile("test"))
```

gzip:
```go
	fmt.Println(utils.GzipEncode("dsxdjdhskfjkdsfhsdjlaal"))
	var m = utils.GzipEncode("dsxdjdhskfjkdsfhsdjlaal")
	fmt.Println(utils.GzipDecode(m))
```

random:
```go
	fmt.Println("RandInt", utils.RandInt(13, 233))

	fmt.Println("RandInt64", utils.RandInt64(13, 233))

	fmt.Println("GetRandomString", utils.GetRandomString(122))

	fmt.Println("GetRandomNumeral", utils.GetRandomNumeral(133))
```

sign:
```go
	var secret, params string
	secret = "123"
	params = "abdsjfhdshfksdhf"
	m, err := utils.GetParamMD5Sign(secret, params)
	if err != nil {
		fmt.Println("Error:", err)
	}
	fmt.Println("GetParamMD5Sign", m)

	var maintain string

	maintain = "dfssjfdsdfghjdsfgdsj"
	n, err := utils.GetSHA(maintain)
	if err != nil {
		fmt.Println("GetSHA failed error", err)
	}
	fmt.Println("GetSHA", n)

	l, err := utils.GetParamHmacSHA256Sign(secret, params)
	if err != nil {
		fmt.Println("GetParamHmacSHA256Sign failed err", err)
	}

	fmt.Println("GetParamHmacSHA256Sign", l)

	p, err := utils.GetParamHmacSHA512Sign(secret, params)
	if err != nil {
		fmt.Println("GetParamHmacSHA512Sign failed error", err)
	}
	fmt.Println("GetParamHmacSHA512Sign", p)

	u, err := utils.GetParamHmacSHA1Sign(secret, params)
	if err != nil {
		fmt.Println("GetParamHmacSHA1Sign failed error", err)
	}

	fmt.Println("GetParamHmacSHA1Sign", u)

	c, err := utils.GetParamHmacMD5Sign(secret, params)
	if err != nil {
		fmt.Println("GetParamHmacMD5Sign failed error", err)
	}

	fmt.Println("GetParamHmacMD5Sign", c)

	d, err := utils.GetParamHmacSha384Sign(secret, params)
	if err != nil {
		fmt.Println("GetParamHmacSha384Sign failed error", err)
	}

	fmt.Println("GetParamHmacSha384Sign", d)

	f, err := utils.GetParamHmacSHA256Base64Sign(secret, params)
	if err != nil {
		fmt.Println("GetParamHmacSHA256Base64Sign failed error", err)
	}

	fmt.Println("GetParamHmacSHA256Base64Sign", f)

	var hmac_key, hmac_data string
	hmac_key = "12322334234"
	hmac_data = "sjhdjsdjfh"
	t := utils.GetParamHmacSHA512Base64Sign(hmac_key, hmac_data)

	fmt.Println("GetParamHmacSHA512Base64Sign", t)
```

urlcode:
```go
	var urls string
	urls = "https://www.sunmi.com/"
	e, err := utils.UrlEncode(urls)
	if err != nil {
		fmt.Println("UrlEncode failed error", err)
	}

	fmt.Println("UrlEncode", e)

	r, err := utils.UrlDecode(urls)
	if err != nil {
		fmt.Println("UrlDecode failed error", err)
	}
	fmt.Println("UrlDecode", r)
```

utils:
```go
	d := utils.GetDate()
	fmt.Println("GetData", d)

	m := utils.GetRunTime()
	fmt.Println("GetRunTime", m)

	var encryption string
	encryption = "1243sdfds"

	t := utils.GetMD5(encryption)
	fmt.Println("GetMD5", t)
```


### Viper

```toml
[system]
port = ":10001"
OpenDES = false
DESkey = "deskey12"
DESiv = "12345678"
MD5key = "SUNMIMD5"
DESParam = "params"

[dbDefault]
dbHost = "xxxxx.com"     #数据库连接地址
dbName = "xxxxx"         #数据库名称
dbUser = "xxxxx"         #数据库用户名
dbPasswd = "xxxxx"       #数据库密码
dbPort = "3306"          #数据库端口号
dbOpenconns_max = 20     #最大连接数
dbIdleconns_max = 1      #最大空闲连接
dbType = "mysql"         #数据库类型

```


```go
	// 指定配置文件所在的目录和文件名称
	viper.NewConfig("config", "conf")
	// 打印读取的配置
	fmt.Println("port : ", viper.C.Get("system.port"))
	fmt.Println("ENV RUN_TIME : ", viper.GetEnvConfig("run.time"))
```





