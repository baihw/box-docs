# 控制层

控制层主要使用的注解为BoxAction，由类上注解与方法注解组合为最终的请求路由。类上注解标记不能省略，但路径可以不指定，方法注解可以省略。默认情况下，标记了BoxAction的类中，所有public方法都会自动映射。当不指定路由路径时，以类名首字母小写后的路径为类路径部分，以方法名为方法路径部分，配合基准路径进行组合。默认的基准路径为："/api"+不包含action包根路径的包名。

## 举个例子

```java
package com.wee0.box.examples.multiModule.action.test;

@BoxAction
public class User {
    public boolean login(String loginId, String loginPwd) {
        ISubject _subject = SubjectContext.getSubject();
        IPasswordToken _passwordToken = SubjectContext.getTokenFactory().createPasswordToken(loginId, loginPwd);
        _subject.login(_passwordToken);
        return true;
    }

    public boolean logout() {
        ISubject _subject = SubjectContext.getSubject();
        _subject.logout();
        return true;
    }

    private void internal() {}
}
```

1. *com.wee0.box.examples.multiModule.action* 为action包根路径。

2. login方法的请求路径为：*/api/test/user/login*。

3. logout方法的请求路径为：*/api/test/user/logout1*。

4. internal方法因为是非public方法，所以不会自动映射。

做些修改，如下：

```java
package com.wee0.box.examples.multiModule.action.test;

@BoxAction("/user")
public class User {
    public boolean login(String loginId, String loginPwd) {
        ISubject _subject = SubjectContext.getSubject();
        IPasswordToken _passwordToken = SubjectContext.getTokenFactory().createPasswordToken(loginId, loginPwd);
        _subject.login(_passwordToken);
        return true;
    }

    @BoxAction("/logout1")
    public boolean logout() {
        ISubject _subject = SubjectContext.getSubject();
        _subject.logout();
        return true;
    }

    private void internal() {}
}
```
1. *com.wee0.box.examples.multiModule.action* 同样为action包根路径。

2. login方法的请求路径为：*/api/user/login*。

3. logout方法的请求路径为：*/api/user/logout1*。

4. internal方法因为是非public方法，所以不会自动映射。

## 统一的响应数据

框架默认响应包含code, message, data三个字段中不为null数据的json格式，如果方法逻辑正常执行得到的返回值将被作为data数据值，code默认为200，message默认为“ok”，可以在配置文件"application.properties"修改，如下：
```properties
# 设置请求处理成功时返回的响应代码。默认：200
box.action.default.resultCode=200
# 设置请求处理成功时返回的响应消息。默认：ok
box.action.default.resultMessage=ok
```
如果方法逻辑未按预期执行，可通过业务异常工厂类抛出表示错误的业务代码与提示消息，如下：
```java
throw BizExceptionFactory.create(BizCodeDef.S000001, "提示消息参数");
```
特殊场景下，如果需要自定义响应数据格式时，仅需要将自定义的响应数据类标记为实现了IStruct接口，然后在业务逻辑方法返回自定义类型即可，如下：
```java
public class CustomResult implements IStruct {
    // ......
}
```

## 文件上传

当前版本的文件上传功能基于 *commons-fileupload* 组件实现，所以使用之前，请先确认正确添加了依赖

```xml
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
</dependency>
```

在启动类上排除springboot对 *commons-fileupload* 组件的自动装配

```java
@SpringBootApplication(exclude = MultipartAutoConfiguration.class)
public class App {
}
```

业务代码中使用上传组件的示例代码如下：

```java
@BoxAction
public class UpDown {
    @BoxInject
    private IUploadRequestUtils uploadRequestUtils;

    public String upload(HttpServletRequest request) {
        IUploadRequest _uploadRequest = uploadRequestUtils.parseRequest(request);
        IUploadFile _file1 = _uploadRequest.getUploadFile("file1");
        String _toFile = "D:/test/" + _file1.getName();
        _file1.saveTo(new File(_toFile));
        return _file;
    }
}
```



