title: oauth2
date: 2016-01-08 18:52:29
tags: oauth2
---

### 授权类型
OAuth2 允许采用不同的授权类型(grant_type) 获取 access_token
- Authorization Code
- Implicit
- Resource Owner Password Credentials
- Client Credentials

### 请求认证

在向资源方请求数据时，需要将获得的 Access_token 传递给资源方进行认证，主要有三种传递方式。

- 放在请求头传递
使用请求头传递 access_token 时，客户端采用 [Bearer 认证方案](https://tools.ietf.org/html/rfc2617)，比如：

```
	GET /resource HTTP/1.1
	Host: server.example.com
	Authorization: Bearer mF_9.B5f-4.1JqM
```

- 作为 Form 表单参数传递
当使用 form 表单来传递 access_token 时， HTTP 请求头实体必须包含 `Content_Type` 字段，并且该字段值必须为 `application/x-www-form-urlencoded`，比如：

```
	POST /resource HTTP/1.1
	Host: server.example.com
	Content-Type: application/x-www-form-urlencoded

	access_token=mF_9.B5f-4.1JqM
```

- 作为 URI query 参数传递
直接把 access_token 放在 URI 中是不推荐的做法，除非另外两种办法都无法进行，因为这样会有安全问题，参数会在浏览器记录中留存下来。资源服务器可以选择不提供这种认证方式。

客户端使用 URI query 参数传递 access_token 时，**应该**在头部添加一个 `Cache-Control` 的字段，值为 `no-store` 。客户端成功返回这类请求的响应时，也**应该**在头部添加一个 `Cache-Control` 字段，值为 `private`。


