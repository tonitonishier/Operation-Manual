# IDaaS开放平台API列表
- [IDaaS开放平台API列表](#idaas开放平台api列表)
	- [开发前必读](#开发前必读)
		- [概述](#概述)
		- [请求根域名](#请求根域名)
		- [授权接口](#授权接口)
	- [通讯录管理](#通讯录管理)
		- [人员管理](#人员管理)
			- [创建人员](#创建人员)
			- [更新人员](#更新人员)
			- [创建管理员](#创建管理员)
			- [获取人员列表/搜索人员](#获取人员列表搜索人员)
			- [通过id批量查询人员](#通过id批量查询人员)
			- [获取人员详情](#获取人员详情)
		- [部门管理](#部门管理)
			- [创建部门](#创建部门)
			- [更新部门](#更新部门)
			- [获取部门列表/搜索部门](#获取部门列表搜索部门)
			- [通过id批量查询部门](#通过id批量查询部门)
			- [获取部门的子部门列表](#获取部门的子部门列表)
			- [获取部门详情](#获取部门详情)
			- [获取部门路径](#获取部门路径)
			- [通过id批量查询部门路径](#通过id批量查询部门路径)
			- [获取根部门](#获取根部门)
			- [获取默认部门](#获取默认部门)
		- [用户组管理](#用户组管理)
			- [创建用户组](#创建用户组)
			- [更新用户组](#更新用户组)
			- [删除用户组](#删除用户组)
			- [获取用户组列表](#获取用户组列表)
			- [获取用户组详情](#获取用户组详情)
			- [获取用户组下成员](#获取用户组下成员)
			- [用户组添加成员](#用户组添加成员)
			- [用户组移除成员](#用户组移除成员)
## 开发前必读

### 概述

通讯录管理相关API接口，可以对人员、部门、用户组信息进行查询、添加、修改、删除等操作。

使用通讯录管理接口，需要使用IDaaS租户级别的ServiceAccount（clientId和privateKey） ，获取租户级别的ServiceAccount的方法目前有两种：

- 使用系统级别ServiceAccount创建IDaaS租户时的返回值中携带租户级别的ServiceAccount相关信息；
- 联系IDaaS实施人员获取租户级别的ServiceAccount；

### 请求根域名

以下为请求API接口的根请求地址，文本中每个API接口的实际请求地址=根请求地址+API接口请求地址，例如创建人员API的请求地址：

https://{domain}-admin.cig.tencentcs.com/contacts/api/v1/users

```
https://{domain}-admin.cig.tencentcs.com/contacts/
```

### 授权接口

在调用本文中的API接口时需要先使用租户级别的ServiceAccount签发JWT，签发的JWT格式如下

header

```
{
  "alg": "RS256",
  "typ": "JWT",
  "kid": "xxxx" //填写ServiceAccount对应的key id
}
```

payload

```
{
  "aud": "contacts",
  "iss": "xxxxx", //填写ServiceAccount对应的clientId
  "account_type": "serviceAccount",
  "iat": 1606752000,
  "exp": 1606752300
}
```

## 通讯录管理

### 人员管理

#### 创建人员

- 请求方式 post 

- 请求地址/api/v1/users

| Header参数详情 |          |                 |             |
| -------------- | -------- | --------------- | ----------- |
| Name           | 是否必填 | 值              | Description |
| Authorization  | 是       | Bearer eyJra*** | admin token |

- 请求体

```
{
	"values": {
		"username": "user@email.com",
		"displayName": "displayName",
		"primaryMail": "user@email.com",
		"secondaryMail": "user@email.com",
		"deptId": "dpf-123",
		"phoneNum": "18311112222",
		"employeeNumber": "123",
	...
	}
}
```

| 请求体字段详情 |          |          |                  |
| -------------- | -------- | -------- | ---------------- |
| Name           | 是否必填 | 数据类型 | Description      |
| username       | 是       | string   | 用户名           |
| displayName    | 是       | string   | 显示名称（姓名） |
| primaryMail    | 是       | string   | 主邮箱           |
| secondaryMail  | 否       | string   | 次邮箱           |
| deptId         | 是       | string   | 所属部门ID       |
| phoneNum       | 否       | string   | 手机号           |
| employeeNumber | 否       | string   | 工号             |

- 请求返回成功

```
HTTP/1.1 201 CREATED
Content-Type: application/json
{
	"id": "us-123",
	"tenantId": "tn-123",
	"modifiedOn": 1556008078864,
	"createdOn": 1536233340177,
	"objectType": "USER",
	"values": {
		"username": "user@email.com",
		"displayName": "displayName",
		"primaryMail": "user@email.com",
		"secondaryMail": "user@email.com",
		"deptId": "dpf-123",
		"phoneNum": "18311112222",
		"status": "ACTIVE",
		"employeeNumber": "123",
	...
	}
}
```

#### 更新人员

- 请求方式 patch

- 请求路径/api/v1/users/{user_id}

| 请求参数详情（Parameters） |          |        |             |
| -------------------------- | -------- | ------ | ----------- |
| Name                       | 是否必填 | 可选值 | Description |
| user_id                    | 是       |        | 人员ID      |

| Header参数详情 |          |                 |             |
| -------------- | -------- | --------------- | ----------- |
| Name           | 是否必填 | 值              | Description |
| Authorization  | 是       | Bearer eyJra*** | admin token |

- 请求体

```
{
	"values": {
		"field": "value"
	...
	}
}
```

| 请求体字段详情 |          |          |                            |
| -------------- | -------- | -------- | -------------------------- |
| Name           | 是否必填 | 数据类型 | Description                |
| values         | 是       | object   | 在values中传入待更新的字段 |

- 请求返回成功：同创建人员接口

#### 创建管理员

- 请求方式 post

- 请求地址/api/v1/users

| 请求参数详情（Parameters） |          |        |             |
| -------------------------- | -------- | ------ | ----------- |
| Name                       | 是否必填 | 可选值 | Description |
| is_admin                   | 是       | true   | 创建admin   |

| Header参数详情 |          |                 |             |
| -------------- | -------- | --------------- | ----------- |
| Name           | 是否必填 | 值              | Description |
| Authorization  | 是       | Bearer eyJra*** | admin token |

- 请求体：

```
{
	"values": {
		"username": "user@email.com",
		"displayName": "displayName",
		"primaryMail": "user@email.com",
		"secondaryMail": "user@email.com",
		"deptId": "dpf-123",
		"phoneNum": "18311112222",
		"employeeNumber": "123",
		"password": "password",
	...
	}
}
```

| 请求体字段详情 |          |          |             |
| -------------- | -------- | -------- | ----------- |
| Name           | 是否必填 | 数据类型 | Description |
| password       | 是       | string   | 密码        |

- 请求返回成功：同创建人员接口

#### 获取人员列表/搜索人员

- 请求方式 get

- 请求地址/api/v1/users

| 请求参数详情（Parameters） |          |              |           |                                                              |                          |
| -------------------------- | -------- | ------------ | --------- | ------------------------------------------------------------ | ------------------------ |
| Name                       | 是否必填 | 可选值       | 默认值    | Description                                                  | 备注                     |
| keywords                   | 否       |              |           | 搜索关键词，支持username、displayName（包括拼音）、primaryMail、phoneNum、employeeNumber字段的搜索 |                          |
| status                     | 否       | 参考人员状态 |           | 通过人员状态筛选                                             |                          |
| dept_id                    | 否       |              |           | 通过人员所属部门搜索                                         |                          |
| deep_search                | 否       | true/false   | false     | 是否搜索包括子部门的人员（前提是dept_id不为空，否则此参数无效） | 仅当keywords不为空时生效 |
| start_time                 | 否       | 毫秒时间戳   |           | 指定人员的更新时间范围，通过更新时间范围筛选                 | 仅当keywords为空时生效   |
| end_time                   | 否       | 毫秒时间戳   |           |                                                              |                          |
| page_size                  | 否       |              | 20        | 分页大小                                                     |                          |
| order_by                   | 否       |              | createdOn | 排序字段                                                     |                          |

| Header参数详情 |          |                 |             |
| -------------- | -------- | --------------- | ----------- |
| Name           | 是否必填 | 值              | Description |
| Authorization  | 是       | Bearer eyJra*** | admin token |

- 请求返回成功

```
HTTP/1.1 200 OK
Content-Type: application/json
{
	"total": 1,
	"data": [{
		"id": "us-123",
		"tenantId": "tn-123",
		"modifiedOn": 1556008078864,
		"createdOn": 1536233340177,
		"objectType": "USER",
		"values": {
			"username": "user@email.com",
			"displayName": "displayName",
			"primaryMail": "user@email.com",
			"deptId": "dpf-123",
			"phoneNum": "18311112222",
			"status": "ACTIVE"
		}
	}]
}
```

| 返回字段详情 |          |                  |
| ------------ | -------- | ---------------- |
| Name         | 数据类型 | Description      |
| total        | int      | 筛选出的结果总数 |
| data         | array    | 筛选出的分页数据 |

#### 通过id批量查询人员

- 请求方式post

- 请求地址/api/v1/users/query/batch

| Header参数详情 |          |                 |             |
| -------------- | -------- | --------------- | ----------- |
| Name           | 是否必填 | 值              | Description |
| Authorization  | 是       | Bearer eyJra*** | admin token |

- 请求体

```
[
    "us-xxx","us-xxx"
]
```

- 请求成功返回

```
HTTP/1.1 200 OK
Content-Type: application/json
[
  {
	"id": "us-xxx",
	"tenantId": "tn-123",
	"modifiedOn": 1556008078864,
	"createdOn": 1536233340177,
	"objectType": "USER",
	"values": {
		"username": "user@email.com",
		"displayName": "displayName",
		"primaryMail": "user@email.com",
		"secondaryMail": "user@email.com",
		"deptId": "dpf-123",
		"phoneNum": "18311112222",
		"status": "ACTIVE",
		"employeeNumber": "123",
	...
	}
  }
]
```

#### 获取人员详情

- 请求方式 get 

- 请求地址/api/v1/users/{user_id}

| 请求参数详情（Parameters） |          |        |             |
| -------------------------- | -------- | ------ | ----------- |
| Name                       | 是否必填 | 可选值 | Description |
| user_id                    | 是       |        | 人员ID      |

| Header参数详情 |          |                 |             |
| -------------- | -------- | --------------- | ----------- |
| Name           | 是否必填 | 值              | Description |
| Authorization  | 是       | Bearer eyJra*** | admin token |

- 请求返回成功

```
HTTP/1.1 200 OK
Content-Type: application/json
{
	"id": "us-123",
	"tenantId": "tn-123",
	"modifiedOn": 1556008078864,
	"createdOn": 1536233340177,
	"objectType": "USER",
	"values": {
		"username": "user@email.com",
		"displayName": "displayName",
		"primaryMail": "user@email.com",
		"secondaryMail": "user@email.com",
		"deptId": "dpf-123",
		"phoneNum": "18311112222",
		"status": "ACTIVE",
		"employeeNumber": "123",
	...
	}
}
```

| 返回字段详情   |          |                  |
| -------------- | -------- | ---------------- |
| Name           | 数据类型 | Description      |
| id             | string   | 筛选出的结果总数 |
| tenantId       | string   | 筛选出的分页数据 |
| createdOn      | number   | 人员创建时间     |
| modifiedOn     | number   | 人员更新时间     |
| username       | string   | 用户名           |
| displayName    | string   | 显示名称（姓名） |
| primaryMail    | string   | 主邮箱           |
| secondaryMail  | string   | 次邮箱           |
| deptId         | string   | 所属部门的id     |
| phoneNum       | string   | 电话号码         |
| status         | string   | 人员状态         |
| employeeNumber | string   | 工号             |

### 部门管理

#### 创建部门

- 请求方式 post

- 请求地址/api/v1/departments

| Header参数详情 |          |                 |             |
| -------------- | -------- | --------------- | ----------- |
| Name           | 是否必填 | 值              | Description |
| Authorization  | 是       | Bearer eyJra*** | admin token |

- 请求体

```
{
	"values": {
		"name": "deptName",
		"parentId": "parent_dept_id",
		"status": "ENABLED",
		...
	}
}
```

| 请求体字段详情 |          |          |              |
| -------------- | -------- | -------- | ------------ |
| Name           | 是否必填 | 数据类型 | Description  |
| name           | 是       | string   | 部门名称     |
| parentId       | 是       | string   | 指定父部门ID |
| status         | 否       | string   | 部门状态     |

- 请求返回成功

```
HTTP/1.1 201 CREATED
Content-Type: application/json
{
	"id": "dp-123",
	"tenantId": "tn-123",
	"modifiedOn": 1556008083913,
	"createdOn": 1520412948110,
	"objectType": "DEPT",
	"values": {
		"name": "dept name",
		"parentId": "dp-parent",
		"ancestor": ["dp-root", "dp-parent"],
		"status": "ENABLED"
	}
}
```

#### 更新部门

- 请求方式 patch

- 请求路径/api/v1/departments/{department_id}

| 请求参数详情（Parameters） |          |        |             |
| -------------------------- | -------- | ------ | ----------- |
| Name                       | 是否必填 | 可选值 | Description |
| department_id              | 是       |        | 部门ID      |

| Header参数详情 |          |                 |             |
| -------------- | -------- | --------------- | ----------- |
| Name           | 是否必填 | 值              | Description |
| Authorization  | 是       | Bearer eyJra*** | admin token |

- 请求体

```
{
	"values": {
		"field": "value"
	...
	}
}

```

| 请求体字段详情 |          |          |                            |
| -------------- | -------- | -------- | -------------------------- |
| Name           | 是否必填 | 数据类型 | Description                |
| values         | 是       | object   | 在values中传入待更新的字段 |

- 请求返回成功：同创建部门接口

#### 获取部门列表/搜索部门

- 请求方式 get

- 请求地址/api/v1/departments

| 请求参数详情（Parameters） |          |                   |           |                                              |                        |
| -------------------------- | -------- | ----------------- | --------- | -------------------------------------------- | ---------------------- |
| Name                       | 是否必填 | 可选值            | 默认值    | Description                                  | 备注                   |
| keywords                   | 否       |                   |           | 搜索关键词，支持name（包括拼音）字段的搜索   |                        |
| status                     | 否       | ENABLED、DISABLED |           | 通过部门状态筛选                             |                        |
| page_index                 | 否       |                   | 0         | 起始页码                                     |                        |
| page_size                  | 否       |                   | 20        | 分页大小                                     |                        |
| order_by                   | 否       |                   | createdOn | 排序字段                                     |                        |
| start_time                 | 否       | 毫秒时间戳        |           | 指定部门的更新时间范围，通过更新时间范围筛选 | 仅当keywords为空时生效 |
| end_time                   | 否       | 毫秒时间戳        |           |                                              |                        |

| Header参数详情 |          |                 |             |
| -------------- | -------- | --------------- | ----------- |
| Name           | 是否必填 | 值              | Description |
| Authorization  | 是       | Bearer eyJra*** | admin token |

- 请求返回成功

```
HTTP/1.1 200 OK
Content-Type: application/json
{
	"total": 1,
	"data": [{
		"id": "dp-123",
		"tenantId": "tn-123",
		"modifiedOn": 1556008082573,
		"createdOn": 1527811227908,
		"objectType": "DEPT",
		"values": {
			"name": "department name",
			"parentId": "dp-root",
			"status": "ENABLED",
			"ancestor": ["dp-root"],
		},
		"pinyinForName": "dangweizuzhibuIrenliziyuanbu"
	}]
}
```

| 返回字段详情 |          |                  |
| ------------ | -------- | ---------------- |
| Name         | 数据类型 | Description      |
| total        | int      | 筛选出的结果总数 |
| data         | array    | 筛选出的分页数据 |

#### 通过id批量查询部门

- 请求方式 post

- 请求地址/api/v1/departments/query/batch

| Header参数详情 |          |                 |             |
| -------------- | -------- | --------------- | ----------- |
| Name           | 是否必填 | 值              | Description |
| Authorization  | 是       | Bearer eyJra*** | admin token |

- 请求体

```
[
    "dp-xxx","dp-xxx"
]
```

- 请求成功返回

```
HTTP/1.1 200 OK
Content-Type: application/json
[
  {
	"id": "dp-xxx",
	"tenantId": "tn-123",
	"modifiedOn": 1556008078864,
	"createdOn": 1536233340177,
	"objectType": "DEPT",
	"values": {	...
	}
  }
]
```

#### 获取部门的子部门列表

- 请求方式 get

- 请求地址/api/v1/departments/{department_id}/children

| 请求参数详情（Parameters） |          |                   |           |                                     |
| -------------------------- | -------- | ----------------- | --------- | ----------------------------------- |
| Name                       | 是否必填 | 可选值            | 默认值    | Description                         |
| status                     | 否       | ENABLED、DISABLED |           | 通过部门状态筛选                    |
| depth                      | 否       |                   | 1         | 指定层级深度（1代表返回一级子部门） |
| page_index                 | 否       |                   | 0         | 起始页码                            |
| page_size                  | 否       |                   | 20        | 分页大小                            |
| order_by                   | 否       |                   | createdOn | 排序字段                            |

| Header参数详情 |          |                 |             |
| -------------- | -------- | --------------- | ----------- |
| Name           | 是否必填 | 值              | Description |
| Authorization  | 是       | Bearer eyJra*** | admin token |

- 请求返回成功

```
HTTP/1.1 200 OK
Content-Type: application/json
[{
		"id": "dp-acc2859b006f43118856a2c08210e05c",
		"tenantId": "tn-123",
		"modifiedOn": 1556008081371,
		"createdOn": 1527811228965,
		"objectType": "DEPT",
		"values": {
			"ancestor": [
				"dp-2b363e466e7347fca70b81b2e626fba0",
				"dp-9636dfcc80b548618977cefbd110341e"
			],
			"name": "department1",
			"parentId": "dp-9636dfcc80b548618977cefbd110341e",
			"status": "ENABLED"
		}
	},
	{
		"id": "dp-6fbaf6b74a754f6989b8a670c114637d",
		"tenantId": "tn-123",
		"modifiedOn": 1556008081495,
		"createdOn": 1527811228838,
		"objectType": "DEPT",
		"values": {
			"ancestor": [
				"dp-2b363e466e7347fca70b81b2e626fba0",
				"dp-9636dfcc80b548618977cefbd110341e"
			],
			"name": "department2",
			"parentId": "dp-9636dfcc80b548618977cefbd110341e",
			"status": "ENABLED"
		}
	}
]
```

#### 获取部门详情

- 请求方式 get

- 请求地址/api/v1/departments/{department_id}

| 请求参数详情（Parameters） |          |        |             |
| -------------------------- | -------- | ------ | ----------- |
| Name                       | 是否必填 | 可选值 | Description |
| department_id              | 是       |        | 部门ID      |

| Header参数详情 |          |                 |             |
| -------------- | -------- | --------------- | ----------- |
| Name           | 是否必填 | 值              | Description |
| Authorization  | 是       | Bearer eyJra*** | admin token |

- 请求返回成功

```
HTTP/1.1 200 OK
Content-Type: application/json
{
	"id": "dp-123",
	"tenantId": "tn-123",
	"modifiedOn": 1556008083913,
	"createdOn": 1520412948110,
	"objectType": "DEPT",
	"values": {
		"name": "dept name",
		"parentId": "dp-parent",
		"ancestor": ["dp-root", "dp-parent"],
		"status": "ENABLED"
	}
}
```

| 返回字段详情 |          |                              |
| ------------ | -------- | ---------------------------- |
| Name         | 数据类型 | Description                  |
| id           | string   | 部门ID                       |
| createdOn    | number   | 创建时间                     |
| modifiedOn   | number   | 更新时间                     |
| name         | string   | 部门名称                     |
| parentId     | string   | 父部门ID                     |
| ancestor     | array    | 此部门的父级部门（部门路径） |
| status       | string   | 部门状态                     |

#### 获取部门路径

- 请求方式 get 

- 请求地址/api/v1/departments/{department_id}/traces

| 请求参数详情（Parameters） |          |        |             |
| -------------------------- | -------- | ------ | ----------- |
| Name                       | 是否必填 | 可选值 | Description |
| department_id              | 是       |        | 部门ID      |

| Header参数详情 |          |                 |             |
| -------------- | -------- | --------------- | ----------- |
| Name           | 是否必填 | 值              | Description |
| Authorization  | 是       | Bearer eyJra*** | admin token |

- 请求返回成功

```
HTTP/1.1 200 OK
Content-Type: application/json
[{
		"id": "dp-eb3f6b7792b0470a9d023e858dc0a982",
		"tenantId": "tn-123",
		"modifiedOn": 1582432469199,
		"createdOn": 1582432345165,
		"objectType": "DEPT",
		"values": {
			"ancestor": [
				"dp-2b363e466e7347fca70b81b2e626fba0",
				"dp-3f269cf1fe634f00bebfd4174136ef1a"
			],
			"name": "department3",
			"parentId": "dp-3f269cf1fe634f00bebfd4174136ef1a",
			"status": "ENABLED"
		}
	},
	{
		"id": "dp-3f269cf1fe634f00bebfd4174136ef1a",
		"tenantId": "tn-123",
		"modifiedOn": 1582431300273,
		"createdOn": 1531907698035,
		"objectType": "DEPT",
		"values": {
			"ancestor": [
				"dp-2b363e466e7347fca70b81b2e626fba0"
			],
			"name": "department2",
			"parentId": "dp-2b363e466e7347fca70b81b2e626fba0",
			"status": "ENABLED"
		}
	},
	{
		"id": "dp-2b363e466e7347fca70b81b2e626fba0",
		"tenantId": "tn-123",
		"modifiedOn": 1556008083913,
		"createdOn": 1520412948110,
		"objectType": "DEPT",
		"values": {
			"ancestor": [],
			"name": "department1",
			"parentId": null,
			"status": "ENABLED"
		}
	}
]
```

#### 通过id批量查询部门路径

- 请求方式 post  

- 请求地址/api/v1/departments/query_trace/batch

| Header参数详情 |          |                 |             |
| -------------- | -------- | --------------- | ----------- |
| Name           | 是否必填 | 值              | Description |
| Authorization  | 是       | Bearer eyJra*** | admin token |

- 请求体

```
[
    "dp-eb3f6b7792b0470a9d023e858dc0a982"
]
```

- 请求返回成功

```
HTTP/1.1 200 OK
Content-Type: application/json
{
    "dp-eb3f6b7792b0470a9d023e858dc0a982":[
        {
            "id":"dp-eb3f6b7792b0470a9d023e858dc0a982",
            "tenantId":"tn-123",
            "modifiedOn":1582432469199,
            "createdOn":1582432345165,
            "objectType":"DEPT",
            "values":{
                "ancestor":[
                    "dp-2b363e466e7347fca70b81b2e626fba0",
                    "dp-3f269cf1fe634f00bebfd4174136ef1a"
                ],
                "name":"department3",
                "parentId":"dp-3f269cf1fe634f00bebfd4174136ef1a",
                "status":"ENABLED"
            }
        },
        {
            "id":"dp-3f269cf1fe634f00bebfd4174136ef1a",
            "tenantId":"tn-123",
            "modifiedOn":1582431300273,
            "createdOn":1531907698035,
            "objectType":"DEPT",
            "values":{
                "ancestor":[
                    "dp-2b363e466e7347fca70b81b2e626fba0"
                ],
                "name":"department2",
                "parentId":"dp-2b363e466e7347fca70b81b2e626fba0",
                "status":"ENABLED"
            }
        },
        {
            "id":"dp-2b363e466e7347fca70b81b2e626fba0",
            "tenantId":"tn-123",
            "modifiedOn":1556008083913,
            "createdOn":1520412948110,
            "objectType":"DEPT",
            "values":{
                "ancestor":[

                ],
                "name":"department1",
                "parentId":null,
                "status":"ENABLED"
            }
        }
    ]
}
```

#### 获取根部门

- 请求方式 get

- 请求地址/api/v1/departments/root

| Header参数详情 |          |                 |             |
| -------------- | -------- | --------------- | ----------- |
| Name           | 是否必填 | 值              | Description |
| Authorization  | 是       | Bearer eyJra*** | admin token |

- 请求返回成功

```
HTTP/1.1 200 OK
Content-Type: application/json
{
	"id": "dp-123",
	"tenantId": "tn-123",
	"modifiedOn": 1556008083913,
	"createdOn": 1520412948110,
	"objectType": "DEPT",
	"values": {
		"name": "root dept name",
		"parentId": null,
		"status": "ENABLED"
	}
}
```

#### 获取默认部门

- 请求方式get

- 请求地址/api/v1/departments/default

| Header参数详情 |          |                 |             |
| -------------- | -------- | --------------- | ----------- |
| Name           | 是否必填 | 值              | Description |
| Authorization  | 是       | Bearer eyJra*** | admin token |

- 请求返回成功

```
HTTP/1.1 200 OK
Content-Type: application/json
{
	"id": "dpf-123",
	"tenantId": "tn-123",
	"modifiedOn": 1574131353144,
	"createdOn": 1574131353144,
	"objectType": "DEPT",
	"values": {
		"name": "Default Department",
		"parentId": null,
		"status": "ENABLED"
	}
}
```



### 用户组管理

#### 创建用户组

- 请求方式post 

- 请求地址/api/v1/groups

| Header参数详情 |          |                 |             |
| -------------- | -------- | --------------- | ----------- |
| Name           | 是否必填 | 值              | Description |
| Authorization  | 是       | Bearer eyJra*** | admin token |

- 请求体

```
{
    "name": "xxxx"
}
```

- 请求成功返回

```
HTTP/1.1 201 Created
Content-Type: application/json

{
    "id": "pe-xxx",
    "tenantId": "tn-xxx",
    "name": "xxxx",
    "type": "USER_GROUP",
    "values": {}
}
```

#### 更新用户组

- 请求方式put

- 请求地址/api/v1/groups/{id}

| 请求参数详情（Parameters） |          |        |             |
| -------------------------- | -------- | ------ | ----------- |
| Name                       | 是否必填 | 默认值 | Description |
| id                         | 是       |        | 用户组id    |

| Header参数详情 |          |                 |             |
| -------------- | -------- | --------------- | ----------- |
| Name           | 是否必填 | 值              | Description |
| Authorization  | 是       | Bearer eyJra*** | admin token |

- 请求体

```
{
    "name": "xxxx"
}
```

- 请求成功返回

```
HTTP/1.1 200 OK
Content-Type: application/json

{
    "id": "pe-xxx",
    "tenantId": "tn-xxx",
    "name": "xxxx",
    "type": "USER_GROUP",
    "values": {}
}
```

#### 删除用户组

- 请求方式delete

- 请求地址/api/v1/groups/{id}

| 请求参数详情（Parameters） |          |        |             |
| -------------------------- | -------- | ------ | ----------- |
| Name                       | 是否必填 | 默认值 | Description |
| id                         | 是       |        | 用户组id    |

| Header参数详情 |          |                 |             |
| -------------- | -------- | --------------- | ----------- |
| Name           | 是否必填 | 值              | Description |
| Authorization  | 是       | Bearer eyJra*** | admin token |

- 请求成功返回

```
HTTP/1.1 200 OK
Content-Type: application/json
```

#### 获取用户组列表

- 请求方式 get

- 请求地址/api/v1/groups

| 请求参数详情（Parameters） |          |        |             |
| -------------------------- | -------- | ------ | ----------- |
| Name                       | 是否必填 | 默认值 | Description |
| page_index                 | 否       | 0      |             |
| page_size                  | 否       | 20     |             |

| Header参数详情 |          |                 |             |
| -------------- | -------- | --------------- | ----------- |
| Name           | 是否必填 | 值              | Description |
| Authorization  | 是       | Bearer eyJra*** | admin token |

- 请求成功返回

```
HTTP/1.1 200 OK
Content-Type: application/json

{
    "total": 1,
    "data": [
        {
            "id": "pe-xxx",
            "name": "xxx",
            "type": "USER_GROUP",
            "values": {}
        }
    ]
}
```

#### 获取用户组详情

- 请求方式get

- 请求地址/api/v1/groups/{id}

| 请求参数详情（Parameters） |          |        |             |
| -------------------------- | -------- | ------ | ----------- |
| Name                       | 是否必填 | 默认值 | Description |
| id                         | 是       |        | 用户组id    |

| Header参数详情 |          |                 |             |
| -------------- | -------- | --------------- | ----------- |
| Name           | 是否必填 | 值              | Description |
| Authorization  | 是       | Bearer eyJra*** | admin token |

- 请求成功返回

```
HTTP/1.1 200 OK
Content-Type: application/json

{
    "id": "pe-xxx",
    "tenantId": "tn-xxx",
    "name": "xxxx",
    "type": "USER_GROUP",
    "values": {}
}
```

#### 获取用户组下成员

- 请求方式 get

- 请求地址/api/v1/groups/{id}/users

| 请求参数详情（Parameters） |          |        |             |
| -------------------------- | -------- | ------ | ----------- |
| Name                       | 是否必填 | 默认值 | Description |
| id                         | 是       |        | 用户组id    |

| Header参数详情 |          |                 |             |
| -------------- | -------- | --------------- | ----------- |
| Name           | 是否必填 | 值              | Description |
| Authorization  | 是       | Bearer eyJra*** | admin token |

- 请求成功返回

```
HTTP/1.1 200 OK
Content-Type: application/json

{
    "total": 1,
    "data": [
        {
            "userId": "us-xxx",
            "username": "xxx",
            "displayName": "xxx",
            "groupId": "pe-xxx"
        }
    ]
}
```

#### 用户组添加成员

- 请求方式 post

- 请求地址/api/v1/groups/{id}/users/add

| 请求参数详情（Parameters） |          |        |             |
| -------------------------- | -------- | ------ | ----------- |
| Name                       | 是否必填 | 默认值 | Description |
| id                         | 是       |        | 用户组id    |

| Header参数详情 |          |                 |             |
| -------------- | -------- | --------------- | ----------- |
| Name           | 是否必填 | 值              | Description |
| Authorization  | 是       | Bearer eyJra*** | admin token |

- 请求体

```
[
   "us-xxx","us-xxx"
]
```

- 请求成功返回

```
HTTP/1.1 200 OK
Content-Type: application/json
```

#### 用户组移除成员

- 请求方式 post 

- 请求地址/api/v1/groups/{id}/users/del

| 请求参数详情（Parameters） |          |        |             |
| -------------------------- | -------- | ------ | ----------- |
| Name                       | 是否必填 | 默认值 | Description |
| id                         | 是       |        | 用户组id    |

| Header参数详情 |          |                 |             |
| -------------- | -------- | --------------- | ----------- |
| Name           | 是否必填 | 值              | Description |
| Authorization  | 是       | Bearer eyJra*** | admin token |

- 请求体

```
[
   "us-xxx","us-xxx"
]
```

- 请求成功返回

```
HTTP/1.1 200 OK
Content-Type: application/json
```

