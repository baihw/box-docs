# 业务编码

统一的业务编码处理组件，提供国际化支持（非默认，需配置开启）。

业务编码默认的配置文件名为 ***biz_code.properties***，配置文件中的配置值可覆盖业务编码初始化器中提供的默认值，配置文件示例如下：
```properties
S0000001=系统异常，请跟管理员联系
S0000002=系统异常，信息：{0} param：{1}  param：{2}
```

## 自定义自己的业务编码
自定义的业务编码必须是**枚举类型**，且**实现IBizcode接口**才能被添加到业务编码管理器中。
类名称建议**统一使用BizCode前缀**，便于使用者在IDE中进行自动提示，如BizCodeSys，BizCodeUser，...，这样不用刻意查找文档，只要在IDE中输入BizCode就可诱导出各模块的自定义编码。如下：

```java
import com.wee0.box.code.IBizCode;
public enum BizCodeTest implements IBizCode{
    SystemError("S0000001"),
    SystemErrorInfo("S0000002");
    
    private final String CODE;
    BizCodeTest(String code) { this.CODE = code; }
    @Override
    public String getCode() { return this.CODE; }
}
```
如果有提供默认值，则可以提供一个实现了IBizCodeInitializer接口的对象。如下：
```java
import com.wee0.box.code.IBizCodeInitializer;
public class BizCodeTestInitializer implements IBizCodeInitializer{
    @Override
    public void initialize(IBizCodeSetter setter) {
        setter.set(BizCodeTest.SystemError, "系统异常，请跟管理员联系");
        setter.set(BizCodeTest.SystemErrorInfo, "系统异常，信息：{0} param：{1}  param：{2}");
    }
}
```
也可以合并在同一个枚举类中完成。如下：
```java
public enum BizCodeTest implements IBizCode, IBizCodeInitializer{
    SystemError("S0000001"),
    SystemErrorInfo("S0000002");
    
    private final String CODE;
    BizCodeTest(String code) { this.CODE = code; }
    @Override
    public String getCode() { return this.CODE; }
    @Override
    public void initialize(IBizCodeSetter setter) {
        setter.set(SystemError, "系统异常，请跟管理员联系");
        setter.set(SystemErrorInfo, "系统异常，信息：{0} param：{1}  param：{2}");
    }
}
```
在Java代码中注册自定义业务枚举类和初始化器
```java
import com.wee0.box.code.BizCodeManager;
public class App{
    public void init(){
       BizCodeManager.impl().addBizCodeEnum(BizCodeTest.class); 
       // 如果枚举类同时实现了IBizCodeInitializer接口，则不需要重复注册初始化器。
       BizCodeManager.impl().addBizCodeInitializer(new BizCodeTestInitializer());
    }
}
```
在框架配置文件***box_config.properties***中注册自定义业务枚举类和初始化器
```properties
# 业务编码枚举类，如果有多个使用逗号（,）隔开。
bizCodeManager.bizCodeEnums=test.BizCodeTest
# 业务编码默认值初始化器，如果有多个使用逗号（,）隔开。如果枚举类同时也是初始化器，不用重复配置。
bizCodeManager.bizCodeInitializers=test.BizCodeTestInitializer
```

### 使用

为了方便使用，在box-api库中提供了快捷入口类BizCodeManager，使用此类可进行常用方法的操作。如下：
```java
import com.wee0.box.code.BizCodeManager;
import org.junit.Test;
import org.junit.Assert;
public class AppTest{
    @Test
    public void testGetCodeInfo() {
       IBizCodeInfo _bizCodeInfo = BizCodeManager.impl().getCodeInfo(BizCodeTest.SystemErrorInfo, "p1", "p2", "p3"); 
       Assert.assertNotNull(_bizCodeInfo);
       Assert.assertEquals("系统异常，信息：{0} param：{1}  param：{2}", _bizCodeInfo.getText());
       Assert.assertEquals("系统异常，信息：p1 param：p2  param：p3", _bizCodeInfo.formatText());
       Assert.assertEquals("系统异常，信息：p2 param：p1  param：p0", _bizCodeInfo.formatText("p2", "p1", "p0"));
       Assert.assertEquals("系统异常，信息：p1 param：{1}  param：{2}", _bizCodeInfo.formatText("p1"));
    }
}
```

#### 业务编码在处理业务异常中的应用

```java
import com.wee0.box.exception.BizExceptionFactory;
public class BizExceptionFactoryTest{
    public void ex1(){
        throw BizExceptionFactory.create(BizCodeTest.SystemError);
    }
    public void ex2(){
        throw BizExceptionFactory.create(BizCodeTest.SystemErrorInfo, "信息参数1", "信息参数2", "信息参数3");
    }
}
```

## 国际化支持

在框架配置文件***box_config.properties***中设置使用支持国际化的业务编码存储对象：

```properties
# 编码存储对象：不配置默认为simpleStore，如果需要国际化支持，则配置为i18nStore。
bizCodeManager.bizCodeStore=i18nStore
```

提供对应语言的业务编码配置文件，当前版本支持中文简体、中文繁体、英文。

- config/biz_code_zh_CN.properties
- config/biz_code_zh_TW.properties
- config/biz_code_en.properties

使用方式与前边介绍的非国际化场景的使用方式一样，只是在需要切换语言的地方增加一个语言切换的调用，默认语言为简体中文。示例代码如下：

```java
import com.wee0.box.i18n.Language;
import com.wee0.box.i18n.Locale;
import com.wee0.box.code.BizCodeManager;
import org.junit.Test;
import org.junit.Assert;
public class AppTest{
    @Test
    public void testGetCodeInfo() {
       IBizCodeInfo _bizCodeInfo;
       Locale.impl().setLanguage(Language.en); 
       _bizCodeInfo = BizCodeManager.impl().getCodeInfo(BizCodeTest.SystemError); 
       Assert.assertEquals("system exception", _bizCodeInfo.getText());
       
       Locale.impl().setLanguage(Language.zh_TW); 
       _bizCodeInfo = BizCodeManager.impl().getCodeInfo(BizCodeTest.SystemError); 
       Assert.assertEquals("系統异常", _bizCodeInfo.getText());
        
       Locale.impl().setLanguage(Language.zh_CN); 
       _bizCodeInfo = BizCodeManager.impl().getCodeInfo(BizCodeTest.SystemError); 
       Assert.assertEquals("系统异常", _bizCodeInfo.getText());
    }
}
```

