---
title: "OAuth2 Study"
date: 2022-06-23T14:50:02+08:00
draft: true
---

## 什么是 OAuth

OAuth 是一个验证授权的开放标准，任何人都可以根据此标准来实现 OAuth

OAuth 2.0 是目前广泛使用的版本，本文中提及到 OAuth 均为**OAuth 2.0**

## OAuth 中心组件

- Scopes and Consent
- Actors
- Clients
- Tokens
- Authorization Server
- Flows

### Actors（角色）

- Resource Owner：资源所有者，简单理解就是用户。例如：我有一个微信账户，我的微信账户里面的信息是属于我的
- Resource Server：资源服务器，资源存储的地方
- Client：想要访问用户资源的客户端
- Authorization Server：授权服务器

### Tokens

- Access token：访问密钥，用来向资源服务器请求用户资源，生存期较短
- Refresh token：刷新密钥，用来向认证授权服务器请求新的 Access token

### Flows

- Implicit Flow
- Authorization Code
- Client Credential Flow
- Resource Owner Password Flow
