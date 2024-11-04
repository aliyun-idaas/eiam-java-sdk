## IdsManager Server JAVA SDK
### 使用
#### 导入包
idsmanager-eiam-java-sdk-1.0.0.jar

#### 新建Client
```java
Client client = new Client(ClientConfig.builder()
                .host("http://localhost")
                .port("80")
                .build());
```
##### Client参数说明
| 参数 | 类型 | 默认值          | 说明 |
| --- | --- |--------------| --- |
| enableRateLimit | Boolean | false        | 开启限流功能 |
| limitTimesPerSecond | Integer | 300          | 每秒限流次数 (QPS)，最大不能设置超过500/s，目前支持限流的接口见 {@link ApiPathEnum.limitable} |
| enableRetry | Boolean | false        | 开启重试功能 |
| maxAttempts | Integer | 3            | 最大重试次数 |
| delay | Long | 1000         | 重试延迟时间，单位：毫秒 |
| multiplier | Integer |
| 延迟计算时间 |
| enableCache | Boolean | false        | 开启缓存功能，目前支持开启缓存的接口见 {@link ApiPathEnum.cacheable |
| maximumCacheSize | Integer | 2000         | 最大缓存数量 |
| expireAfterWrite | Long | 2400000      | 写入缓存后过期时间，单位：毫秒 |
| host | String |
| IDaas service host |
| port | String |
| IDaas service port |
| httpConnectTimeout | Integer | 5000         | HTTP连接超时时间，单位：毫秒 |
| httpPoolMaxTotal | Integer | 300          | HTTP连接池最大连接数 |
| httpPoolMaxIdle | Integer | 5            | HTTP连接池最大空闲连接数 |
| httpReadTimeout | Integer | 30000        | HTTP读取超时时间，单位：毫秒 |
| httpWriteTimeout | Integer | 30000        | HTTP写入超时时间，单位：毫秒 |
| maxResponseSize | Integer | 1024*1024*20 | 设置最大响应大小，默认20MB |

#### 方法调用
##### 获取AccessToken
说明：所有的权限系统相关的接口，都是受保护的资源。只有通过本接口获取到 access_token，才能继续调用后续接口。
###### 请求参数
| 参数 | 类型 | 必须 | 示例值 | 描述 |
| --- | --- | --- | --- | --- |
| client_id | string | 是 | 78d8be99ef30197c31888f5d2d1390a6pHn71****** | 从权限系统详情中获取到的 AppKey |
| client_secret | string | 是 | 96csUmei1g0tL629ufrVMZviFie7NWBOnGYs****** | 从权限系统详情中获取到的 AppSecret |
| grant_type | string | 是 | 固定值：client_credentials |  |
| scope | string | 是 | 固定值：read |  |

###### **返回参数**
| 参数 | 类型 | 示例值 | 描述 |
| --- | --- | --- | --- |
| access_token | string | bd3a80ca-24c3-4da8-836f-9efcb2c52c4b | 调用接口的令牌 |
| token_type | string | Bearer | 固定值 |
| expires_in | string | 43199 | access_token 过期时间，单位为秒。7200 秒为 2 小时。 |
| scope | Array<String> | ["read"] | 固定值 |

###### 示例
```java
@org.junit.Test
public void testOAuthToken() {
    OAuthTokenRequest request = new OAuthTokenRequest()
            .setClient_id("6abe2e5de7b8e294f73b9bd211bce8445nLWSxxxxx")
            .setClient_secret("9OB2m25jUEQq1A72dWIl2pNigy09ryxjlHxxxxx")
            .setGrant_type("client_credentials")
            .setScope("read");
    OpenApiResponse<OAuthTokenResponse> response = client.getAccessToken(request);
    System.out.println(JsonUtil.toJson(response));
}
```
结果：
```json
{
    "data": {
        "access_token": "AT71828ec3261c1195a72bf5610f9bd71acoXxxxxxx",
        "token_type": "bearer",
        "expires_in": "7199",
        "scope": [
            "read"
        ]
    },
    "success": true,
    "code": "200",
    "timeCost": 1099,
    "isCacheHit": false
}
```
##### 验证AccessToken
说明：验证用户的 accessToken 是否失效或已过期。
###### 请求参数
| 参数 | 类型 | 必须 | 示例值 | 描述 |
| --- | --- | --- | --- | --- |
| accessToken | string | 是 | bd3a80ca-24c3-4da8-836f-9efcb2c52c4b | 用户的 accessToken 值 |

###### **返回参数**
| 参数 | 类型 | 示例值 | 描述 |
| --- | --- | --- | --- |
| errorCode | Int | 错误码 | 错误码：0 代表 accessToken 正常，301：错误的 accessToken， 305：请求参数检查失败 |

###### 示例
```java
@org.junit.Test
public void testValidateAccessToken() {
    OAuthTokenValidationRequest request = new OAuthTokenValidationRequest().setAccessToken(accessToken);
    GlobalHeader globalHeader = new GlobalHeader()
            .setAccess_token(accessToken);
    OpenApiResponse<OAuthTokenValidationResponse> response = client.validateAccessToken(request, globalHeader);
    System.out.println(JsonUtil.toJson(response));
}
```
结果：
```json
{
    "data": {
        "errorCode": 0
    },
    "success": true,
    "code": "200",
    "timeCost": 1095,
    "isCacheHit": false
}
```
##### 获取权限详情
说明：获取权限系统的基本信息。
###### 请求参数
| 参数 | 类型 | 必须 | 示例值 | 描述 |
| --- | --- | --- | --- | --- |
| access_token | string | 是 | bd3a80ca-24c3-4da8-836f-9efcb2c52c4b |  |
| psId | string | 是 | Fgh9sKSS | 系统id，详情处可见 |

###### **返回参数**
| 参数 | 类型 | 示例值 | 描述 |
| --- | --- | --- | --- |
| uuid | string | bd3a80ca-24c3-4da8-836f-9efcb2c52c4b | 权限系统唯一标识 |
| createTime | string | 2019-09-14 13:54 | 权限系统创建时间 |
| archived | boolean | false | 逻辑删除，false代表正常，true代表已删除 |
| psId | string | AbcaPSKkss | 权限系统id |
| name | string | 43199 | 权限系统名称 |
| creator | string | admin | 权限系统创建人 |
| enabled | boolean | true | 是否已启用，true代表启用，false代表禁用 |

###### 示例
```java
@org.junit.Test
public void testGetPsDetail() {
    PSDetailRequest psDetailRequest = new PSDetailRequest()
            .setPsId("ENWnhbxxxP");
    GlobalHeader globalHeader = new GlobalHeader()
            .setAccess_token(accessToken);
    OpenApiResponse<PSDetailResponse> response = client.getPSDetails(psDetailRequest, globalHeader);
    System.out.println(JsonUtil.toJson(response));
}
```
结果：
```json
{
    "data": {
        "uuid": "5ec26796bbf8bae229a6806c99c9743fxxxxx",
        "createTime": "2024-08-05 15:20",
        "archived": false,
        "psId": "ENWnhxxxxP",
        "name": "1",
        "creator": "test",
        "enabled": true
    },
    "success": true,
    "requestId": "1722933247623$8ae927a6-3a0c-652d-6387-fe16c1f25018",
    "code": "200",
    "timeCost": 1143,
    "isCacheHit": false
}
```
##### 获取权限系统的所有角色（分页）
说明：获取指定权限系统下的角色信息。
###### 请求参数
| 参数 | 类型 | 必须 | 示例值 | 描述 |
| --- | --- | --- | --- | --- |
| access_token | string | 是 | bd3a80ca-24c3-4da8-836f-9efcb2c52c4b |  |
| currentPage | int | 否 | 1 | 查询第几页的数据，从 1 开始 |
| pageSize | int | 否 | 10 | 每页查询多少条目，不填写默认10条 |
| psId | String | 是 | hU2czV4pMR | 权限系统id |

###### **返回参数**
| 参数 | 类型 | 示例值 | 描述 |
| --- | --- | --- | --- |
| List | Array | 见下面LIST表 |  |
| currentPage | int | 1 | 当前页数，从 1 开始 |
| totalSize | int | 2 | 总角色数量 |
| pageSize | int | 10 | 一页包含的角色数量 |
| perPageSize | int | 10 | 当前请求时参数返回 |
| psId | string | hU2czV4pMR | 当前 PS 系统 id |

**list 表：**

| 参数 | 类型 | 示例值 | 描述 |
| --- | --- | --- | --- |
| uuid | string | 4c9b9f14e82a5d4978d5ed229ec1ae4d6912xmCXA1l | 角色唯一标识(角色外部id) |
| name | string | 测试角色 | 角色名，同一个权限系统内，角色名是唯一的，最长64字符，不能为空 |
| permissionValue | string | test_role | 角色的权限值 |
| remark | string | admin | 描述 |
| enabled | boolean | true | 是否已启用 |

###### 示例
```java
@org.junit.Test
public void testGetPsAllRoles() {
    PSAllRoleRequest psAllRoleRequest = new PSAllRoleRequest()
            .setPsId("ENWnhbpnWP");
    GlobalHeader globalHeader = new GlobalHeader()
            .setAccess_token(accessToken);
    OpenApiResponse<PSAllRoleResponse> response = client.getPSAllRoles(psAllRoleRequest, globalHeader);
    System.out.println(JsonUtil.toJson(response));
}
```
结果：
```json
{
    "data": {
        "list": [
            {
                "uuid": "8bd64a583d9be3ab9c024fcd3e0acf2cK1hgAbnA95R",
                "name": "测试角色Up7d991ff3-ca79-40db-bd7c-a87284a302b7",
                "permissionValue": "test_role_upc63f2c0f-9d90-402b-ade2-1173f6ce074f",
                "remark": "这是一个测试角色Up",
                "enabled": false
            },
            {
                "uuid": "8bfde287d72b1a32b41c474986752032AvL666UOKpx",
                "name": "测试角色ec2b7e53-0fc8-47e6-81c3-6996e00ce911",
                "permissionValue": "test_role212f18d3-d01f-4dc8-b6b5-340a9252078b",
                "remark": "这是一个测试角色",
                "enabled": true
            }
        ],
        "currentPage": 1,
        "totalSize": 2,
        "pageSize": 10,
        "perPageSize": 0,
        "psId": "ENWnhbpnWP"
    },
    "success": true,
    "requestId": "1722933483441$86cfd8c0-0c47-98cc-7829-cb5c6d4211bf",
    "code": "200",
    "timeCost": 991,
    "isCacheHit": false
}
```
##### 新增角色
说明：向当前的权限系统中新增一个角色。
###### 请求参数
| 参数 | 类型 | 必须 | 示例值 | 描述 |
| --- | --- | --- | --- | --- |
| access_token | string | 是 | bd3a80ca-24c3-4da8-836f-9efcb2c52c4b |  |
| name | string | 是 | 测试角色 | 角色名，同一个权限系统内，角色名是唯一的，最长64字符，不能为空 |
| permissionValue | string | 是 | test_role | 角色的权限值，统一个权限系统内，角色的权限值是唯一的，最长64字符，
只能包含字母数字和“-”或者“_”，不能为空 |
| enabled | boolean | 是 | true | 是否启用角色 |
| psId | String | 是 | hU2czV4pMR | PS系统id |
| remark | String | 否 | 这是一个测试使用的角色 | 角色描述信息，最长255字符，可以为空 |
| clientToken | String | 否 | Kqji1upjakhjdihq3e | IDaaS 幂等机制字段，由调用方自动随机生成，标识一次请求操作。 |

###### **返回参数**
| 参数 | 类型 | 示例值 | 描述 |
| --- | --- | --- | --- |
| roleUuid | string | 447fed833d8739ecb1caf6f38af14e65tthuiDBac88 | 新生成的角色 全局唯一标识 |

###### 示例
```java
@org.junit.Test
public void testCreatePSRole() {
    PSRoleCreateRequest request = new PSRoleCreateRequest()
            .setName("测试角色" + UUID.randomUUID())
            .setPermissionValue("test_role" + UUID.randomUUID())
            .setEnabled(true)
            .setRemark("这是一个测试角色")
            .setPsId("ENWnhbpnWP");
    GlobalHeader globalHeader = new GlobalHeader()
            .setAccess_token(accessToken);
    OpenApiResponse<PSRoleCreateResponse> response = client.createPSRole(request, globalHeader);
    System.out.println(JsonUtil.toJson(response));
}
```
结果：
```json
{
    "data": {
        "roleUuid": "74e98cff081f4a5c6eacc66e64f0b82eMLAeM9tnrPN"
    },
    "success": true,
    "requestId": "1722933589805$0a48b113-ddac-e3d9-bebc-0d717d49453e",
    "code": "200",
    "timeCost": 471,
    "isCacheHit": false
}
```
