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


