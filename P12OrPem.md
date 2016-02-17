## 推送证书p12、pem文件的制作
#### 关于证书制作网上有很多，此处给出两个链接：
- 目前最新的符合当前官网UI的是 [百度云iOS api 文档](http://push.baidu.com/doc/ios/api "百度云iOS api 文档")
- 比较老的但是很详细的[一步一步教你做ios推送](http://blog.csdn.net/showhilllee/article/details/8631734 "一步一步教你做ios推送")

#### 说明
- 有些第三方平台需要的pem文件而不是p12文件，这样要说一下    asp(微软)用p12 ,  linux 用pem编码 ,  java用der编码。
- 有些时候有些第三方平台无法识别证书，这是因为我们的PushChatKey.pem有密码 ，这时我们必须要用去掉密码的  openssl rsa -in PushChatKey.pem -out key-noenc.pem 通过此命令产生的的key-noenc.pem 为去掉密码后的,然后再通过cat PushChatCert.pem key-noenc.pem > PUSH.pem合成不带密码的pem证书文件
