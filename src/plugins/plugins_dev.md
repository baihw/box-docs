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
3. 补充插件描述文件：“META-INF/box/hello.json”。

   ```json
   {
     "pluginInterface": "com.wee0.box.plugins.hello.IHelloPlugin",
     "pluginImplementation": "com.wee0.box.plugins.hello.impl.HelloImpl"
   }
   ```

   

4. 打包项目：`mvn clean package`。

5. 导出项目依赖的所有第三方jar包：`mvn dependency:copy-dependencies -DoutputDirectory=target/lib   -DincludeScope=compile`。

6. 将项目jar包与所有依赖的第三方jar包，打包到一个命名为“{plguinId}-{pluginVersion}”的zip文件中，如：*hello-0.1.0.zip*。

7. 将zip文件发布到第三方仓库，或者自己搭建的静态文件服务器。

## 在项目中使用上边的插件实现

1. 插件本地存储目录查找规则：如果项目下存在plugins目录，使用项目下的plugins目录；否则使用操作系统中当前登陆用户的家目录下的".box/plugins"目录。
2. 联网环境下，插件会自动下载；但是如果是无网的离线环境中，可以手动放置插件到插件本地存储目录，放置规则为：{plguinId}/{pluginVersion}/{plguinId}-{pluginVersion}.zip。
4. 修改 ***config/box_config.properties*** 配置，启用插件。
```properties
# 插件部分
# 指定使用的插件管理器，默认为：SimpleHttpPluginManager。
# com.wee0.box.plugin.IPluginManager=com.wee0.box.plugin.impl.SimpleHttpPluginManager
# 插件仓库地址，可以修改为自己搭建的静态文件服务器。
plugin.repository=http://repo.wee0.com/box/plugins
# 依赖的插件集合，多个之间用逗号隔开
# plugin.dependencies=office-poi:0.1.0,storage-oss:0.1.0
plugin.dependencies=hello:0.1.0
# 插件自定义参数配置，具体配置方法参考选择的插件使用说明
plugin.hello.param1=xx
plugin.hello.param2=xx
```
5. 搞定。