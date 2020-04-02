# 阿里云短信

基于阿里云短信服务平台的短信通知实现。[官方文档地址](https://help.aliyun.com/document_detail/101414.html?spm=a2c4g.11186623.6.624.35ddbc45pQDjZ6)

## 使用方法

1. 将阿里云短信服务平台申请到的访问密钥等信息录入 ***box_config.properties*** 配置文件

   ```properties
   # 短信操作助手实现者，当前版本默认为阿里云短信服务平台
   # com.wee0.box.notify.sms.ISmsHelper=com.wee0.box.notify.sms.dy.DySmsHelper
   # 短信相关配置，具体支持的配置参数请参考对应的实现者提供的文档说明，当前版本为阿里云短信的配置示例
   # 短信api请求端点
   # sms.endpoint=dysmsapi.aliyuncs.com
   # 接口访问密钥标识
   sms.accessKeyId=testId
   # 接口访问密钥
   sms.accessSecret=testSecret
   # 默认使用的短信发送者署名
   sms.defaultSignName=测试发送者
   # 默认使用的短信模板编码
   sms.defaultTemplateCode=SMS_123456789
   ```

   

2. 在需要发送短信的地方，使用***SmsHelper***进行调用。

   ```java
   @Test
   public void sendSms() {
       // 测试发送短信，具体的参数与返回值说明请查阅官方文档。
       java.util.Map<String, String> _params = new java.util.HashMap<>();
       // 注释掉的部分为默认值，可以不用传。
       // _params.put("Action", "SendSms");
       // _params.put("SignName", "发送者署名");
       // _params.put("TemplateCode", "短信模板编码");
       _params.put("PhoneNumbers", "13xxxxxxxxx");
       _params.put("TemplateParam", "{\"code\":\"123456\"}");
       CMD _result = SmsHelper.impl().sendSms(_params);
       System.out.println("result: " + _result);
       if (_result.isOK()) System.out.println("发送成功！");
   }
   
   @Test
   public void querySendSmsDetail() {
       // 查询短信发送状态，具体的参数与返回值说明请查阅官方文档。
       java.util.Map<String, String> _params = new java.util.HashMap<>();
       // 注释掉的部分为默认值，可以不用传。
       // _params.put("Action", "QuerySendDetails");
       // _params.put("PageSize", "10");
       // _params.put("CurrentPage", "1");
       // _params.put("SendDate", new SimpleDateFormat("yyyyMMdd").format(new Date()));
       _params.put("PhoneNumber", "13xxxxxxxxx");
       CMD _result = SmsHelper.impl().querySendSmsDetail(_params);
       System.out.println("result: " + _result);
   }
   ```