# 插件开发

## 工程划分

插件分为两个工程：暴露给使用者可见的规范接口工程与使用者不可直接使用的接口实现工程。通常我们命名为“pluginName-api”与“pluginName-impl”。
接口工程可以在开发过程中加入到项目的依赖中直接引用，实现工程的相关产出与依赖物件放置在项目根目录下的plugins目录下的插件标识名称目录下，不可在开发过程中使用。

## 插件接口工程开发

1. 创建一个普通的打包类型为“jar”的项目，如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.wee0.box.plugins</groupId>
    <artifactId>hello-api</artifactId>
    <version>0.1.0</version>
    <name>${project.artifactId}</name>
    <packaging>jar</packaging>
    <dependencies>
        <dependency>
            <groupId>com.wee0.box</groupId>
            <artifactId>box-api</artifactId>
	        <version>${box.version}</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>
</project>
```
2. 编写插件操作主接口，插件可以定义多个暴露的接口，但是只能有一个主接口，作为统一的入口。
```java
package com.wee0.box.plugins.hello;
import com.wee0.box.plugin.IPlugin;
public interface IHelloPlugin extends IPlugin {
    String hello(String name);
}
```
3. 打包发布后就可以给项目开发人员引入项目中进行开发使用了。


## 插件实现工程开发

1. 创建一个普通的打包类型为“jar”的项目，如下：
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <groupId>com.wee0.box.plugins</groupId>
    <artifactId>hello-impl</artifactId>
    <version>0.1.0</version>
    <name>${project.artifactId}</name>
    <packaging>jar</packaging>
    <dependencies>
        <!-- 引入插件接口依赖 -->
        <dependency>
            <groupId>com.wee0.box.plugins</groupId>
            <artifactId>hello-api</artifactId>
	        <version>0.1.0</version>
            <scope>provided</scope>
        </dependency>
        <!-- 引入其它需要的第三方依赖 -->
    </dependencies>
</project>
```
2. 编写插件规范接口实现逻辑。
```java
package com.wee0.box.plugins.hello.impl;
import com.wee0.box.plugins.hello.IHelloPlugin;
public class HelloImpl implements IHelloPlugin {
    private static ILogger log = LoggerFactory.getLogger(HelloImpl.class);
    
    @Override
    public String hello(String name){
        return "hello" + name;
    }
    
    @Override
    public void init(Map<String, String> params) {
        log.debug("init... params: {}", params);
    }
    
    @Override
    public void destroy() {
        log.debug("destroy...");
    }
}
```
3. 打包项目：`mvn clean package`。
4. 导出项目依赖的所有第三方jar包：`mvn dependency:copy-dependencies -DincludeScope=compile`。

## 在项目中使用上边的插件实现

1. 检查项目根目录下是否存在 “plugins” 目录，如果没有就新建一个。
2. 在 “plugins” 目录下创建插件标识目录，当前版本无强制检查，可根据自己喜好建立，如："hello"。
3. 将插件实现工程的产出jar与 *target/dependency* 目录下导出的所有第三方依赖jar包，拷贝至 “plugins/hello” 目录中。
4. 修改 ***config/box_config.properties*** 配置，启用插件。
```properties
# 插件部分
# 使用的插件标识集合，多个之间用逗号隔开
plugin.ids=hello
# 继承自IPlugin接口的插件主接口完全限定名称
plugin.hello.interface=com.wee0.box.plugins.hello.IHelloPlugin
# 插件主接口实现类完全限定名称
plugin.hello.impl=com.wee0.box.plugins.hello.impl.HelloImpl
plugin.hello.param1=test1
plugin.hello.param2=test2
```
5. 搞定。