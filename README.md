# HTTP API 设计指南

## 简介

这份指南描述了一套HTTP+JSON API设计实践, 最初摘自 [Heroku Platform API](https://devcenter.heroku.com/articles/platform-api-reference).

这份指南是上面那份API的补充，同时也是Heroku的内部API指南。我们也希望它可以引起其他API设计者们的兴趣。

我们的目标是一致性和专注于业务开发，同时避免浪费过多的时间在设计层面上(译者注：bikeshdding=Futile investment of time and energy in marginal technical issues)。 我们正在寻找的是 _一个优秀的，一致的，文档齐全的方法_ 去设计APIs, 不见得是_唯一的/完美的方法_。

我们认为您熟悉HTTP+JSON APIs的基础知识和不覆盖所有这些在本指南的基础。

我们欢迎您的[贡献](https://github.com/interagent/http-api-design/blob/master/CONTRIBUTING.md).

## 内容

* [基础](#foundations)
  *  [需要TLS](#require-tls)
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
