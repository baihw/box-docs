# 日志

日志组件包：**com.wee0.box.log** 

当前版本采用slf4j消息格式规范，消息文本中使用{}作为参数占位符。

> 示例：log.trace("test p1:{}, p2:{}.", 1, 2 );
> 输出：test p1:1, p2:2.

独立代码中使用方式如下：
```java
import com.wee0.box.log.ILogger;
import com.wee0.box.log.LoggerFactory;
public class LogTest{
    private static ILogger log = LoggerFactory.getLogger(LogTest.class);
    public static void main(String[] args) {
        // 最低级别的跟踪日志
        log.trace("trace..., p1:{}", 1);
        // 开发调试过程中建议使用的调试日志
        log.debug("debug..., p1:{}", 2);
        // 需要向用户展示的日志
        log.info("info..., p1:{}", 3);
        // 警告日志
        log.warn("warn..., p1:{}", 4);
        // 错误日志
        log.error("error..., p1:{}", 5);
    }
}
```