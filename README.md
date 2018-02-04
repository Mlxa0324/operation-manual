# operation-manual

## 目录
* [常见问题](#常见问题)
* [快速开始](#快速开始)
    * [Android](https://github.com/Xiaomi-mimc/mimc-android-sdk)
    * [Web](https://github.com/Xiaomi-mimc/mimc-webjs-sdk)
    * [iOS](https://github.com/Xiaomi-mimc/mimc-ios-sdk)
* [如何接入](#如何接入)
* [安全认证](#安全认证)
* [推荐消息格式](#推荐消息格式)
    * [检查用户在线](#检查用户在线)
    * [文本消息](#文本消息)
    * [多媒体消息](#多媒体消息)
    * [撤回消息](#撤回消息)
    * [已读消息](#已读消息)
* [跨应用聊天](#跨应用聊天)
* [推送消息](#推送消息)
    * [推送单聊信息](#推送单聊信息)
    * [推送群聊信息](#推送群聊信息)
* [消息回调](#消息回调)
    * [即时消息回调](#即时消息回调)
    * [离线消息回调](#离线消息回调)
* [群聊消息](#群聊消息)
    * [创建群](#创建群)
    * [查询指定群信息](#查询指定群信息)
    * [查询所属群信息](#查询所属群信息)
    * [邀请用户加入群](#邀请用户加入群)
    * [非群主用户退群](#非群主用户退群)
    * [群主踢用户退群](#群主踢用户退群)
    * [群主更新群信息](#群主更新群信息)
    * [群主销毁群](#群主销毁群)
* [消息漫游](#消息漫游)
    * [拉取单聊消息记录](#拉取单聊消息记录)
    * [拉取群聊消息记录](#拉取群聊消息记录)
* [联系我们](#联系我们)

## 常见问题
#### 1)MIMC(小米即时消息云)收费吗？
```
消息服务免费使用，不收费（雷军：小米想赚钱不难 但永生永世不以利润为中心）
```

#### 2)MIMC适用于哪些应用场景？
```
MIMC适用于所有消息传递场景，不限于单聊/群聊/在线客服/私信/IoT信令传递等
```

#### 3)MIMC 为什么不提供聊天UI？
```
我们不提供聊天UI，是基于以下理由：
    1. APP都有自己的风格，万紫千红才是春，一套UI显然不能满足大家需求
    2. UI对于开发者而言，开发成本足够低，由开发者自行开发可100%自定义界面
所以，我们认为由开发者根据自己APP的风格来自定义UI比较合适
```

#### 4)开发者需要维护 用户帐号ID-->MIMC帐号ID 映射吗？
```
不需要，MIMC用户登录/消息收发等都使用APP自己帐号系统里的ID，MIMC帐号体系对APP开发者透明
```
###### 为什么这样做？
```
APP开发者接入其他IM提供商时，要访问IM提供商服务，主动为每一个appAccount注册一个新的ID，
开发者还需要在自己的后台系统储存以下信息：
    1. appAccount --> IM提供商系统内ID
    2. IM提供商系统内ID + IM提供商系统内登录密码(明文)
这样做有以下弊端：
    1. 开发者维护帐号映射成本高，一旦出错难以修正
    2. 明文存储登录密码，安全性极差，开发者承担极高的安全风险

所以，MIMC(小米即时消息云)没有采取以上方案，MIMC自维护帐号映射，保证MIMC ID对开发者透明
这不仅降低了开发者负担，增强了帐号安全性，还能让开发者感觉MIMC就是"自己的"消息系统
```
###### 举例说明：
```
    假设开发者拥有某APP叫"言士文学"，集成了MIMC(小米即时消息云)服务
    假设"言士文学"在小米开放平台注册的appId为"580012345678"
    某用户A登录"言士文学"所用帐号名为手机号"13800000001"
    某用户B登录"言士文学"所用帐号名为手机号"13800000002"

    当用户A首次登录"言士文学": 
        userA = new User("580012345678", "13800000001");
        userA.login();
	MIMC后台服务会为A创建MIMC ID("8049600000000001")，并维护以下映射：
            "580012345678" + "13800000001" --> "8049600000000001"
	    
    然后，用户A给用户B发消息: 
        userA.sendMessage("13800000002", "Hello B");
        由于B还未登录过"言士文学"，所以MIMC会自动为B创建创建MIMC ID("8049600000000002")
	并维护以下映射：
            "580012345678" + "13800000002" --> "8049600000000002"
	并将消息"Hello B"暂存在服务器(七天后过期)
	
    用户B登录"言士文学"时：
        userB = new User("580012345678", "13800000002");
        userB.login();
        因为B在MIMC已经有了ID，所以MIMC后台服务不会再为其分配新ID
        MIMC后台服务检测到B登录，下发离线消息"Hello B"给B
```

#### 5)既然MIMC帐号体系对APP开发者透明，那么APP如何接入MIMC帐号认证？
###### 请参阅：
* [如何接入](#如何接入)
* [安全认证](#安全认证)

#### 6)MIMC是否支持图片/语音/消息撤回/消息已读/红包等高级功能？
```
MIMC传递的消息格式，由APP开发者定义和解析，MIMC不理解消息的具体内容。
所以APP开发者可以基于MIMC实现任何高级功能，包括但不限于以上几种。
```
###### 高级消息功能实现可参考：
* [推荐消息格式](#推荐消息格式)
    * [检查用户在线](#检查用户在线)
    * [文本消息](#文本消息)
    * [多媒体消息](#多媒体消息)
    * [撤回消息](#撤回消息)
    * [已读消息](#已读消息)

## 快速开始

|应用平台 |描述 |备注 |
|:------:|:----:|:----:|
|Web|同一个用户允许同时登陆多个设备|无|
|iOS|同一个用户允许同时登陆多个设备|无|
|Android|同一个用户允许同时登录多个设备|无|

#### 1）[Android](https://github.com/Xiaomi-mimc/mimc-android-sdk)

#### 2）[Web](https://github.com/Xiaomi-mimc/mimc-webjs-sdk)

#### 3）[iOS](https://github.com/Xiaomi-mimc/mimc-ios-sdk)

## 如何接入

APP开发者访问小米开放平台（dev.mi.com）申请appId/appKey/appSec。
 
步骤如下：登录小米开放平台网页 -> ”管理控制台” -> ”小米应用商店” -> ”创建应用” ->  填入应用名和包名 -> ”创建” -> 记下看到的AppId/AppKey/AppSec 。
 
#### 备注1：建议MIMC与小米推送使用的APP信息一致
#### 备注2：安卓/iOS/Web用一个APP即可，不需要申请多个

## 安全认证

#### APP客户端获取Token的逻辑如下：
```
    APP <--> AppProxyService(APP开发者实现) <--> 小米TokenService(MIMC)
```
#### APP <--> AppProxyService(APP开发者实现)
######  安卓APP：
```
实现MIMCTokenFetcher：
   访问AppProxyService，从AppProxyService返回结果中获取[小米TokenService下发的原始数据]
```
###### iOS APP：
```
初始化NSMutableURLRequest：
    用于访问AppProxyService
实现delegate parseTokenDelegate：
    从AppProxyService返回结果中获取[小米TokenService下发的原始数据]
```
###### Web：
```
实现function fetchMIMCToken()：
   访问AppProxyService，从AppProxyService返回结果中获取[小米TokenService下发的原始数据]	
```
#### AppProxyService(APP开发者实现)需实现以下功能：
```
    1. 存储appId/appKey/appSec（不应存储在客户端，防止泄露）
    2. 用户在APP系统内的合法鉴权
    3. 调用小米TokenService服务，并将[小米TokenService下发的原始数据]返回客户端
```

#### AppProxyService访问小米TokenService的方式如下：
###### 参数列表

|   Variable          | Meanings  |
| :------------------ | :--------------------------------------------------------------|
|   $appId            |   小米开放平台申请的AppId                                       |
|   $appKey           |   小米开放平台申请的AppKey                                      |
|   $appSecret        |   小米开放平台申请的AppSecret	                               |
|   $appPackage       |   小米开放平台申请的AppPackage                                  |
|   $appAccount       |   用户在APP帐号系统内唯一ID                                     |
|   $chid             |   MIMC服务的标识，为常量9			               |
|   $uuid             |   $appAccount在MIMC内对应userId，开发者可忽略                   |
|   $token	      |   $appAccount在MIMC系统中的token 	                          |
|   $packetId         |   发送消息包ID，由随机串+递增ID构成，在$appAccount角度看可认为唯一 |

+ HTTP 请求
```
curl "https://mimc.chat.xiaomi.net/api/account/token" -XPOST -d '{"appId":$appId,"appKey":$appKey,"appSecret":$appSecret,"appAccount":$appAccount}' -H "Content-Type: application/json"
```

+ JSON结果
```
{
	"code": 200,
	"message": "success",
	"data": {
		"appId": $appId,
		"appPackage": $appPackage,
		"appAccount": $appAccount,
		"miChid": $chid,
		"miUserId": $uuid,
		"miUserSecurityKey": $appSecret,
		"token": $token
	}
}
```
#### 备注1：对于以上JSON结果，APP不需要理解其格式，通过MIMCTokenFetcher(安卓)/parseTokenDelegate(iOS)原样返回即可
[回到顶部](#readme)

## 推荐消息格式

#### MIMC的消息格式由APP开发者自定义自解析
#### MIMC系统不对用户消息内容进行理解

#### 检查用户在线
+ 用户A`通过MIMC`发送ping给用户B
+ 用户B接收到ping后，`通过MIMC`发送pong给用户A
```
服务器上存储的用户状态会有延迟，所以要获取最精确的用户在线状态，一般用端到端ping-pong方式
```
###### 检查用户在线ping
```
Ping消息建议格式：
    {
    	version: 0, // 建议保留version字段，方便后续协议升级兼容
    	msgId: "PING_12345", // APP业务层面自维护消息ID
    	msgType: "PING", // PING|PONG|...
    	timestamp: "1516763973000", // 建议精确到毫秒
    	payload: "A_account",
    }
```
###### 检查用户在线pong
```
Pong消息建议格式：
    {
    	version: 0, // 建议保留version字段，方便后续协议升级兼容
    	msgId: "PONG_12345", // APP业务层面自维护消息ID
    	msgType: "PONG", // PING|PONG|...
    	timestamp: "1516763973000", // 建议精确到毫秒
    	payload: "B_account",
    }
```
#### 文本消息
+ 用户A`通过MIMC`发送文本消息`(msgId="TEXT_12345")`给用户B
+ 用户B接收文本消息`(msgId="TEXT_12345")`

```
文本消息建议格式：
    {
    	version: 0, // 建议保留version字段，方便后续协议升级兼容
    	msgId: "TEXT_12345", // APP业务层面自维护消息ID
    	msgType: "TEXT", // TEXT|PIC_FILE|AUDIO_FILE|BIN_FILE|...
    	timestamp: "1516763973000", // 建议精确到毫秒
    	payload: "欢迎使用小米即时消息云(MIMC)",
    }
```
#### 多媒体消息

+ 用户A将图片文件/语音文件/视频文件`(非实时语音视频聊天)`上传到文件存储服务器，获得一个URL
+ 用户A`通过MIMC`发送多媒体消息`(msgId="PIC_FILE_12345", payload=URL)`给用户B
+ 用户B接收多媒体消息`(msgId="PIC_FILE_12345", payload=URL)`，通过URL下载图片文件/语音文件/视频文件
```
多媒体消息建议格式：
    {
	version: 0, // 建议保留version字段，方便后续协议升级兼容
	msgId: "PIC_FILE_12345", // APP业务层面自维护消息ID
	msgType: "PIC_FILE", // TEXT|PIC_FILE|AUDIO_FILE|BIN_FILE|...
	timestamp: "1516763973000", // 建议精确到毫秒
	payload: "https://github.com/Xiaomi-mimc/operation-manual/blob/master/img-folder/MIMC-Official-Accounts.jpg",
    }
```

#### 撤回消息
+ 用户A发送文本消息`(msgId="TEXT_12345")`给用户B
+ 用户B接收文本消息`(msgId="TEXT_12345")`
+ 用户A发送撤回消息`(msgId="TEXT_RECALL_12345")`给用户B
+ 用户B接收撤回消息`(msgId="TEXT_RECALL_12345")`并删除文本消息`(msgId="TEXT_12345")`
```
撤回消息建议格式：
    {
    	version: 0, // 建议保留version字段，方便后续协议升级兼容
    	msgId: "TEXT_RECALL_12345", // APP业务层面自维护消息ID
    	msgType: "TEXT_RECALL", // TEXT_RECALL|PIC_FILE_RECALL|AUDIO_FILE_RECALL|BIN_FILE_RECALL|...
	timestamp: "1516763973090", // 建议精确到毫秒
     	payload: {recall_msgId: "TEXT_12345"}, // 撤回msgId为TEXT_12345的消息
    }
```

#### 已读消息
+ 用户A发送文本消息`(msgId="TEXT_12345")`给用户B
+ 用户B接收文本消息`(msgId="TEXT_12345")`
+ 用户B发送已读消息`(msgId="TEXT_READ_12345")`给用户A
+ 用户A接收已读消息`(msgId="TEXT_READ_12345")`并标记文本消息`(msgId="TEXT_12345")`已读

```
已读消息建议格式：
    {
    	version: 0, // 建议保留version字段，方便后续协议升级兼容
	msgId: "TEXT_READ_12345", // APP业务层面自维护消息ID
	msgType: "TEXT_READ", // TEXT_READ|PIC_FILE_READ|AUDIO_FILE_READ|BIN_FILE_READ|...
	timestamp: "1516763973134", // 建议精确到毫秒
	payload: {read_msgId: "TEXT_12345"}, // 已读msgId为TEXT_12345的消息
    }
```

#### 其他自定义消息功能
+ 参考以上，设置对应的msgType/payload

[回到顶部](#readme)

## 跨应用聊天

#### 实现两个不同的APP之间聊天，使用同一个appId/appKey/appSec即可。

## 推送消息

### 参数列表

|   Variable          | Meanings  |
| :------------------ | :-----------------------------------|
|   $appId            |   小米开放平台申请的AppId             |
|   $appKey           |   小米开放平台申请的AppKey            |
|   $appSecret        |   小米开放平台申请的AppSecret	     |
|   $fromAccount      |   表示消息发送方在APP帐号系统内唯一ID   |
|   $fromResource     |   表示消息发送方设备的标识             |
|   $toAccount        |   表示消息接收方在APP帐号系统内唯一ID   |
|   $msgType          |   表示发送消息的类型<br />msgType="base64": msg是base64编码后的数据，一般传输二进制数据时使用;<br />msgType="": msg是原始数据，一般传输String数据时使用 |
|   $topicId	      |   表示群ID                           |
|   $packetId         |   表示发送消息包ID                    |

### 推送单聊信息

+ HTTP 请求
```
curl https://mimc.chat.xiaomi.net/api/push/p2p/ -XPOST -d '{"appId":$appId, "appKey":$appKey，"appSecret":$appSecret, "fromAccount":$fromAccount, "fromResource":$fromResource, "toAccount":$toAccount, "msg":$msg, "msgType":$msgType}' -H "Content-Type: application/json"
```

+ JSON结果
```
{
	"code":200,
	"data":{"packetId":$packetId},
	"message":"success"
}
```

### 推送群聊信息

+ HTTP 请求
```
curl https://mimc.chat.xiaomi.net/api/push/p2t/ -XPOST -d '{"appId":$appId, "appKey":$appKey，"appSecret":$appSecret, "fromAccount":$fromAccount, "fromResource":$fromResource, "msg":$msg, "topicId":$topicId, "msgType":$msgType}' -H "Content-Type: application/json"
```

+ JSON结果
```
{
	"code":200,
	"data":{"packetId":$packetId},
	"message":"success"
}
```
[回到顶部](#readme)

## 消息回调

### 即时消息回调
```
2018.02上线
```
### 离线消息回调
```
2018.02上线
```
[回到顶部](#readme)

## 群聊消息

### 参数列表

|   Variable   | Meanings  |
| :------------------ | :--------------------------------------------------------|
|   $appId            |   小米开放平台申请的AppId                                  |
|   $appKey           |   小米开放平台申请的AppKey                                 |
|   $appSecret        |   小米开放平台申请的AppSecret	                          |
|   $topicId          |   表示群ID                                                |
|   $topicName        |   表示创建群的时候所指定的群名称                            |
|   $topicId1         |   表示查询所属群信息时用户所加入群的群ID	                   |
|   $topicId2         |   表示查询所属群信息时用户所加入群的群ID                     |
|   $topicName1       |   表示查询所属群信息时用户所加入群的群名称                    |
|   $topicName2       |   表示查询所属群信息时用户所加入群的群名称	                  |
|   $topicBulletin1   |	  表示查询所属群信息时用户所加入群的群公告                    |
|   $topicBulletin2   |   表示查询所属群信息时用户所加入群的群公告                    |
|   $newBulletin      |	  表示更新群时设置的新群公告                                |
|   $newTopicName     |   表示更新群时设置的新群名称                                |
|   $ownerUuid	      |	  表示群主在MIMC帐号系统内uuid(使用user.getUuid()获取)      |
|   $ownerAccount     |	  表示群主在APP帐号系统内唯一ID                             |
|   $ownerToken	      |	  表示群主token（使用user.getToken()获取）                 |
|   $userAccount1     |	  表示群成员1号在APP帐号系统内唯一ID                        |
|   $userAccount2     |	  表示群成员2号在APP帐号系统内唯一ID                        |
|   $userAccount3     |   表示群成员3号在APP帐号系统内唯一ID                        |
|   $userAccount4     |	  表示群成员4号在APP帐号系统内唯一ID                        |
|   $userAccount5     |	  表示群成员5号在APP帐号系统内唯一ID                        |
|   $userUuid1	      |	  表示userAccount1在MIMC帐号系统内uuid(使用user.getUuid()获取)|
|   $userToken1	      |	  表示userAccount1的token（使用user.getToken()获取）      |


#### 备注：
```
token的获取使用User.getToken()方法。
uuid的获取使用User.getUuid()方法，uuid由MIMC根据($appId, $appAccount)生成，全局唯一。
身份认证有两种方式：1. token（$ownerToken/$userToken1）; 2. app信息,app帐号（$appKey，$appSecret，$ownerAccount/$userAccount1）。
当两种认证信息都存在时，优先验证前者。前者一般用于app客户端，后者一般用于app服务端。下面给出了这两种的使用方式。
```

### 创建群

#### 如下为$ownerAccount创建群

+ HTTPS请求
```
curl "https://mimc.chat.xiaomi.net/api/topic/$appId" -XPOST -d '{"topicName":$topicName,"accounts":"$userAccount1,$userAccount2,$userAccount3"}' -H "Content-Type: application/json" -H "token:$ownerToken"

curl "https://mimc.chat.xiaomi.net/api/topic/$appId" -XPOST -d '{"topicName":$topicName,"accounts":"$userAccount1,$userAccount2,$userAccount3"}' -H "Content-Type: application/json" -H "appKey:$appKey" -H "appSecret:$appSecret" -H "appAccount:$ownerAccount"
```

+ JSON结果
```
{
	 "code":200,"message":"success",
	 "data":{
		"topicInfo":{
			"topicId":$topicId,
			"ownerUuid":$ownerUuid,
			"ownerAccount":$ownerAccount,
			"topicName":$topicName,
			"bulletin":""
		},
		"members":[
			{"uuid":$ownerUuid,"account":$ownerAccount},
			{"uuid":$userUuid1,"account":$userAccount1},
			{"uuid":$userUuid2,"account":$userAccount2},
			{"uuid":$userUuid3,"account":$userAccount3}
		]
	}
}
```

### 查询指定群信息

#### 如下为$userAccount1查询群信息

+ HTTPS请求
```
curl "https://mimc.chat.xiaomi.net/api/topic/$appId/$topicId" -H "Content-Type: application/json" -H "token:$userToken1"

curl "https://mimc.chat.xiaomi.net/api/topic/$appId/$topicId" -H "Content-Type: application/json" -H "appKey:$appKey" -H "appSecret:$appSecret" -H "appAccount:$userAccount1"
```

+ JSON结果
```
{
	 "code":200,"message":"success",
	 "data":{
		"topicInfo":{
			"topicId":$topicId,
			"ownerUuid":$ownerUuid,
			"ownerAccount":$ownerAccount,
			"topicName":$topicName,
			"bulletin":""
		},
		"members":[
			{"uuid":$ownerUuid,"account":$ownerAccount},
			{"uuid":$userUuid1,"account":$userAccount1},
			{"uuid":$userUuid2,"account":$userAccount2},
			{"uuid":$userUuid3,"account":$userAccount3}
		]
	 }
}
```

### 查询所属群信息

#### 如下为$userAccount1查询加入的所有群信息

+ HTTPS请求
```
curl "https://mimc.chat.xiaomi.net/api/topic/$appId/account" -H "Content-Type: application/json" -H "token:$userToken1"

curl "https://mimc.chat.xiaomi.net/api/topic/$appId/account" -H "Content-Type: application/json" -H "appKey:$appKey" -H "appSecret:$appSecret" -H "appAccount:$userAccount1"
```

+ JSON结果
```
{
	"code":200,
	"message":"success",
	"data":[
		{
			"topicId":$topicId1,
			"ownerUuid":$ownerUuid,
			"ownerAccount":$ownerAccount,
			"topicName":$topicName1,
			"bulletin":$topicBulletin1
		},
		{
			"topicId":$topicId2,
			"ownerUuid":$ownerUuid,
			"ownerAccount":$ownerAccount,
			"topicName":$topicName2,
			"bulletin":$topicBulletin2
		}
	]
}
```

### 邀请用户加入群

#### 如下为$userAccount1邀请$userAccount4,$userAccount5加入群

+ HTTPS请求
```
curl "https://mimc.chat.xiaomi.net/api/topic/$appId/$topicId/accounts" -XPOST -d '{"accounts":"$userAccount4,$userAccount5"}' -H "Content-Type: application/json" -H "token:$userToken1"

curl "https://mimc.chat.xiaomi.net/api/topic/$appId/$topicId/accounts" -XPOST -d '{"accounts":"$userAccount4,$userAccount5"}' -H "Content-Type: application/json" -H "appKey:$appKey" -H "appSecret:$appSecret" -H "appAccount:$userAccount1"
```

+ JSON结果
```
{
	 "code":200,"message":"success",
	 "data":{
		"topicInfo":{
			"topicId":$topicId,
			"ownerUuid":$ownerUuid,
			"ownerAccount":$ownerAccount,
			"topicName":$topicName,
			"bulletin":""
		},
		"members":[
			{"uuid":$ownerUuid,"account":$ownerAccount},
			{"uuid":$userUuid1,"account":$userAccount1},
			{"uuid":$userUuid2,"account":$userAccount2},
			{"uuid":$userUuid3,"account":$userAccount3},
			{"uuid":$userUuid4,"account":$userAccount4},
			{"uuid":$userUuid5,"account":$userAccount5}
		]
	}
}
```

### 非群主用户退群

#### 如下为$userAccount1退群

+ HTTPS请求
```
curl "https://mimc.chat.xiaomi.net/api/topic/$appId/$topicId/account" -XDELETE -H "Content-Type: application/json" -H "token:$userToken1"

curl "https://mimc.chat.xiaomi.net/api/topic/$appId/$topicId/account" -XDELETE -H "Content-Type: application/json" -H "appKey:$appKey" -H "appSecret:$appSecret" -H "appAccount:$userAccount1"
```

+ JSON结果
```
{
	 "code":200,"message":"success",
	 "data":{
		"topicInfo":{
			"topicId":$topicId,
			"ownerUuid":$ownerUuid,
			"ownerAccount":$ownerAccount,
			"topicName":$topicName,
			"bulletin":""
		},
		"members":[
			{"uuid":$ownerUuid,"account":$ownerAccount},
			{"uuid":$userUuid2,"account":$userAccount2},
			{"uuid":$userUuid3,"account":$userAccount3},
			{"uuid":$userUuid4,"account":$userAccount4},
			{"uuid":$userUuid5,"account":$userAccount5}
		]
	}
}
```

+ 若是群主退群，则JSON结果如下：
```
{"code":500,"message":"quit topic fail","data":null}
```
 
### 群主踢用户退群

#### 如下为$ownerAccount踢$userAccount4,$userAccount5退出群

+ HTTPS请求
```
curl "https://mimc.chat.xiaomi.net/api/topic/$appId/$topicId/accounts?accounts=$userAccount4,$userAccount5" -XDELETE -H "Content-Type: application/json" -H "token:$ownerToken"

curl "https://mimc.chat.xiaomi.net/api/topic/$appId/$topicId/accounts?accounts=$userAccount4,$userAccount5" -XDELETE -H "Content-Type: application/json" -H "appKey:$appKey" -H "appSecret:$appSecret" -H "appAccount:$ownerAccount"
```

+ JSON结果
```
{
	 "code":200,"message":"success",
	 "data":{
		"topicInfo":{
			"topicId":$topicId,
			"ownerUuid":$ownerUuid,
			"ownerAccount":$ownerAccount,
			"topicName":$topicName,
			"bulletin":""
		},
		"members":[
			{"uuid":$ownerUuid,"account":$ownerAccount},
			{"uuid":$userUuid2,"account":$userAccount2},
			{"uuid":$userUuid3,"account":$userAccount3}
		]
	}
}
```

### 群主更新群信息

#### 如下为$ownerAccount更新群信息：群主为$userAccount2，群名称为$newTopicName，群公告为$newBulletin

+ HTTPS请求
```
curl "https://mimc.chat.xiaomi.net/api/topic/$appId/$topicId" -XPUT -d '{"topicId":$topicId, "ownerUuid":$userUuid2,"topicName":$newTopicName,"bulletin":$newBulletin}' -H "Content-Type: application/json" -H "token:$ownerToken"

curl "https://mimc.chat.xiaomi.net/api/topic/$appId/$topicId" -XPUT -d '{"topicId":$topicId, "ownerUuid":$userUuid2,"topicName":$newTopicName,"bulletin":$newBulletin}' -H "Content-Type: application/json" -H "appKey:$appKey" -H "appSecret:$appSecret" -H "appAccount:$ownerAccount"
```

+ JSON结果
```
{
	 "code":200,"message":"success",
	 "data":{
		"topicInfo":{
			"topicId":$topicId,
			"ownerUuid":$userUuid2,
			"ownerAccount":$userAccount2,
			"topicName":$newTopicName,
			"bulletin":$newBulletin
		},
		"members":[
			{"uuid":$ownerUuid,"account":$ownerAccount},
			{"uuid":$userUuid2,"account":$userAccount2},
			{"uuid":$userUuid3,"account":$userAccount3}
		]
	}
}
```

### 群主销毁群

#### 如下为群主销毁群

+ HTTPS请求
```
curl "https://mimc.chat.xiaomi.net/api/topic/$appId/$topicId" -XDELETE -H "Content-Type: application/json" -H "token:$ownerToken"

curl "https://mimc.chat.xiaomi.net/api/topic/$appId/$topicId" -XDELETE -H "Content-Type: application/json" -H "appKey:$appKey" -H "appSecret:$appSecret" -H "appAccount:$ownerAccount"
```

+ JSON结果
```
{"code":200,"message":"success！","data":null}
```
[回到顶部](#readme)

## 消息漫游

### 参数列表

|   Variable   | Meanings  |
| :----------------- | :---------------------------------------------------------|
|  $appId            |  小米开放平台申请的AppId                  		  |
|  $token            |  查询方的token（使用user.getToken()获取）                    |
|  $account          |  查询方在APP系统内唯一ID                                    |
|  $fromAccount      |  消息发送方在APP帐号系统内唯一ID                             |
|  $toAccount        |  消息接收方在APP帐号系统内唯一ID                             |
|  $topicId          |  表示群ID                                		      |
|  $utcFromTime      |  表示查询开始时间，UTC时间，单位毫秒       		    |
|  $utcToTime        |  表示查询结束时间，UTC时间，单位毫秒       		    |
|  $row              |  表示返回的消息条数                       		  |
|  $messages         |  表示返回的消息集合                       		  |
|  $sequence         |  sequence主要用来做消息的排序和去重，全局唯一		   |	 
|  $payload	     |  表示经过Base64编码的消息体，app端需要进行Base64解码          |
|  $ts               |  表示消息时间戳                          		    |

#### 备注：
```
utcFromTime和utcToTime的时间间隔不能超过24小时，查询状态为[utcFromTime,utcToTime)
消息漫游为用户保存最近半年的历史消息
```

### 拉取单聊消息记录

#### 如下为拉取单聊消息记录

+ HTTPS请求(POST)
```
curl https://mimc.chat.xiaomi.net/api/msg/p2p/query/ -XPOST -d '{"appId":$appId,"toAccount":$toAccount,"fromAccount":$fromAccount,"utcFromTime":$utcFromTime,"utcToTime":$utcToTime}' -H "Content-Type: application/json;charset=UTF-8" -H "Accept:application/json;charset=UTF-8" -H "token:$token"
```

+ JSON结果示例
```
{
     "code": 200,
     "message": "success",
     "data": {
         "appId": $appId,
         "toAccount": $toAccount,
         "fromAccount":$fromAccount,
         "messages": [
             {
                 "sequence": $sequence,
                 "payload": $payload,
                 "ts": $ts
             }
         ],
         "row": 1
     }
 }
```

### 拉取群聊消息记录

#### 如下为拉取群聊消息记录

+ HTTPS请求(POST)
```
curl https://mimc.chat.xiaomi.net/api/msg/p2t/query/ -XPOST -d '{"appId":$appId,"account":$account,"topicId":$topicId,"utcFromTime":$utcFromTime,"utcToTime":$utcToTime}' -H "Content-Type: application/json;charset=UTF-8" -H "Accept:application/json;charset=UTF-8" -H "token:$token"
```

+ JSON结果示例
```
{
     "code": 200,
     "message": "success",
     "data": {
         "appId": $appId,
         "topicId": $topicId,
         "row": 2,
         "messages": [
             {
                 "sequence": $sequence,
                 "fromAccount": $fromAccount,
                 "payload": $payload,
                 "ts": $ts
             },
             {
                 "sequence": $sequence,
                 "fromAccount": $fromAccount,
                 "payload": $payload,
                 "ts": $ts
             }
         ]
     }
 }
```
[回到顶部](#readme)

## 联系我们

#### 邮箱
mimc-help@xiaomi.com

#### 微信公众号
<div align="center"><img width="200" height="200" src="https://github.com/Xiaomi-mimc/operation-manual/blob/master/img-folder/MIMC-Official-Accounts.jpg"/></div>

#### MIMC官方QQ群
<div align="center"><img width="200" height="260" src="https://github.com/Xiaomi-mimc/operation-manual/blob/master/img-folder/MIMC-QQGroup.png"/></div>

[回到顶部](#readme)
