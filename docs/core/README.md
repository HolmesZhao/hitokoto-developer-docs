# 核心接口

## 简介

一言核心接口是萌创团队在开发过程中，暴露给外部环境，方便开发的系列接口。它们实现了：

* 用户操作
* 句子操作
* 评分操作
* 举报操作

的功能。目前，这些接口可通过无状态模型使用。

## 接口地址

核心接口实现了两套认证机制，分别对应两个入口：

* 无状态服务： <https://hitokoto.cn/api/restful/v1>
* 会话认证：<https://hitokoto.cn/api/common/v1>

> 基于会话认证的接口的最初目的是便于前端调用，因此具备 CSRF 验证，不适合开发者使用。

## 约束

> 接下来，我们便以 **无状态** 接口作为入口进行讲解。

### 认证方式

无状态服务通过令牌（`token`）认证，因此每个需要验权的接口，其请求必须包含 `token` 字段。
目前支持三种验证方式：

* 请求参数，形如： `?token=xxxxx`
* POST 参数（表单），例如：在 POST 表单中添加 `<input type="hidden" name="token" value="xxxx" >`
* `bearer` 授权

### 返回约束

返回结构以方便，可快捷测试，方便检验为目标。以下为参数的约束：

| 参数    | 例子          | 定义                                                                                                                               |
|---------|---------------|------------------------------------------------------------------------------------------------------------------------------------|
| status  | 200           | 状态码。正整数定义符合 HTTP 状态码约束，自定义错误为负值。请注意，由于偷懒，目前错误码存在重复定义，请以具体接口为准。             |
| message | "ok."         | 信息，当状态码不为 200 时，补充说明错误的字段。                                                                                    |
|  data   | []            | 当状态码为 200 时，且接口不为无返回接口，该字段会返回一个长度大于等于 1 的数组；400 状态码下可能包含校验器的报错信息；其余时候为空数组 |
| ts      | 1581759895072 | 接口时间戳，可用于时间比对                                                                                                         |

下面是一个示例：
`GET` `https://hitokoto.cn/api/restful/v1/hitokoto/62c12303-b3fa-4720-a7c5-2985bf049e60?token=xxx`

```json
{
  "status": 200,
  "message": "ok.",
  "data": [
    {
      "hitokoto": "过去都是假的，回忆是一条没有归途的路，以往的一切春天都无法复原，即使最狂热最坚贞的爱情，归根结底也不过是一种瞬息即逝的现实，唯有孤独永恒。",
      "uuid": "62c12303-b3fa-4720-a7c5-2985bf049e60",
      "type": "d",
      "from": "百年孤独",
      "from_who": null,
      "creator": "戎歌",
      "creator_uid": 1195,
      "reviewer": 0,
      "commit_from": "web",
      "created_at": "1532672964",
      "status": "accept"
    }
  ],
  "ts": 1582053586728
}
```

> **请注意: 接口会尽量使 HTTP 状态码为 200，但是由于上游中间件的问题，会出现 HTTP 不为 200 的情况，需要开发者自行处理。**

## 接口方法

本节将提供目前暴露的接口表，方便快速查询。

> **请求头请添加 `Accept: application/json` 以保证接口输出一定是 JSON。**  
> 由于 Laravel 底层对 PUT 方法的支持不好，提交 PUT 参数时一定要以 `x-www-form-urlencoded` 方法提交参数。  
> 或者，你可以使用 `POST` 方法，额外添加一个字段 `_method: "PUT"` 模拟 PUT 请求。

### 用户部分

| 地址                        | 方法 | 鉴权 | 无返回 | 备注                                     |
|-----------------------------|------|------|--------|------------------------------------------|
| /auth/login                 | POST | N    | N      | 登录接口，成功返回用户信息（包含令牌）   |
| /auth/register              | POST | N    | N      | 注册接口，成功返回用户信息。             |
| /auth/password/reset        | POST | N    | Y      | 重置密码接口                             |
| /like                       | GET  | N    | N      | 返回句子赞的相关信息                     |
| /like                       | POST | N    | N      | 提交赞，成功返回提交者 IP                |
| /like/cancel                | GET  | Y    | N      | 撤回已经发出的喜爱                      |
| /mark                       | GET  | N    | N      | 获取所有的审核标记                             |
| /user                       | GET  | Y    | N      | 获取用户信息                             |
| /user/email/verify          | PUT  | Y    | Y      | 申请验证邮箱                             |
| /user/token                 | GET  | Y    | N      | 返回用户令牌的相关信息                   |
| /user/token/refresh         | PUT  | Y    | N      | 重置令牌，返回新令牌的相关信息           |
| /user/password              | PUT  | Y    | Y      | 修改密码                                 |
| /user/email                 | PUT  | Y    | Y      | 修改邮箱（未来行为可能发生变化）         |
| /user/notification/settings | GET  | Y    | N      | 获取用户通知设置                         |
| /user/notification/settings | PUT  | Y    | N      | 设定用户通知设置，返回新设置             |
| /user/hitokoto/summary      | GET  | Y    | N      | 获得用户一言提交数据的概览               |
| /user/hitokoto/like         | GET  | Y    | N      | 获得用户赞的句子                        |
| /user/hitokoto/history      | GET  | Y    | N      | 获得用户历史的一言提交                   |
| /user/hitokoto/history/pending     | GET  | Y      | N      | 获得用户历史的一言提交（审核中部分）|
| /user/hitokoto/history/refuse      | GET  | Y      | N      | 获得用户历史的一言提交（已拒绝部分）      |
| /user/hitokoto/history/accept      | GET  | Y      | N      | 获得用户历史的一言提交（已上线部分）       |
| /hitokoto/append            | POST | Y    | N      | 添加一言，返回审核队列中新句子的信息     |
| /hitokoto/:uuid             | GET  | Y    | N      | 查看指定一言的信息（通过 UUID）    |
| /hitokoto/:uuid/mark        | GET  | Y    | N      | 查看指定一言的审核标记（通过 UUID）    |
| /hitokoto/score             | POST | Y    | N      | 为已上线的句子评分，返回评分相关信息     |
| /hitokoto/score             | GET  | Y    | N      | 获得句子的评分信息                       |
| /hitokoto/report            | POST | Y    | N      | 举报一言存在问题，返回提交举报的相关信息 |
