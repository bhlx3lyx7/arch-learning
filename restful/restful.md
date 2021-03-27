# RESTful Learning

## Terminology
- resource
- collection of resource
- client
- server
- idempotent
- URL

## Concept
Resource oriented: 
- URI indicates a resource
- method indicates the way to visit the resource

Stateless in server side

## Method
- GET
- POST
- PUT
- PATCH
- DELETE

## Restriction of RESTful
- 客户端-服务端（Client-Server）: 这个更专注客户端和服务端的分离，服务端独立可更好服务于前端、安卓、IOS等客户端设备。
- 无状态（Stateless）：服务端不保存客户端状态，客户端保存状态信息每次请求携带状态信息。
- 可缓存性（Cacheability） ：服务端需回复是否可以缓存以让客户端甄别是否缓存提高效率。
- 统一接口（Uniform Interface）：通过一定原则设计接口降低耦合，简化系统架构，这是RESTful设计的基本出发点。
- 分层系统（Layered System）：客户端无法直接知道连接的是终端还是中间设备，分层允许你灵活的部署服务端项目。
- 按需代码（Code-On-Demand，可选）：按需代码允许我们灵活的发送一些看似特殊的代码给客户端例如JavaScript代码。

## Sample URL
```
/{version}/{resources}/{resource_id}/{subresources}/{subresource_id}
/{version}/{resources}/{resource_id}/action
```

## Status Code
```
1xx：相关信息
2xx：操作成功
3xx：重定向
4xx：客户端错误
5xx：服务器错误
```

```
200 OK - [GET]：服务器成功返回用户请求的数据，该操作是幂等的（Idempotent）。
201 CREATED - [POST/PUT/PATCH]：用户新建或修改数据成功。
202 Accepted - [*]：表示一个请求已经进入后台排队（异步任务）
204 NO CONTENT - [DELETE]：用户删除数据成功。
400 INVALID REQUEST - [POST/PUT/PATCH]：用户发出的请求有错误，服务器没有进行新建或修改数据的操作，该操作是幂等的。
401 Unauthorized - [*]：表示用户没有权限（令牌、用户名、密码错误）。
403 Forbidden - [*] 表示用户得到授权（与401错误相对），但是访问是被禁止的。
404 NOT FOUND - [*]：用户发出的请求针对的是不存在的记录，服务器没有进行操作，该操作是幂等的。
406 Not Acceptable - [GET]：用户请求的格式不可得（比如用户请求JSON格式，但是只有XML格式）。
410 Gone -[GET]：用户请求的资源被永久删除，且不会再得到的。
422 Unprocesable entity - [POST/PUT/PATCH] 当创建一个对象时，发生一个验证错误。
500 INTERNAL SERVER ERROR - [*]：服务器发生错误，用户将无法判断发出的请求是否成功。
```

## Error Handling
For 4xx status code, always response with error message.
```
{
    "error": "Invalid API key"
}
```

## Reference
- https://www.cnblogs.com/bigsai/p/14099154.html