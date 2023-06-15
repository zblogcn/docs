# API 整体设计

## 概要

以 Z-BlogPHP 的 **功能模块（Module）** 为划分依据，可以分为以下九个模块：

- 用户模块
- 文章模块（包括页面）
- 应用模块（包括插件和主题）
- 侧栏模块
- 附件模块
- 评论模块
- 标签模块
- 分类模块
- 系统及设置模块

**接口行为（Action）** 简单来说就是实现某个模块的数据增删改查操作，比如用户模块的新增用户、用户登录、用户信息的获取与修改等操作。

ZBP API 的整体思想是：服务端根据客户端发送的请求，针对 **模块** 进行相应的 **行为** 操作， 并将执行结果返回给客户端。本质上跟原有的网页版差别不大，就是同一套业务逻辑下的不同输出形式，网页版返回的是 HTML，API 返回的是 JSON。


## 统一入口

规定接口入口为：`http[s]://<域名>/zb_system/api.php`


## 请求方法

客户端主要使用 GET 和 POST 这两种 HTTP 请求方法来请求服务端资源。

其中，GET 表示“获取”操作，对应数据库中的操作为 SELECT。如：获取某篇文章。

POST 表示“新增”、“修改”和“删除”操作，对应数据库中的操作为 INSERT、 UPDATE 和 DELETE。如：新增一个用户。

为保证对大量参数的支持，对于“获取/查询”类型的接口，同时支持 GET 和 POST 两种请求方式；对于“增删改”类型的接口，只支持 POST 请求方式。


## 公共请求消息头

| 消息头（Header） | 是否必需 | 示例值                          | 说明                                                                 |
| ---------------- | -------- | ------------------------------- | -------------------------------------------------------------------- |
| Content-Type     | 可选     | application/json; charset=utf-8 | 客户端接受的消息格式。<br />不管怎样，服务端始终返回 JSON 格式内容。 |
| Accept-Encoding  | 可选     | gzip, deflate, br               | 客户端接受的压缩算法                                                 |
| User-Agent       | 可选     | Mozilla/5.0                     | -                                                                    |
| Referer          | 可选     | `https://example.com/`          | 来源地址                                                             |
| Accept-Language  | 可选     | zh-cn                           | 客户端接受的语言代码                                                 |


## 权限认证

### 获取鉴权

POST `https://example.com/zb_system/api.php?mod=member&act=login`

HTTP Body:

```json
{
  "username": "zblog_usr",
  "password": "zblog_pwd",
  "savedate": 30
}
```
`"savedate": 30`：有效期 30 天；

**注：`password`建议使用 MD5(密码原文) 值；也可以使用密码原文(有被抓包泄漏风险) ↑↑**

Response Body:

```json
{
  "code": 200,
  "message": "操作成功",
  "data": {
    "user": {
      "ID": "1",
      "Level": "1",
      "Name": "zblog_usr"
    },
    "token": "6K+t6K="
  },
  "error": null
}
```

### 使用

对于后续需要身份验证的请求，接受三种方式传递「鉴权 Token」：

1. 设置如下 `authorization` 头：`Authorization: Bearer {YourToken}` ***（推荐）***

2. POST 方式访问 api 时，可以提交一个 input 表单，name 为 token，value 为 {YourToken}

3. GET 方式访问 api 时，附加在 URL 中：`https://example.com/zb_system/api.php?mod=setting&act=get&token={YourToken}`（这种方式会泄漏 token）

**重要：对于第 1 种方法，可能需要额外对 web 空间进行设置，参见「[常见问题](books/api-20-faq?id=apache-%e8%8e%b7%e5%8f%96%e4%b8%8d%e5%88%b0-authorization-%e5%a4%b4%e4%bf%a1%e6%81%af "常见问题")」；**

## 构造请求

<!-- ### 规定

- URI 对大小写不敏感（不区分大小写）。

- POST 的 HTTP Body 中，JSON 内容的属性名一律使用小写。 -->


### 模块命名

| 模块                       | 命名     |
| -------------------------- | -------- |
| 用户模块                   | member   |
| 文章模块（包括页面）       | post     |
| 应用模块（包括插件和主题） | app      |
| 侧栏模块                   | sidebar  |
| 附件模块                   | upload   |
| 评论模块                   | comment  |
| 标签模块                   | tag      |
| 分类模块                   | category |
| 系统及设置模块             | system   |


### URL 通用格式

> `https://example.com/zb_system/api.php?mod=<模块名>[&act=<行为名>][&其他...]`

<!-- 伪静态的情况：

> `http[s]://<域名>/zb_system/api?mod=<模块名>[&act=<行为名>][&其他...]` -->

由于是单一入口，因此不同的模块不能用路径表示，需要用 URL 参数表示。参数排序不分先后。


### URL 参数

**模块名**

- 模块名就是功能模块的英文名。

**行为名**

- 行为名代表操作，比如 `act=post` 表示新增资源，`act=update` 表示修改/更新资源，`act=delete` 表示删除资源。

- 行为名缺省机制：

  在请求方法为 GET 的前提下，因为 GET 语义即为“获取”，故不指定行为名时，默认进行的操作即为 `act=get` 这一行为。

  因此，“获取某个用户的信息”这一操作的接口为`mod=user&act=get&id=123`，可以省略掉 `act=get`，直接用 `mod=user&id=123` 也行。

  POST 请求不支持行为名缺省机制，必须指定行为名。

