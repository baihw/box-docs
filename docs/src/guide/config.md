# 框架配置

框架配置性的文本统一放置在web项目的*src/main/resources*目录下，主要目录文件如下：

- application.properties - springboot主配置文件
- application-dev.properties - springboot开发环境相关信息配置文件
- application-prod.properties - springboot正式环境相关信息配置文件
- config - 不依赖springboot环境的相关配置文件存放目录
  - box_config.properties - 框架配置文件
  - redis.properties - redis配置文件（可选）
 - shiro.ini - shiro配置文件（可选）
- mybatis - mybatis相关文件存放目录（可选）
  - mybatis-config.xml - mybatis配置文件
  - mapper - mapper xml 文件存放目录
- static - 静态文件存放目录（可选）

## 配置对象

配置对象提供了框架中一些可配置组件的定制化功能，这些组件使用Java对象进行配置而不是配置文件，因为这通常是为了配合统一的开发规范，比如前后端约定的数据交互标准等，有定制的需求，但不会在程序开发发布完成后由运维人员调整，没有运行期进行配置更改的必要。

配置对象是一个实现了`com.wee0.box.IBoxConfigObject`接口的类，如下：

```java
public class SimpleBoxConfigObject implements IBoxConfigObject {
    @Override
    public int getBizExceptionHttpStatusCode() {
        return 500;
    }
    @Override
    public int getPermissionExceptionHttpStatusCode() {
        return 403;
    }
    @Override
    public IBizCode getSystemErrorBizCode() {
        return BizCodeDef.S000000;
    }
    @Override
    public IBizCode getSystemErrorInfoBizCode() {
        return BizCodeDef.S000001;
    }
    @Override
    public IBizCode getNeedLoginBizCode() {
        return BizCodeDef.NeedLogin;
    }
    @Override
    public IBizCode getUnauthorizedBizCode() {
        return BizCodeDef.Unauthorized;
    }
    @Override
    public Object getWrappedActionReturnValue(Object returnValue) {
        return CmdFactory.create("200", "ok", returnValue);
    }
}
```

在`box_config.properties`文件中指定，如下：

```properties
configObject=com.wee0.box.impl.SimpleBoxConfigObject
```




