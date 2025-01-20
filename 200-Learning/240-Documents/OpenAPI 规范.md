---
title: OpenAPI 规范
description: 
pubDate: 2025-01-20 18:48
heroImage: https://images.unsplash.com/photo-1735584103272-31c42d9535a0?crop=entropy&cs=srgb&fm=jpg&ixid=M3w2Mjc5MjV8MHwxfHJhbmRvbXx8fHx8fHx8fDE3MzczNzAxMTZ8&ixlib=rb-4.0.3&q=85&w=1200h=400
date created: 2025-01-20 18:48
date modified: 2025-01-20
draft: true
tags:
  - notes
---

# OpenAPI 规范

> [!quote] Notice that the stiffest tree is most easily cracked, while the bamboo or willow survives by bending with the wind.
> — Bruce Lee

第一个正式版本：OpenAPI 3.0（从2015年从Swagger规范重命名为OpenAPI 规范）

规范内所有字段名**小写**
字段分为两种：
1. 固定字段
2. 模式字段

推荐将根OpenAPI文档命名为`openapi.json`或`openapi.yaml`


`null`不是被支持的类型

规范中的`description`被标记为支持`CommonMark markdown`格式。

除非明确指定，所有URL类型的属性值都可以是相对地址

## 固定字段

| 字段名          |            类型            | 描述                                                        |
| :----------- | :----------------------: | :-------------------------------------------------------- |
| openapi      |         `string`         | 必选。用于被工具或客户端解释为OpenAPI文档                                  |
| info         |       [[#Info对象]]        | 必选。提供API相关的元数据。                                           |
| servers      |      [[#Server对象]]       | 一个Server对象的数组，提供到服务器的连接信息。如未提供或为空数组，则默认是url值为`/`的Server对象 |
| paths        |       [[#Paths对象]]       | 必选。对所提供的API有效的路径和操作。                                      |
| components   |    [[#Components对象]]     | 一个包含多种结构的元素。                                              |
| security     | [Security Requirement对象] | 声明API使用的安全机制。                                             |
| tags         |         [Tag对象]          | 提供更多元数据的一系列标签，标签的顺序可以背转换工具用来决定API的顺序。                     |
| externalDocs |   External Documents对象   | 附加的文档。                                                    |
### Info对象

这个对象提供API的元数据。如果客户端需要时可能会用到这些元数据，而且可能会被呈现在编辑工具或者文档生成工具中。

#### 固定字段

| 字段名            |    类型     | 描述                            |
| :------------- | :-------: | :---------------------------- |
| title          | `string`  | 必选。应用的名称                      |
| description    | `string`  | 对应用的简短描述（支持CommonMark syntax） |
| termsOfService | `string`  | 指向服务条款的URL地址，必须是URL地址格式       |
| contact        | Contact对象 | 所开放的API的联系人信息。                |
| license        | License对象 | 所开放的API的证书信息                  |
| version        | `string`  | 必选。API文档的版本信息。                |
#### Info对象示例

```json
{
  "title": "Sample Pet Store App",
  "description": "This is a sample server for a pet store.",
  "termsOfService": "http://example.com/terms/",
  "contact": {
    "name": "API Support",
    "url": "http://www.example.com/support",
    "email": "support@example.com"
  },
  "license": {
    "name": "Apache 2.0",
    "url": "http://www.apache.org/licenses/LICENSE-2.0.html"
  },
  "version": "1.0.1"
}
```

```yaml
title: Sample Pet Store App
description: This is a sample server for a pet store.
termsOfService: http://example.com/terms/
contact:
  name: API Support
  url: http://www.example.com/support
  email: support@example.com
license:
  name: Apache 2.0
  url: http://www.apache.org/licenses/LICENSE-2.0.html
version: 1.0.1
```

### Server对象

表示一个服务器的对象。

#### 固定字段

| 字段名         |                    类型                    | 描述                                |
| :---------- | :--------------------------------------: | :-------------------------------- |
| url         |                 `string`                 | 必选。指向目标主机的URL地址                   |
| description |                 `string`                 | 用来描述此URL地址（支持CommonMark syntax）   |
| variables   | Map\[`string`,  [[#Server Variable对象]]\] | 一组变量和值的映射，这些值被用来替换服务器URL地址内的模板参数。 |
#### Server对象示例

描述多个服务器：
```json
{
  "servers": [
    {
      "url": "https://development.gigantic-server.com/v1",
      "description": "Development server"
    },
    {
      "url": "https://staging.gigantic-server.com/v1",
      "description": "Staging server"
    },
    {
      "url": "https://api.gigantic-server.com/v1",
      "description": "Production server"
    }
  ]
}
```

使用变量配置服务器：
```json
{
  "servers": [
    {
      "url": "https://{username}.gigantic-server.com:{port}/{basePath}",
      "description": "The production API server",
      "variables": {
        "username": {
          "default": "demo",
          "description": "this value is assigned by the service provider, in this example `gigantic-server.com`"
        },
        "port": {
          "enum": [
            "8443",
            "443"
          ],
          "default": "8443"
        },
        "basePath": {
          "default": "v2"
        }
      }
    }
  ]
}
```

#### Server Variable对象

表示可用于服务器URL地址模板变量替换的对象。

##### 固定字段

| 字段名         |      类型      | 描述                             |
| :---------- | :----------: | :----------------------------- |
| enum        | \[`string`\] | 一组可枚举字符串值，当可替换选项只能设置为固定的某些值时使用 |
| default     |   `string`   | 必选。当可替换的值没有被使用者指定时的默认值。        |
| description |   `string`   | 对服务器变量的可选的描述。                  |
### Components对象

包含开发API规范固定的各种可重用组件。当没有被其他对象引用时，在这里定义的组件不会产生任何效果。

#### 固定字段

| 字段名             | 类型                                                | 描述                    |
| :-------------- | :------------------------------------------------ | :-------------------- |
| schemas         | Map\[`string`, Schema对象 \| Reference对象\]          | 定义可重用的Schema对象的对象     |
| response        | Map\[`string`, Response \| Reference对象\]          | 定义可重用的Response对象的对象   |
| parameters      | Map\[`string`, Parameter对象 \| Reference对象\]       | 定义可重用的Parameters对象的对象 |
| examples        | Map\[`string`, Example对象 \| Reference对象\]         |                       |
| requestBodies   | Map\[`string`, Request Body对象 \| Reference对象\]    |                       |
| headers         | Map\[`string`, Header对象 \| Reference对象\]          |                       |
| securitySchemes | Map\[`string`, Security Scheme对象 \| Reference对象\] |                       |
| links           | Map\[`string`, Link对象 \| Reference对象\]            |                       |
| callbacks       | Map\[`string`, Callback对象 \| Reference对象\]        |                       |
上面定义的所有固定字段都是对象，对象包含的key命名必须满足正则表达式：`^[a-zA-Z0-9\.\-_]+$`。
字段名示例：
```
User
User_1
User_Name
user-name
my.org.User
```

### Component对象示例

```json
"components": {
  "schemas": {
    "Category": {
      "type": "object",
      "properties": {
        "id": {
          "type": "integer",
          "format": "int64"
        },
        "name": {
          "type": "string"
        }
      }
    },
    "Tag": {
      "type": "object",
      "properties": {
        "id": {
          "type": "integer",
          "format": "int64"
        },
        "name": {
          "type": "string"
        }
      }
    }
  },
  "parameters": {
    "skipParam": {
      "name": "skip",
      "in": "query",
      "description": "number of items to skip",
      "required": true,
      "schema": {
        "type": "integer",
        "format": "int32"
      }
    },
    "limitParam": {
      "name": "limit",
      "in": "query",
      "description": "max records to return",
      "required": true,
      "schema" : {
        "type": "integer",
        "format": "int32"
      }
    }
  },
  "responses": {
    "NotFound": {
      "description": "Entity not found."
    },
    "IllegalInput": {
      "description": "Illegal input for operation."
    },
    "GeneralError": {
      "description": "General Error",
      "content": {
        "application/json": {
          "schema": {
            "$ref": "#/components/schemas/GeneralError"
          }
        }
      }
    }
  },
  "securitySchemes": {
    "api_key": {
      "type": "apiKey",
      "name": "api_key",
      "in": "header"
    },
    "petstore_auth": {
      "type": "oauth2",
      "flows": {
        "implicit": {
          "authorizationUrl": "http://example.org/api/oauth/dialog",
          "scopes": {
            "write:pets": "modify pets in your account",
            "read:pets": "read your pets"
          }
        }
      }
    }
  }
}
```

### Paths对象

定义各个的端点和操作的相对路径。这里指定的路径会和`Server`对象内指定的URL地址组成完整的URL地址。路径可以为空。

#### 模式字段

|  字段名模式  |        类型        | 描述                                                                                                                               |
| :-----: | :--------------: | :------------------------------------------------------------------------------------------------------------------------------- |
| /{path} | [[#Path Item对象]] | 到各个端点的相对路径，路径必须以`/`开头，这个路径会被直接连接到[[#Server对象]]的`url`字段以形成完整的URL地址（不会考虑是否为相对路径）。这里可以使用Path templating，当作URL地址匹配时，不带路径参数的路径会被优先匹配。 |
##### 路径模板匹配
明确定义的路径（`/pets/mine`）会被优先匹配：
```
/pets/{petId}
  /pets/mine
```

以下路径被认为等价而且是无效的：
```
/pets/{petId}
  /pets/{name}
```

以下路径会产生歧义：
```
/{entity}/me
  /books/{id}
```

##### Path对象示例

```json
{
  "/pets": {
    "get": {
      "description": "Returns all pets from the system that the user has access to",
      "responses": {
        "200": {
          "description": "A list of pets.",
          "content": {
            "application/json": {
              "schema": {
                "type": "array",
                "items": {
                  "$ref": "#/components/schemas/pet"
                }
              }
            }
          }
        }
      }
    }
  }
}
```

#### Path Item对象

描述对一个路径可执行的有效操作。