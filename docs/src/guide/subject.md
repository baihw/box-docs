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
boxJdbcRealm = com.wee0.box.subject.shiro.BoxJdbcRealm
boxJdbcRealm.queryUser = select id from sys_user where user_name=? and password= ?
boxJdbcRealm.queryRole = select a.role_name from sys_role a join sys_user_role_rel b on a.id=b.role_id where b.user_id=?
boxJdbcRealm.queryPermission = select a.permission_code from sys_permission a join sys_role_permission_rel b on a.id=b.permission_id join sys_user_role_rel c on c.role_id=b.role_id where c.user_id=?
securityManager.realms = $boxJdbcRealm

# 基于缓存的会话数据访问对象
sessionDAO = org.apache.shiro.session.mgt.eis.EnterpriseCacheSessionDAO
securityManager.sessionManager.sessionDAO = $sessionDAO
# 会话标识生成器
sessionIdGenerator = com.wee0.box.subject.shiro.BoxSessionIdGenerator
securityManager.sessionManager.sessionDAO.sessionIdGenerator = $sessionIdGenerator
# 3,600,000 milliseconds = 1 hour
securityManager.sessionManager.globalSessionTimeout = 3600000

cacheManager = com.wee0.box.subject.shiro.BoxCacheManager
securityManager.cacheManager = $cacheManager
```

- BoxJdbcRealm - 通过关系型数据库管理用户、角色、权限信息的认证实现
- boxJdbcRealm.queryUser - 配置进行登陆认证时执行的数据库查询语句，接收2个参数：登陆标识、登陆密码，返回一个用户唯一标识。可按照此规则根据项目数据库设计来修改此语句。
- boxJdbcRealm.queryRole - 配置查询登陆用户角色名称集合的语句，接收1个参数：用户登陆成功后上边语句返回的用户唯一标识 ，返回一组角色名称集合。如：admin，guest，...。
- boxJdbcRealm.queryPermission - 配置查询登陆用户权限名称集合的语句，接收1个参数：用户登陆成功后上边语句返回的用户唯一标识 ，返回一组权限名称集合。如：common_read，common_write，...。
- BoxCacheManager - 基于框架缓存组件适配的shiro会话数据管理数据存储组件。当框架使用redis做为缓存实现时，可实现多节点共享会话数据功能。

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


