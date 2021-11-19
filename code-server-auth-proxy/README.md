# code-server-auth-proxy

Securing Visual Studio [code-server][], support multi-user.

为 [code-server][]（VS Code Web 版） 增加外部认证，并支持多用户，不同用户的 code-server 实例完全隔离。

主要为了解决问题：

1. code-server 本身支持配置文件形式的用户名密码认证（截止目前，以后也许会改进）。所以引入了外部认证系统，Google、GitHub、 okta、CAS、Keycloak 等理论上都是支持的。

2. code-server 默认没有数据隔离，所以又加了一层 proxy，为每个用户创建一个（或多个）code-server 实例，以实现用户间的数据隔离。

3. 使用开源 Auth Proxy，自己不需要实现复杂的登录流程了，比如 `code flow with pkce` 对大部分人来说读懂这个协议都很困难。

## 使用组件

- [keycloak][]

  Redhat 开源 IAM 系统，提供用户、组织服务，提供标准 OIDC。

- [oauth2-proxy][]

  认证代理，配合 keycloak 提供完整 OAuth2 Code Flow 认证流程。也可以试试 [pomerium][]，看样子也不错。

架构图如下。

![code-server-auth-proxy](https://www.plantuml.com/plantuml/proxy?cache=no&src=https://raw.githubusercontent.com/l10178/architecture-diagram/main/code-server-auth-proxy/code-server-auth-proxy.puml)

[code-server]: https://github.com/cdr/code-server
[keycloak]: https://github.com/keycloak/keycloak
[oauth2-proxy]: https://github.com/oauth2-proxy/oauth2-proxy
[pomerium]: https://github.com/pomerium/pomerium
