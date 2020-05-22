# OSS云存储插件使用说明

基于阿里云OSS存储文件的云存储插件实现。

## 使用方法

编辑 ***box_config.properties*** 配置文件，配置插件及依赖

```properties
# 插件部分
# 指定使用的插件管理器，默认为：SimpleHttpPluginManager。
# com.wee0.box.plugin.IPluginManager=com.wee0.box.plugin.impl.SimpleHttpPluginManager
# 插件仓库地址
plugin.repository=http://repo.xxx.com/box/plugins
# 依赖的插件集合，多个之间用逗号隔开
plugin.dependencies=storage-oss:0.1.0
# oss访问端点，根据自己开通的地址域名填写
plugin.storage-oss.endpoint=oss-cn-shanghai.aliyuncs.com
# oss访问密钥
plugin.storage-oss.accessKey=xx
plugin.storage-oss.secretKey=xx
# 默认操作的存储桶
plugin.storage-oss.bucket=test1
# 自定义访问域名
plugin.storage-oss.domain=http://upload.xxx.com
```

### 常用操作

```java
import com.wee0.box.plugin.PluginManager;
import com.wee0.box.plugins.storage.ICloudStoragePlugin;

public class Test1 {
    public static void main(String[] args) {
        ICloudStoragePlugin _storage = PluginManager.getPlugin(ICloudStoragePlugin.class);
        
        // 判断文件是否存在
        _storage.exists("test/1.jpg");
        // 获取公开文件“test/start.jpg”的公开访问地址
        _storage.getPublicUrl("test/start.jpg");
        // 获取私有文件“private/start.jpg”的授权访问地址，地址有效时间300秒。
        _storage.getDownloadUrl("private/start.jpg", 300);
        // 复制文件
        _storage.copy("test/1.jpg","test/1copy.jpg");
        // 删除文件
        _storage.remove("test/1copy.jpg");
        // 获取指定文件的PUT方式授权上传地址，地址有效时间300秒。
        _storage.getUploadUrl("test/1.jpg", 300, "image/jpeg");
        // 获取指定文件的POST方式表单上传需要携带的相关参数，有效时间300秒。
        _storage.getUploadForm("test/1.jpg", 300, "image/jpeg");
    }
}
```



### web端文件直传

1. 请求后端接口，获取POST方式表单上传需要携带的相关参数。

2. 构建Form表单对象，上传文件。

   ```html
   <!DOCTYPE html>
   <html>
       <head>
           <meta charset='UTF-8'>
           <title>OSS Upload Demo</title>
       </head>
   	<body>
           <div>
               <input type="file" id="f1" accept="image/*" />
               <button id="btn_upload">Upload</button>
           </div>
           <script src="http://code.jquery.com/jquery-3.5.1.min.js"></script>
           <script type="text/javascript">
               $(function(){
   			$( "#btn_upload" ).on( "click", function(){
   				const _file = $('#f1')[0].files[0] ;
   				// 根据业务规则设置文件上传后的唯一标识，这里直接使用文件名。
   				const _fileId = file.name;
                   const _fileType = file.type;
                   // 请求后端获取构建form上传所需的授权参数。
   				$.get(`/api/examples/getUploadFormData?fid=${_fileId}`, ( res ) => {
   					console.log( "res" + res );
   					let _fd = new FormData();
   					_fd.append( 'OSSAccessKeyId', res.ak );
   					_fd.append( 'policy', res.policy );
   					_fd.append( 'signature', res.signature );
   					_fd.append( 'key', res.fid );
   					_fd.append( 'file', _file );
   					_fd.append( 'Content-Type', _fileType );
                       // 让服务端返回200,不然默认会返回204
   					_fd.append('success_action_status','200');
   					let _xhr = new XMLHttpRequest();
   					_xhr.open('POST', res.url, true);
   					_xhr.send(fd);
   					_xhr.onload = () => {
                           console.log('status:' + _xhr.status);
   					};
   				} );
   			} );
   		});
           </script>
       </body>
   </html>
   ```

   