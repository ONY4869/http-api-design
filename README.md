# HTTP API 设计指南

## 简介

这份指南描述了一套HTTP+JSON API设计实践, 最初摘自 [Heroku Platform API](https://devcenter.heroku.com/articles/platform-api-reference).

这份指南是上面那份API的补充，同时也是Heroku的内部API指南。我们也希望它可以引起其他API设计者们的兴趣。

我们的目标是一致性和专注于业务开发，同时避免浪费过多的时间在设计层面上(译者注：bikeshdding=Futile investment of time and energy in marginal technical issues)。 我们正在寻找的是 _一个优秀的，一致的，文档齐全的方法_ 去设计APIs, 不见得是_唯一的/完美的方法_。

我们认为您熟悉HTTP+JSON APIs的基础知识和不覆盖所有这些在本指南的基础。

我们欢迎您的[贡献](https://github.com/interagent/http-api-design/blob/master/CONTRIBUTING.md).

## 内容

* [基础](#foundations)
  *  [要求使用TLS](#require-tls)
  *  [添加版本到Accepts头部](#version-with-accepts-header)
  *  [用Etags支持缓存](#support-caching-with-etags)
  *  [用ID跟踪请求](#trace-requests-with-request-ids)
  *  [页码范围](#paginate-with-ranges)
* [请求](#requests)
  *  [返回合适的状态码](#return-appropriate-status-codes)
  *  [Provide full resources where available](#provide-full-resources-where-available)
  *  [在请求体中接受系列化JSON](#accept-serialized-json-in-request-bodies)
  *  [使用统一路径格式](#use-consistent-path-formats)
  *  [路径和属性用小写](#downcase-paths-and-attributes)
  *  [支持非ID取值方便查找](#support-non-id-dereferencing-for-convenience)
  *  [最小化路径嵌套](#minimize-path-nesting)
* [返回](#responses)
  *  [提供资源UUIDs](#provide-resource-uuids)
  *  [提供标准时间戳Provide standard timestamps](#provide-standard-timestamps)
  *  [使用ISO8601 UTC时间格式](#use-utc-times-formatted-in-iso8601)
  *  [嵌套外键关联](#nest-foreign-key-relations)
  *  [生成结构化错误](#generate-structured-errors)
  *  [显示rate limit状态](#show-rate-limit-status)
  *  [在所有返回中保持JSON最小化](#keep-json-minified-in-all-responses)
* [产出物](#artifacts)
  *  [提供 machine-readable JSON模式](#provide-machine-readable-json-schema)
  *  [提供 human-readable 文档](#provide-human-readable-docs)
  *  [提供执行示例](#provide-executable-examples)
  *  [描述API稳定性](#describe-stability)

### Foundations

#### Require TLS

要求使用TLS(传输层安全协议)才能访问API，没有任何例外，试图找出或解释什么时候该用TLS，什么时候不该用TLS是没必要的。用上就对了。

理论上，对于http或者80端口非TLS请求，简单的拒绝它们，不用返回数据，避免不安全的数据交换。对于那些不允许这么做的环境, 返回 `403 Forbidden`吧。

不鼓励使用重定向因为这会允许不好的客户端行为，且不提供任何收益(译者注：吃力不讨好)。重定向的客户端不仅增加了服务端的压力，而且使得TLS无效，因为敏感数据可能已经在第一次请求中就泄露了。

#### Version with Accepts header

从一开始就版本化API. 使用`Accepts`头部独立一个自定义的内容类型来显示版本，例如：

```
Accept: application/vnd.heroku+json; version=3
```

最好是不要使用默认版本，取而代之的是让客户端使用一个确切的版本号来明确他们的用途。

#### Support caching with Etags

在所有的返回中包含一个`ETag`头部，用于标识返回资源的特定版本。这样用户就可以通过在`If-None-Match`头部提供的这些值来检查他们后续的请求是否过时。

#### Trace requests with Request-Ids

在每个API返回中包含一个用UUID值生成的`Request-Id`头部。如果服务端和客户端都记录这些值，这将帮助我们追踪和调试请求信息。

#### Paginate with Ranges

对于大数据返回分页是有必要的，使用`Content-Range`头部来表达分页请求，想知道请求和返回头部，状态码，限制，排序和页码使用的更多详情，请参考[Heroku Platform API on Ranges](https://devcenter.heroku.com/articles/platform-api-reference#ranges)的示例。