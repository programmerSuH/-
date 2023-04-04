# 概述

SpringSecurity是一个功能强大且高度可定制的认证和访问控制框架





# 基本原理介绍

## 过滤器链

FilterSecurityInterceptor：方法级的权限过滤器，基本位于过滤链的最底部

ExceptionTranslationFilter：异常过滤器，用来处理认证授权过程中抛出的异常

UsernamePasswordAuthenticationFilter：对/login请求做拦截





## **接口**

UserDetailsService：查询数据库中的用户名和密码

- 创建继承自UsernamePasswordAuthenticationFilter，重写3个方法
- 创建类实现UserDetailsService，编写数据查询过程，返回User对象



PasswordEncoder：数据加密接口





