- 多语义行为名：

  多语义行为名可以表示针对更为具体的资源对象进行操作，多个语义的行为名词之间用英文下划线“_”分隔。

  如：对于侧栏模块（mod=sidebar），存在两种资源对象，一是侧栏本身，二是侧栏当中的“模块”。`mod=sidebar&act=get&id=1` 这一接口表示 “获取 ID 为 1 的侧栏的信息”，若要实现“获取侧栏中的模块的信息”，则需要更为详细的语义 `act=get_module` ，即 `mod=sidebar&act=get_module&id=name`。其他的存在多个资源对象的情况均以此类推。

**其他参数**

- 表示一些附加内容，比如约束条件、授权信息的 Token 这些，或者其他第三方开发者自定义的内容。


### 约束与过滤

约束过滤器（Filter）一般用于列表类的资源，比如文章列表这种。

具体支持的约束条件由该功能模块的数据表决定。

多个约束条件默认使用“AND”（与）逻辑。

| 参数    | 类型   | 示例值      | 说明                                       |
| ------- | ------ | ----------- | ------------------------------------------ |
| perpage | int    | 50          | 每页返回数量，未鉴权请求受最大值限制       |
| page    | int    | 2           | 指定第几页                                 |
| sortby  | string | name        | 排序依据，大小写敏感，具体见「排序依据表」 |
| order   | string | asc 或 desc | 排序顺序，asc：升序，desc：降序            |

各「API 模块」支持的排序依据：

| mod      | sortby                                                                   |
| -------- | ------------------------------------------------------------------------ |
| category | ID, Order, Count, Group                                                  |
| comment  | ID, PostTime                                                             |
| member   | ID, CreateTime, PostTime, UpdateTime, Articles, Pages, Comments, Uploads |
| post     | ID, CreateTime, PostTime, UpdateTime, CommNums, ViewNums                 |
| tag      | ID, Order, Count                                                         |
| upload   | ID, PostTime, DownNums                                                   |

不同「API 模块」专有的查询参数见：「[接口列表](books/api-10-mods "接口列表")」；

## 公共消息响应头

| 消息头（Header） | 说明 | 示例值                          | 说明                                             |
| ---------------- | ---- | ------------------------------- | ------------------------------------------------ |
| Content-Type     | 必需 | application/json; charset=utf-8 | 目前默认只支持返回 JSON 格式                     |
| Content-Encoding | 可选 | gzip                            | 内容压缩方式，根据客户端选择，默认启用 gzip 压缩 |
| Date             | 可选 | Sun, 23 Feb 2020 07:03:41 GMT   | 服务端响应时间，有些时候可以用来测试延迟         |


## 通用返回格式

服务端响应的 Body 内容为 JSON 字符串，统一为如下格式：

| 参数名  | 类型   | 说明     |
| ------- | ------ | -------- |
| message | string | 响应描述 |
| data    | object | 主要数据 |
| error   | object | 错误信息 |
| runtime | object | 调试信息 |

详细的错误信息和调试信息仅在启用了调试模式的情况下输出。

属性名一律使用小写，如有需要，多个单词之间使用英文下划线”_“分隔，如”new_data“。

## 响应内容示例及状态码

示例一，没有权限：

> 原因：未登录或登录用户权限不够；「[登录和鉴权](books/api-05-design?id=权限认证 "登录和鉴权")」

```json
{
  "code": 401,
  "message": "没有权限",
  "data": null,
  "error": null,
  "runtime": {
    "time": "94.76",
    "query": 4,
    "memory": 1684
  }
}
```
-----


示例二，非法访问：

> 原因：未正确设置鉴权 Token；「[登录和鉴权](books/api-05-design?id=权限认证 "登录和鉴权")」

```json
{
  "code": 419,
  "message": "非法访问",
  "data": null,
  "error": null,
  "runtime": {
    "time": "94.29",
    "query": 4,
    "memory": 1657
  }
}
```

-----

示例三，成功请求某个用户的信息：

```json
{
  "code": 200,
  "message": "OK",
  "data": {
    "member": {
      "ID": 123,
      "Name": "Chris",
      "Email": "123@example.com",
      "StaticName": "admin"
    }
  },
  "error": null,
  "runtime": {
    "time": "90.74",
    "query": 5,
    "memory": 1807
  }
}
```

## 内部调用

为了便于系统本身（包括主题、插件）调用网站的 API，我们提供了内部调用的方法。

### 调用

`ApiExecute`($mod, $act, $get = array(), $post = array())

示例

```php
var_dump(ApiExecute('post', 'get', array('id' => '1')));
```

### 增加/删除 Private API

> **说明**  
> 在 API 系统中，Private API 为只能被自身调用，即只能通过 `ApiExecute` 方法调用的 API.

```php
ApiAddPrivateMod('newapi', __DIR__ . '/myapi.php'); // 实现文件以 'newapi' 为模块名插入进系统 Private API 里
ApiRemovePrivateMod($modname); // 删除以'newapi'为模块名的 Private API里
```

***本篇内容所用代码需要由主程序 1.7.3.3230 版本及更高版本实现***

