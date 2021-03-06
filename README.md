# 联通物联网服务平台接口同步服务

> 联通平台网址： https://cc1.10646.cn/
>
>这个服务主要用于同步联通物联网卡流量相关的REST API 接口数据，便于本地化使用与分析
>
>

## 本地编译说明

1. 从git下载项目： git clone https://github.com/xinjiayu/SimServerUnicom
2. 安装mysql数据库，创建db，导入deploy下db.sql脚本
3. 修改config下config.example.toml为config.toml，并配置相关项

请跟据注释进行配置，包括服务相关配置，数据库配置，联通物联网平台相关配置
```toml
# 数据库配置
[database]
    [[database.default]]
        host = "127.0.0.1"

```
4. go run main.go


## 编译生成环境说明
可通过交叉编译，生成目标平台的可执行版本。目前在linux、mac os 测试通过，其它平台未做过测试。
将编译生成的文件放到bin目录下，将bin目录下的文件放到目标服务器，执行`./curl.sh start` 运行。

注意：编译生成的文件名必须是simserver_unicom ，如果是其它名称，请自行修改curl.sh脚本文件中的app变量。

```
curl.sh脚本参数：

start|stop|restart|status|tail


```


可以使用gox工具
`go get github.com/mitchellh/gox`

交叉编译：进入到mian函数文件所在目录下：

`gox -os "windows linux" -arch amd64`

`gox -os "linux" -arch amd64`



## 接口说明

### 数据采集接口

1、 CtdUsages 获取指定sim卡的流量接口，返回指定设备的周期累计用量信息。
调用地址：`/unicom/ctdusages`

2、CardList 通过联通平台的devices接口获取所有激活的sim卡流量数据
调用地址：`/unicom/cardlist`

>注：从联通平台获取到的数据是当前计费周期内的流量使用数据。单位为字节。
>这两个接口可以配合着定时任务使用，跟据需要定时拉取联通物联网平台的sim卡流量数据
>
>

### 数据分析输出接口

1、TwoDaysSimCardFlow 获取最近两天流量的sim卡列表
调用地址：`/unicom/analyse/twodayssimcardflow`



2、AllSimFlowList 获取计费周期内所有sim卡用量信息
调用地址：`/unicom/analyse/allsimflowlist`
参数：planName 计划名称,默可以为空，输出全部数据


3、PlanInfoCountList 获取计费套餐计划的统计信息
调用地址：`/unicom/analyse/planinfocountlist`

4、MonthSimFlowByIccid 获取指定sim卡最近两个月的流量
调用地址：`/unicom/analyse/monthsimflowbyiccid`
参数：iccid 必填

5、MonthSimFlowCount 获取所有sim卡最近两个月的流量
调用地址：`/unicom/analyse/monthsimflowcount`

### 远程操作接口

1、ChangePlan 跟据sim卡流量池使用情况进行自动设置平衡，平衡的顺序为1G池超出时，自动变更部分sim卡到2G池中，
//当2G池流量超出时自动将部分sim卡变更到3G池。当3G池也超出时将1G池中部分sim卡变更到3G池
调用地址：`/unicom/op/changeplan`

说明：直接调用，后台自动执行。

2、ChangeInitPlan 自动初始化所有sim卡的流量套餐变更为1G套餐
调用地址：`/unicom/op/changeinitplan`

说明：直接调用，后台自动执行。

3、Change19to20 将sim卡从19位转换为20位的接口
调用地址`/unicom/op/change19to20`
谳用参数为：iccid 不能为空。


### 接收通知接口
1、DataReceive 通知数据接收
调用地址：`/unicom/notice/datareceive`

参数：event_type，字符类型，默认为空，将显示本周期内所有的通知信息。当前事件类型支持：PAST24H_DATA_USAGE_EXCEEDED 24小时内流量超过指定量的通知、CTD_USAGE 周期使用超过指定的通知

## 感谢

1. gf框架 [https://github.com/gogf/gf](https://github.com/gogf/gf) 


# Jasper API 策略和最佳做法

Cisco Jasper 已经建立了一项 API 公平使用策略，确保所有 Control Center 用户都可以获得最佳的体验。任何使用 Control Center API 的客户端代码都必须遵守下述限制规定。

## 并发连接

Cisco Jasper 建议企业避免进行并发处理，因为它可能会显著降低响应速度。我们建议仅在单线程应用程序或例程中使用 API。在多线程情况下使用 API 的账户可能会受到限流或其他限制，具体视环境而定。

***策略*：** **使用单线程代码 - 一次一项 API 调用**。



## 每秒调用数

Cisco Jasper 通过限制账户每秒可以进行的 API 调用数来确保可靠的 API 性能。如果您的账户超出限制，则在测量期间(一秒)内的后续 API 调用将根据 API 类型获得下表中显示的响应。经过适当的延迟后，程序应该重试 API 调用。


| API 类型 | 返回代码 | 消息 |
| --- | --- | --- |
| REST | 40000029 | 超出资费限制 |
| SOAP | 400101 | 许可已超出 API 调用的资费限制 |


随着我们不断地调整优化平台以提高速度和可靠性，Cisco Jasper 可能会不时更新每秒调用数限制。如果您使用以下所述的最佳做法(单线程代码、动态速率限制等)，这项限制将不会产生影响。需要时，运营商可以提供有关这项限制的更多详细信息。

请注意，此限制同时包括账户和账户的客户进行的 API 调用。例如，如果某账户有两位 API 用户，他们及其 5 位客户同时进行了 API 调用， Control Center 会将该账户的每秒调用总数计为 7 个。最初的 API 调用会成功，但是在达到限制之后，其余的 API 调用将返回上面显示的其中一条消息。

这个限制也适用于 REST 和 SOAP API 调用的组合。例如，假设某个账户同时进行了 1 次 REST API 调用和 1 次 SOAP API 调用，同时该账户的其中一位客户进行了一次 REST API 调用。Control Center 会将该账户的每秒调用数计为 3 个。

**策略**。使用动态速率限制和其他最佳做法来遵守这项限制。



