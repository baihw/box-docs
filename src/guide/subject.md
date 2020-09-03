# 认证鉴权

当前版本的认证鉴权组件时通过shiro实现的，所以使用此组件需要增加shiro依赖，并增加shiro.ini配置。

```xml
<dependency>
      <groupId>org.apache.shiro</groupId>
      <artifactId>shiro-core</artifactId>
</dependency>
```
shiro.ini参考配置：
```ini
[main]
# boxToken管理对象
boxTokanManager = com.wee0.box.subject.shiro.BoxTokenManager
# 8位token密钥。
boxTokanManager.tokenSecret = 1234567x
# token最大生存时间，2,592,000,000 milliseconds = 30 day
boxTokanManager.tokenMaxLifeTime = 2592000000
# token最大空闲时间，3,600,000 milliseconds = 1 hour
# 当token超过最大生存时间之后，如果处于活跃状态，则可以继续使用到满足最大空闲时间之后失效。
boxTokanManager.tokenMaxIdleTime = 3600000
#
# 微信登陆认证
weiXinRealm = com.wee0.box.subject.shiro.BoxWeiXinRealm
# appId, secret替换为自己在微信开放平台申请到的密钥对
weiXinRealm.appId = xx
weiXinRealm.secret = xx
# 使用的boxToken管理器
weiXinRealm.boxTokenManager = $boxTokanManager
# 配置根据微信返回的用户标识查询系统用户标识的语句，接收1个参数。
weiXinRealm.queryUser1 = select id from sys_user where wx_unionId= ?
# jdbc认证，综合了常见的登陆需求：验证码登陆，账号密码登陆，邮箱密码登陆等场景。
# PC浏览器访问时，默认关闭浏览器清除认证信息。
# APP访问时，可根据需要调整令牌的有效时间，令牌到期后如果用户正在活跃状态，不会马上失效，可继续使用到超过最大空闲时间后失效。
boxJdbcRealm = com.wee0.box.subject.shiro.BoxJdbcRealm
# 验证码前缀，配置了此项则开启了验证码登陆支持。使用规则如下：
# 当使用手机号+验证码登陆时，将“验证码前缀 + _ + 手机号 + _ + 验证码”作为键名，用户唯一标识作为键值，存入默认缓存中。
# 登陆时使用手机号作为登陆名，验证码作为登陆密码传入参数。
# 如果查询到缓存数据，则直接登陆成功。
# 如果没有查询到缓存数据，则进入后边的登陆信息查询流程。
boxJdbcRealm.queryCodePrefix = BoxQueryCode_
# 使用的boxToken管理器
boxJdbcRealm.boxTokenManager = $boxTokanManager
# 配置根据登陆信息查询用户标识的语句，最多支持5个，用户登陆时从第1个开始依次试之，遇到成功则终止。
boxJdbcRealm.queryUser1 = select id from sys_user where user_name=? and user_pwd= ?
boxJdbcRealm.queryUser2 = select id from sys_user where mobile=? and user_pwd= ?
# boxJdbcRealm.queryUser3 = select id from sys_user where email=? and user_pwd= ?
# boxJdbcRealm.queryUser4 = select id from sys_user where xx=? and user_pwd= ?
# boxJdbcRealm.queryUser5 = select id from sys_user where xxx=? and user_pwd= ?
boxJdbcRealm.queryRole = select a.role_name from sys_role a join sys_user_role_rel b on a.id=b.role_id where b.user_id=?
boxJdbcRealm.queryPermission = select a.permission_code from sys_permission a join sys_role_permission_rel b on a.id=b.permission_id join sys_user_role_rel c on c.role_id=b.role_id where c.user_id=?
# securityManager.realms = $weiXinRealm, $boxJdbcRealm
securityManager.realms = $boxJdbcRealm

cacheManager = com.wee0.box.subject.shiro.BoxCacheManager
securityManager.cacheManager = $cacheManager
securityManager.subjectDAO.sessionStorageEvaluator.sessionStorageEnabled = false
```

- BoxTokenManager - boxToken管理对象，通常共用一个。
- BoxWeiXinRealm - 通过微信登陆的认证实现
- BoxJdbcRealm - 通过关系型数据库管理用户、角色、权限信息的认证实现
- boxJdbcRealm.queryUser1 - 配置进行登陆认证时执行的数据库查询语句，接收2个参数：登陆标识、登陆密码，返回一个用户唯一标识。可按照此规则根据项目数据库设计来修改此语句。当前版本最多支持到queryUser5。
- boxJdbcRealm.queryRole - 配置查询登陆用户角色名称集合的语句，接收1个参数：用户登陆成功后上边语句返回的用户唯一标识 ，返回一组角色名称集合。如：admin，guest，...。
- boxJdbcRealm.queryPermission - 配置查询登陆用户权限名称集合的语句，接收1个参数：用户登陆成功后上边语句返回的用户唯一标识 ，返回一组权限名称集合。如：common_read，common_write，...。
- BoxCacheManager - 基于框架缓存组件适配的shiro缓存数据存储组件。当框架使用redis做为缓存实现时，可实现多节点共享缓存数据功能。

## Java注解

为了方便在Java代码中使用认证鉴权功能，提供了权限与角色检查支持的注解，可以在BoxAction注解的类中使用。

```java
@BoxAction
public class Test {
    // 请求此方法需要用户具备名称为“common_delete”的权限。
    @BoxRequirePermissions("common_delete")
    public Boolean permission1() {}
    
    // 请求此方法需要用户同时具备名称为“common_read”和“common_query”的权限。
    @BoxRequirePermissions(value = {"common_read", "common_query"})
    public Boolean permission2() {}
    
    // 请求此方法需要用户具备名称为“admin”的角色。
    @BoxRequireRoles("admin")
    public Boolean role1() {}

    // 请求此方法需要用户具备名称为“admin”或者“guest”的角色。
    @BoxRequireRoles(value = {"admin", "guest"}, logical = BoxRequireLogical.OR)
    public Boolean role2() {}
    
    // 请求此方法需要用户具备名称为“admin”的角色，并且具备名称为“common_delete”的权限。
    @BoxRequireRoles("admin")
    @BoxRequirePermissions("common_delete")
    public Boolean rolePermission() {}

}
```

其它注解：

- BoxRequireIgnore - 用在默认不允许访问的策略环境中，用来标识不需要用户登陆即可访问。
- BoxRequireUser - 用在默认允许访问的策略环境中，用来标识需要用户登陆后访问。


