---
title: "go项目初始化"
date: 2023-10-24T19:38:37+08:00
author: ["loveyu"]
draft: false
categories: 
- go
tags: 
- go
---

# 服务优雅关闭启动

>1. 使用`signal.NotifyContext`接收退出相关信号,获取到ctx
>2. 传递ctx到初始化api中
>3. 初始化api
>   1. 先获取到项目api handler
>   2. 创建chan空结构体管道
>   3. 启动网络监听
>   4. 开启goroutine
>      1. 打印输出
>      2. 向chan发送数据
>      3. 运行server
>   5. 接收chan信号
>   6. 开启goroutine
>      1. 打印输出
>      2. 向chan发送数据
>      3. 拦截ctx的done信号
>      4. 关闭server
>   7. 接收chan信号
>4. 接收ctx的done信号
>5. 设置content.Timeout三秒关闭

```go
func main() {
	ctx, cancel := signal.NotifyContext(context.Background(), syscall.SIGHUP, syscall.SIGINT, syscall.SIGTERM, syscall.SIGQUIT)
	defer cancel()

	opt.MustInitConfig()

	api.MustStartApi(ctx)

	<-ctx.Done()

	logrus.Warn("[gateway] service stopped by gctx in 3s...")
	<-util.Timeout(3).Done()
}

func initApp() http.Handler {
    engine := gin.Default()
    ....
    return engine.Handler()
}


func initApp() *fiber.App {
    engine := fiber.New(fiber.Config{BodyLimit: 200 * 1024 * 1024})
    ....
    return engine.Handler()
}

func MustStartApi(ctx context.Context) {
	var (
		err error
		ln  net.Listener
        // gin是在go的http上进行包装的框架所以可以直接使用http.Server来接受运行
		srv = http.Server{
			Handler: initApp(),
		}
        
        // fiber使用fasthttp所以需要使用fasthttp.Server来接收
        srv = fasthttp.Server{
			Handler:   initApp(),
			TLSConfig: nil,
		}
		address = fmt.Sprintf("%s:%d", opt.Cfg.Gateway.ListenHost, opt.Cfg.Gateway.ListenPort)
		ready   = make(chan struct{})
	)

	if ln, err = net.Listen("tcp", address); err != nil {
		logrus.Fatalf("[%s] MustStart: listen address: %s err: %v", opt.Cfg.Gateway.Name, address, err)
	}

	go func() {
		logrus.Infof("[%s] MustStart: service serve at: %s", opt.Cfg.Gateway.Name, address)
		ready <- struct{}{}
		if err = srv.Serve(ln); err != nil {
			logrus.Fatalf("[%s] MustStart: service serve err: %v", opt.Cfg.Gateway.Name, err)
		}
	}()

	<-ready

	go func() {
		ready <- struct{}{}
		<-ctx.Done()
		logrus.Infof("[%s] MustStart: service shutdown by gctx", opt.Cfg.Gateway.Name)
		if err = srv.Shutdown(tools.Timeout(3)); err != nil {
			logrus.Errorf("[%s] MustStart: service shutdown by gctx with err: %v", opt.Cfg.Gateway.Name, err)
		}
	}()

	<-ready
}


```

# 项目配置初始化

>1. 必须在`init`里面进行参数解析
>
>   1. ```go
>      flag.StringVar(&configFile, "c", "etc/config.json", "")
>      flag.IntVar(&Debug, "debug", 0, "")
>      ```
>
>2. 时区设置
>
>   1. ```go
>      time.Local = time.FixedZone("CST", 8*3600)
>      	common.GMT = time.FixedZone("GMT", 0)
>      ```
>
>3. 在初始化函数里面设置日志等级
>
>   1. ```go
>      logrus.SetLevel
>      ```
>
>      

```go
type config struct {
	Project        string     `mapstructure:"project"`
	Redis          redis      `mapstructure:"redis"`
	ES             es         `mapstructure:"es"`
	Mysql          mysql      `mapstructure:"mysql"`
	User           service    `mapstructure:"user"`
}

var (
	configFile string
	Cfg        = new(config)
	Debug      int
)

func init() {
	time.Local = time.FixedZone("CST", 8*3600)
	common.GMT = time.FixedZone("GMT", 0)
	flag.StringVar(&configFile, "c", "etc/config.json", "")
	flag.IntVar(&Debug, "debug", 0, "")
}

func MustInitConfig() {
	flag.Parse()

	var (
		err error
	)

	viper.SetConfigType("json")
	viper.SetConfigFile(configFile)
	if err = viper.ReadInConfig(); err != nil {
		logrus.Panicf("read in config file err: %v", err)
	}

	if err = viper.Unmarshal(Cfg); err != nil {
		logrus.Panicf("unmarshal config file err: %v", err)
	}

	switch Debug {
	case 1:
		logrus.SetLevel(logrus.DebugLevel)
	case 9999:
		logrus.SetLevel(logrus.TraceLevel)
		logrus.SetReportCaller(true)
		logrus.SetFormatter(&logrus.TextFormatter{
			ForceColors: true,
			CallerPrettyfier: func(frame *runtime.Frame) (string, string) {
				file := path.Base(frame.File)
				return "", " " + file + ":" + strconv.Itoa(frame.Line)
			},
		})
	}

	logrus.Infof("read in config detail: %+v", *Cfg)
}

```

