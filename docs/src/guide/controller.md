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



