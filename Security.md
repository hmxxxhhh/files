# 安全性接口设计

[参考](http://blog.csdn.net/linlzk/article/details/45536065)

---
## 目录

* [流程](#流程)
* [加密机制](#加密机制)
* [安全性加固](#安全性加固)
* [状态](#状态) 
* [范例](#范例) 

---
## 流程
* 接口请求，都要传递 deviceId 和 user-agent
* 登录： (帐号 + 密码 + 时间戳) + sign 。(登录后，Server产生唯一 token)
* 登录成功后：数据库 token 绑定帐号 。
* 登录成功后：接口请求，(信息 + 时间戳 + token) + sign.

---
## 加密机制

### token
* 产生方式；根据用户的信息或一些随机信息（比如时间戳）再通过（比如md5、sha1等）生成唯一码。
* 以 device 和 user-agent 请求，取得 token ，device 和 user-agent 绑定 token.

* token失效方式：
	- token 设定过期时间，自动过期。 或  过多久时间没有操作过期。
	- 安全登出将 token 清除。
	- 进入背景将 token 清除。

* token 验证机制
	- requst 会带 device 或 user-agent、token，检查是否与server 相同 。
	- token 失效（失效包括 时间过期 或 没有此 token）
		
* token 验证失败流程
	- 则重新登录，取得 token.
	- 或透过 response 更换目前 token.(前提示 token 加密之下)

### 加密方式 
* 数据：一般信息 + 时间戳 + token。
* 自制加密方法： AES(key,IV,数据) 加密。
* key 储存于 APP 和 Server 端，IV值 由 cleint 自动产生。
* 验证机制：
	- 解密后，明文检查时间戳（北京5分钟以内都可以。）
	- token 检查，如以上 token 验证机制。
	
* response，需要 AES解密 、签名验证才算成功。 

### 签名(sign)
* 数据：一般信息 + 时间戳 + token。
* 加密方式：md5 或 sha1。如 md5(数据) 或 sha1(数据) 
* 验证机制：
	- request解密后的明文，md5 或 sha1 加密后相同则表示验证通过。

### 安全性加固
* ip改变就登出。 

---
## 状态

* sucess
* invalid_token
* other error

---
## 范例
* 登录： (帐号 + 密码 + 时间戳) + sign 。(登录后，Server产生唯一 token)
	- 请求 加密方式 + 签名 = AES(帐号 + 密码 + 时间戳) + md5(帐号 + 密码 + 时间戳)
	- 返回 加密方式 + 签名 = AES(response数据 + token) + md5(response数据 + token)	
* 登录成功后：数据库 token 绑定帐号 。

* 登录成功后：接口请求，(信息 + 时间戳 + token) + sign.
	- 请求 加密方式 + 签名 =  AES(信息 + 时间戳 + token) + md5(信息 + 时间戳 + token)
	- 返回 加密方式 + 签名 = AES(response数据) + md5(response数据)


---
## 补充

* Request Token是长时间保存的，Access Token是一次性的是吧