# label标签

>数据包装

```go
type PrivilegeTimeType string

const (
	PrivilegeTimeTypeInf   PrivilegeTimeType = "inf"
	PrivilegeTimeTypeRange PrivilegeTimeType = "range"
)

func (pt PrivilegeTimeType) Label() string {
	switch pt {
	case PrivilegeTimeTypeInf:
		return "无限制"
	case PrivilegeTimeTypeRange:
		return "时间区间"
	default:
		return "未知"
	}
}
```

```go
type PrivilegeType int

const (
	PrivilegeTypeRead PrivilegeType = 1 << (iota + 1)
	PrivilegeTypeWrite
	PrivilegeTypeOwn
	PrivilegeTypeCharge
)

var (
	AllPrivilegeTypes = []PrivilegeType{
		PrivilegeTypeRead,
		PrivilegeTypeWrite,
		PrivilegeTypeOwn,
	}
	AllPrivilegeTimeTypes = []PrivilegeTimeType{
		PrivilegeTimeTypeInf,
		PrivilegeTimeTypeRange,
	}
)

func (pt PrivilegeType) Label() string {
	switch pt {
	case PrivilegeTypeRead:
		return "只读"
	case PrivilegeTypeWrite:
		return "读写"
	case PrivilegeTypeOwn:
		return "创建者"
	default:
		return "未知"
	}
}

type ReadablePrivilege struct {
	Value PrivilegeType `json:"value"`
	Label string        `json:"label"`
}
```

# model

>1. 在model里面的``只写公用的``,例如user的go文件里面账号密码登录这样的结构体就不要写而是在登录的handler里面单独声明出来
>2. 结构体的方法`只写公用的`,其他例如获取用户信息之类的可以放到control的user里面去写
>3. 例如用户的token获取,权限相关例如用户是否有相关权限这类的公用方法可以写到model的user中

# handler

>1. 每个handler里面``单独声明出需要的结构体``而不是把这些结构体放到model或者其他地方
>2. 需要使用的变量提前使用vars声明出来
>3. 错误的返回放到msg而不是data
>4. 只有在这个handler里面用到的数据库查询就直接在handler里面写就可以不要放到model或者其他地方
>5. 如果需要用循环处理且循环里面内容特别多的可以吧这些内容单独拿出来作为一个函数放到handler文件里面,然后在循环调用这个函数

# 数据库数据初始化

>1. 在model里面写一个init的go文件配置需要初始化的数据
>2. 声明一个大的`MustInit`供`main`函数调用
>3. 在`MustInit`里面调用每一个需要初始化的`model`
>4. 如果有默认必须得数据库信息则存放到对应的model里面
>5. 在`MustInit`同级写初始化函数
>   1. 先`AutoMigrate`创建出数据库表
>   2. 查询出当前表有多少条数据
>   3. 判断数据库存在的数据条数和需要的默认数据条数是否相同
>   4. 如果不相同则循环默认数据进行创建
>      1. 记得判断错误类型是否为已存在, `if !strings.Contains(err.Error(), "SQLSTATE 23505") {}` 
>   5. 如果是pgsql要把序列重置为当前的数据库数据条数
>      1. `SELECT setval('users_id_seq', (SELECT MAX(id) FROM users))` 
>6. 如果有需要的数据库函数可以写好调用进行创建

```go
func MustInit(db *gorm.DB) {
	mustInitRoles(db)
	mustInitUsers(db)
    mustInitFunctions(db)
	....
}
func mustInitUsers(db *gorm.DB) {
	....
}
....
func mustInitFunctions(db *gorm.DB) {
	var (
		err error
		sql = `CREATE FUNCTION array_intersect(anyarray, anyarray)
  RETURNS anyarray
  language sql
as $FUNCTION$
    SELECT ARRAY(
        SELECT UNNEST($1)
        INTERSECT
        SELECT UNNEST($2)
    );
$FUNCTION$;`
	)

	if err = db.Exec(sql).Error; err != nil {
		if !strings.Contains(err.Error(), "SQLSTATE 42723") {
			panic(err)
		}
	}
}
```

