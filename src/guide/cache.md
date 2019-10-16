# 缓存

缓存组件包：**com.wee0.box.cache** 

当前版本默认使用基于内存的缓存实现，如果需要切换到redis缓存，需要在框架配置文件***box_config.properties***中进行如下配置：

> ```ini
> com.wee0.box.cache.ICacheManager = com.wee0.box.cache.redis.RedisCacheManager
> ```

独立代码中使用方式如下：
```java
import com.wee0.box.cache.CacheManager;
import com.wee0.box.cache.ICache;
import org.junit.*;
public class CacheTest{
    @Test
    public void test(){
        // 获取test命名空间缓存对象
        ICache _cache = CacheManager.impl().getCache("test");
        // 设置缓存，最大生命周期 
        _cache.put("string1", "string1.v");
        // 设置缓存，60秒后过期。
        _cache.put("string2", "string2.v", 60);
        
        Assert.assertEquals("string1.v", cache.get("string1"));
        Assert.assertEquals("string2.v", cache.get("string2"));
        
        // 读取缓存，重设60秒过期时间
        cache.get("string2", 60)
        
        // 清除缓存
        _cache.clear();
    }
}
```



