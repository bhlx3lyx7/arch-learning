# Authn & Authz

## differences

| No. | Authentication | Authorization |
| ----- | ----- | ----- |
| 1 | In authentication process, the identity of users are checked for providing the access to the system. | While in authorization process, person’s or user’s authorities are checked for accessing the resources. |
| 2 | In authentication process, users or persons are verified.	| While in this process, users or persons are validated. |
| 3 | It is done before the authorization process. | While this process is done after the authentication process. |
| 4 | It needs usually user’s login details. | While it needs user’s privilege or security levels. |
| 5 | Authentication determines whether the person is user or not. | While it determines What permission do user have? |

## OAuth (Open Authorization)
OAuth is an open standard for access delegation, commonly used as a way for internet users to grant websites or applications access to their information on other websites but without giving them the passwords. 

Essentially, OAuth contains both authn & authz

## SSO (Single Sign-On)
Single sign-on (SSO) is an authentication method that enables users to securely authenticate with multiple applications and websites by using just one set of credentials.

OAuth makes SSO possible.

## References
- Difference between Authentication and Authorization: https://www.geeksforgeeks.org/difference-between-authentication-and-authorization/
- authn vs authz: https://www.cloudflare.com/zh-cn/learning/access-management/authn-vs-authz/
- oauth: https://en.wikipedia.org/wiki/OAuth
- SSO: https://www.onelogin.com/learn/how-single-sign-on-works