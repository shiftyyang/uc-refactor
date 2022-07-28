# UC 现有登录方式

### 1. 独立版登录

#### 1）PC 端

```mermaid
sequenceDiagram
	用户->>UC:输入账密登录
	UC->>UC:登录
	UC->>CRM:选择最近登录（或最新创建）<br>用户，携带 ticket 跳转
	CRM-->>UC:通过 ticket 获取用户 uid
```

**安全校验**：极验、账密、登录失败次数限制

**业务校验**：套餐、授权、禁用状态、用户状态

**排序**：符合条件的用户进行排序，选择最近登录的用户

**跳转**：根据企业套餐选择跳转 CRM 或 进销存

**错误提示**：安全校验失败提示、业务校验失败提示

#### 2）移动端

```mermaid
sequenceDiagram
	用户->>移动端:输入账密
	移动端->>CRM:携带账密
	CRM->>UC:携带账密
	UC-->>CRM:跟 PC 端一样的校验，最终返回 uid
```



#### 3）openApi 登录

```mermaid
sequenceDiagram
	调用方->>UC:携带账密、corpId
	UC-->>调用方:返回 uid
```

#### 4）切换企业登录

```mermaid
sequenceDiagram
	用户->>CRM:点击切换企业按钮
	CRM->>UC:获取该账号企业列表
	UC-->>CRM:返回账号企业列表
	用户->>CRM:选择企业
	CRM->>UC:根据账号访问 uc 后端获取accountId
	UC-->>CRM:返回临时一次性 accountId
	CRM->>UC:携带 accountId 和 oid 登录
	UC->>UC:登录
	UC->>CRM:携带 ticket 跳转 CRM
	CRM->>UC:ticket 换取用户信息
	UC-->>CRM:返回 uid
```



### 2. 企微登录

#### 1）扫码登录

```mermaid
sequenceDiagram
	用户->>励销云官网:点击扫码登录
	励销云官网->>企微:申请扫码页面
	用户->>企微:扫码
	企微-->>励销云官网:返回 code
	励销云官网->>uc:后端调用 uc 扫码登录接口
	uc->>uc:重定向到uc前端（目的是uc登录，写cookie）
	uc->>企微:传code获取用户信息
	企微-->>uc:返回用户 openUserid 
	uc->>uc:uc登录
	uc->>CRM/JXC:跳转 crm/jxc
```

**用户映射关系**：企微 corpid 和 userid --> uid

**安全校验**：uc 后端通过 code 调用企微获取加密用户 openuserid，成功则代表合法

**业务校验**：套餐、授权

**特殊逻辑**：明文转密文 userid、找不到映射用户则新创建用户

#### 2）账密登录

```mermaid
sequenceDiagram
	用户->>移动端:用户选择企微账密登录
	移动端->>CRM:携带账密
	CRM->>UC:携带账密获取用uid
	UC-->>CRM:返回 uid
```

#### 3）免密登录（企微应用内免登、管理员后台免登）

```mermaid
sequenceDiagram
	用户->>企微:点击登录企微励销云APP
	企微->>CRM:颁发code，跳转登录
	CRM->>UC:携带code
	UC->>企微:通过 code 换取用户openUserid
	企微-->>UC:返回 openuserid
	UC-->>CRM:通过 openuserid 找到 uid 返回
	CRM->>UC:调用 uc 后端获取 issueToken
	UC-->>CRM:返回 issueToken
	CRM->>UC:调用 uc 前端登录,写 cookie
	UC->>UC:登录
```

#### 4）自建应用登录（企微助手侧边栏）

```mermaid
sequenceDiagram
	用户->>企微: 点击侧边栏
	企微-->企微助手:颁发 code,跳转登录
	企微助手->>UC:携带 code 换取uid
	UC->>企微:code 换取 userid
	企微-->>UC:返回 userid
	UC-->>企微助手:通过 userid 找到 uid 返回
```

### 3.钉钉登录

#### 免登、扫码登录

```mermaid
sequenceDiagram
	用户->>钉钉:用户应用免登/扫码/管理后台登录
	钉钉->>CRM:颁发 code
	CRM->>UC:携带 code
	UC->>钉钉:code 获取用户信息
	钉钉-->>UC:返回用户 unionid
	UC-->>CRM:通过 unionid 映射到 uid 返回
	CRM->>UC:freeLogin 登录 uc
	UC->>UC:登录
```

### 4. 腾讯云登录

```mermaid
sequenceDiagram
	用户(创建人)->>腾讯云:创建人访问后台免登链接
	腾讯云->>UC:授权后,颁发 code
	UC->>腾讯云:code 获取用户信息
	腾讯云-->>UC:返回 openId
	UC->>UC:隐射到uid,登录
	UC->>CRM:携带 ticket 跳转 CRM
	
```



### 5. 千帆营销通登录

```mermaid
sequenceDiagram
	用户->>营销通:用户企微中点击营销通应用
	营销通->>UC:携带 code
	UC->>营销通:code 换取用户 openid
	营销通-->>UC:返回 openid
	UC->>UC:openid 映射 uid,登录
	UC->>CRM:携带 ticket 登录
```



### 6. WPS 登录

```mermaid
sequenceDiagram
	用户->>WPS:点击励销云应用
	WPS->>UC:颁发 code
	UC-->>UC:校验隐私协议
	UC->>UC:重定向自己登录
	UC->>CRM:携带 ticket 跳转 CRM，<br>登录体验账号/励销云账号
```

