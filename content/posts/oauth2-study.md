---
title: "OAuth2 Study"
date: 2022-06-23T14:50:02+08:00
draft: true
---

## 什么是 OAuth

OAuth 是一个验证授权的开放标准，任何人都可以根据此标准来实现 OAuth

OAuth 2.0 是目前广泛使用的版本，本文中提及到 OAuth 均为**OAuth 2.0**

### Roles（角色）

- resource owner：资源所有者，能够授予对受保护资源的访问权限的实体，当资源所有者是一个人时，被称为最终用户。
- resource server：托管受保护资源的服务器，能够接收和响应使用访问令牌对受保护资源的请求。
- client：经资源所有者授权后访问受保护资源的应用程序。术语 ”client“ 并不意味着任何特定的实现。
- authorization server：授权服务器可以是与资源服务器相同的服务器，也可以是单独的实体。单个授权服务器可以发布多个资源服务器接受的访问令牌。

### Protocol Flow（协议流）

- 抽象协议流

  ```plaintext
  +--------+                               +---------------+
  |        |--(A)- Authorization Request ->|   Resource    |
  |        |                               |     Owner     |
  |        |<-(B)-- Authorization Grant ---|               |
  |        |                               +---------------+
  |        |
  |        |                               +---------------+
  |        |--(C)-- Authorization Grant -->| Authorization |
  | Client |                               |     Server    |
  |        |<-(D)----- Access Token -------|               |
  |        |                               +---------------+
  |        |
  |        |                               +---------------+
  |        |--(E)----- Access Token ------>|    Resource   |
  |        |                               |     Server    |
  |        |<-(F)--- Protected Resource ---|               |
  +--------+                               +---------------+
  ```

  - A. 客户端向资源所有者请求授权
  - B. 客户端接收到授权许可
  - C. 客户端与授权服务器进行身份认证并通过授权许可获取访问令牌
  - D. 授权服务器对客户端进行身份认证并检查授权许可，如果有效，则颁发访问令牌
  - E. 客户端向资源服务器请求受保护的资源，并提供访问令牌
  - F. 资源服务器验证访问令牌，如果有效，则为其提供服务

### Tokens（令牌）

- access token：访问令牌是用于访问受保护资源的凭据。访问令牌是一个字符串，表示颁发给客户端的授权。该字符串通常对客户端是不透明的。令牌代表特定的访问范围和持续时间，由资源所有者授予，由授权服务器颁发并用于资源服务器验证。
- refresh token：刷新令牌是用于获取访问令牌的凭据。刷新令牌由授权服务器下发给客户端，用于在当前访问令牌失效或过期时获取新的访问令牌，或者获取范围相同或更窄的额外访问令牌（访问令牌的生命周期可能较短，并且权限少于资源所有者授权的权限）。授权服务器可自行决定是否发布刷新令牌。如果授权服务器发出刷新令牌，则在发出访问令牌时会包含它。

### Authorization Grant & Flows（授权许可和模式）

OAuth 有 4 种 模式，分别是：

- Implicit Flow
  简易模式，授权和获取 token 都在 front channel 完成，不支持 refresh token，安全性不高，一般只用在纯前端应用。
- Authorization Code
  授权码模式，授权码模式是四种模式中最复杂，同时也是最安全的授权模式，通过 frontend 获取授权码，然后在 backend 使用授权码去获取 token，广泛用于各种 web 环境。

  ```plaintext
  +----------+
  | Resource |
  |   Owner  |
  |          |
  +----------+
       ^
       |
      (B)
  +----|-----+          Client Identifier      +---------------+
  |         -+----(A)-- & Redirection URI ---->|               |
  |  User-   |                                 | Authorization |
  |  Agent  -+----(B)-- User authenticates --->|     Server    |
  |          |                                 |               |
  |         -+----(C)-- Authorization Code ---<|               |
  +-|----|---+                                 +---------------+
    |    |                                         ^      v
   (A)  (C)                                        |      |
    |    |                                         |      |
    ^    v                                         |      |
  +---------+                                      |      |
  |         |>---(D)-- Authorization Code ---------'      |
  |  Client |          & Redirection URI                  |
  |         |                                             |
  |         |<---(E)----- Access Token -------------------'
  +---------+       (w/ Optional Refresh Token)

  Note: The lines illustrating steps (A), (B), and (C) are broken into
  two parts as they pass through the user-agent.
  ```

- Client Credential Flow
  客户端模式，通常是对整个应用程序进行授权，适用于 server-to-server，有点像使用云服务商的 AK 和 SK 获取云服务商的资源。
- Resource Owner Password Flow
  用户名密码模式，通过客户端输入用户名和密码，不安全，不推荐使用。
