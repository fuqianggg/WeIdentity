
.. _weidentity-rest-api-doc:

WeIdentity RestService API 文档
=================================

1. 总体介绍
------------

具体地，每个API的入参都是满足以下格式的json字符串：

.. code-block:: java

    {
        "functionArg": 随调用SDK方法而变的入参json字符串 {
        },
        "transactionArg": 交易参数json字符串 {
            "nonce": 交易随机数，请调用weidentity-client相应函数生成 "<..the nonce>",
            "data": 交易特征值 "<..the data>"
        }
        "functionName": 调用SDK方法名 "<..the function name>",
        "v": 调用版本 "<..the API version>"
    }

参数说明：

- functionArg是随不同的SDK调用方法而变的，具体的参数可以查看SDK接口文档（后面每个接口会给出对应的链接）。
- transactionArg包含两个参数，但不是所有的接口都会用到，用不到的传空即可：
    - 调用接口encodeTransaction的时候，需要nonce随机数用于生成交易特征值data；
    - 调用接口sendTransaction的时候，需要之前的nonce随机数和交易特征值data，用于组装区块链交易；
    - 调用接口InvokeFunction的时候，这一部分会被完全忽略，因为不需要链上交易。
- functionName是调用的SDK方法名，用于决定具体调用WeIdentity Java SDK的什么功能。
    - 为了便于体验，functionName是大小写无关的
- v是调用的API方法版本

每个API的接口返回都是满足以下格式的json字符串：

.. code-block:: java

    {
        "respBody": 随调用SDK方法而变的输出值json字符串 {
        }
        "ErrorCode": 错误码
        "ErrorMessage": 错误信息 "success"
    }


其中具体的输出值result亦是随不同的SDK调用方法而变的。

在后文中，我们将会逐一说明目前所提供的功能及其使用方式。

2. 创建WeID
-------------

第一步：
~~~~~~~~~

调用接口：

.. list-table::
   :header-rows: 1
   :widths: 30 50

   * - 标题
     - 描述
   * - 接口名
     - weIdentity/encodeTransaction
   * - Method
     - POST
   * - Content-Type
     - application/json

接口入参: 

.. list-table::
   :header-rows: 1
   :widths: 30 60 20

   * - Key
     - Value
     - Required
   * - functionName
     - createWeId
     - Y
   * - functionArg
     - 
     - Y
   * - functionArg.publicKey
     - 符合ECDSA标准的公钥整型数，与[SDK直接调用的方式入参](https://weidentity.readthedocs.io/projects/javasdk/zh_CN/latest/docs/weidentity-java-sdk-doc.html#id9)一致。
     - Y
   * - transactionArg
     - 
     - Y
   * - transactionArg.nonce
     - 交易随机数，请调用 weidentity-java-client 相应函数生成
     - Y
   * - transactionArg.data
     - 交易特征值
     - N
   * - v
     - 版本号
     - Y

接口入参示例：

.. code-block:: java

    {
        "functionName": "createWeId",
        "functionArg": {
            "publicKey": ECDSA公钥，如"11111111"
        },
        "transactionArg": {
            "nonce": "14616548136584"
        },
        "v": "1.0.0"
    }

接口返回: application/json

.. list-table::
   :header-rows: 1
   :widths: 30 50

   * - Key
     - Value
   * - ErrorCode
     - 错误码，0表示成功
   * - ErrorMessage
     - 错误信息
   * - respBody
     - 
   * - respBody.encodedTransaction
     - Base64编码的encode交易信息
   * - respBody.data
     - 交易特征值（rawTransaction的方法成员）

接口返回示例：

.. code-block:: java

    {
        "ErrorCode": 0,
        "ErrorMessage": "success"
        "respBody": {
            "encodedTransaction": Base64字符串的encode交易信息
            "data": rawTransaction的方法成员字符串
        },
    }

result包含encodedTransaction和data两项。调用者将data妥善保管。

第二步：
~~~~~~~~~

调用者随后需要使用自己的ECDSA私钥对encodeTransaction接口返回值进行签名（可以直接使用我们提供的方便函数），并生成signedMessage。

第三步：
~~~~~~~~~~

调用接口：

.. list-table::
   :header-rows: 1
   :widths: 30 50

   * - 标题
     - 描述
   * - 接口名
     - weIdentity/sendTransaction
   * - Method
     - POST
   * - Content-Type
     - application/json

接口入参：

.. list-table::
   :header-rows: 1
   :widths: 30 60 20

   * - Key
     - Value
     - Required
   * - functionName
     - createWeId
     - Y
   * - functionArg
     - 
     - N
   * - transactionArg
     - 
     - Y
   * - transactionArg.signedMessage
     - Y
     - 格式为Base64编码后的签名值
   * - transactionArg.nonce
     - 交易随机数，请调用 weidentity-java-client 相应函数生成
     - Y
   * - transactionArg.data
     - 交易特征值，为第一步调用中返回的 respBody.data 值
     - Y
   * - v
     - 版本号
     - Y

接口入参示例：

.. code-block:: java

    {
        "functionName": "createWeId",
        "functionArg": {
            "signedMessage": Base64字符串定长签名值
        },
        "transactionArg": {
            "nonce": "14616548136584"
            "data": 和第一步中返回值一致
        },
        "v": "1.0.0"
    }


接口返回: application/json


.. list-table::
   :header-rows: 1
   :widths: 30 50

   * - Key
     - Value
   * - ErrorCode
     - 错误码，0表示成功
   * - ErrorMessage
     - 错误信息
   * - respBody
     - True

返回示例：

.. code-block:: java

    {
        "ErrorCode": 0,
        "ErrorMessage": "success",
        "respBody": True
    }


3. 获取WeID Document
---------------------

调用接口：

.. list-table::
   :header-rows: 1
   :widths: 30 50

   * - 标题
     - 描述
   * - 接口名
     - weIdentity/InvokeFunction
   * - Method
     - POST
   * - Content-Type
     - application/json

接口入参：

.. list-table::
   :header-rows: 1
   :widths: 30 60 20

   * - Key
     - Value
     - Required
   * - functionName
     - getWeIdDocument
     - Y
   * - functionArg
     - 
     - Y
   * - functionArg.weId
     - WeIdentity DID，与[SDK直接调用的方式入参](https://weidentity.readthedocs.io/projects/javasdk/zh_CN/latest/docs/weidentity-java-sdk-doc.html#id9)一致。
     - Y
   * - transactionArg
     - 
     - N
   * - v
     - 版本号
     - Y

接口入参示例：

.. code-block:: java

    {
        "functionArg": {
            "weId": weId地址，如"did:weid:0x12025448644151248e5c1115b23a3fe55f4158e4153"
        },
        "transactionArg": {
        },
        "functionName": "getWeIdDocument",
        "v": "1.0.0"
    }


接口返回: application/json

.. list-table::
   :header-rows: 1
   :widths: 30 50

   * - Key
     - Value
   * - ErrorCode
     - 错误码，0表示成功
   * - ErrorMessage
     - 错误信息
   * - respBody
     - WeIdentity DID Document

返回示例：

.. code-block:: java

    {
        "respBody": {
            "@context" : "https://w3id.org/did/v1",
            "id" : "did:weid:0x2c194c296c0235ad92560629fffa281b3deff08a",
            "created" : 1553224394993,
            "updated" : 1553224394993,
            "publicKey" : [ ],
            "authentication" : [ ],
            "service" : [ ]
        },
        "ErrorCode": 0,
        "ErrorMessage": "success"
    }


4. 创建AuthorityIssuer
---------------------

第一步：
~~~~~~~~~~~~

调用接口：

.. list-table::
   :header-rows: 1
   :widths: 30 50

   * - 标题
     - 描述
   * - 接口名
     - weIdentity/encodeTransaction
   * - Method
     - POST
   * - Content-Type
     - application/json

接口入参示例：


.. list-table::
   :header-rows: 1
   :widths: 30 60 20

   * - Key
     - Value
     - Required
   * - functionName
     - registerAuthorityIssuer
     - Y
   * - functionArg
     - 
     - Y
   * - functionArg.weId
     - WeIdentity DID，与[SDK直接调用的方式入参](https://weidentity.readthedocs.io/projects/javasdk/zh_CN/latest/docs/weidentity-java-sdk-doc.html#id9)一致，下同
     - Y
   * - functionArg.name
     - 机构名
     - Y
   * - transactionArg
     - 
     - Y
   * - transactionArg.nonce
     - 交易随机数，请调用 weidentity-java-client 相应函数生成
     - Y
   * - transactionArg.data
     - 交易特征值
     - N
   * - v
     - 版本号
     - Y

接口调用示例：

.. code-block:: java

    {
        "functionArg": {
            "weid": "did:weid:0x12025448644151248e5c1115b23a3fe55f4158e4153",
            "name": "Sample College"
        },
        "transactionArg": {
            "nonce": "14616548136584"
        },
        "functionName": "registerAuthorityIssuer",
        "v": "1.0.0"
    }

接口返回: application/json

.. list-table::
   :header-rows: 1
   :widths: 30 50

   * - Key
     - Value
   * - ErrorCode
     - 错误码，0表示成功
   * - ErrorMessage
     - 错误信息
   * - respBody
     - 
   * - respBody.encodedTransaction
     - Base64编码的encode交易信息
   * - respBody.data
     - 交易特征值（rawTransaction的方法成员）

返回示例：

.. code-block:: java

    {
        "respBody": {
            "encodedTransaction": Base64字符串的encode交易信息
            "data": rawTransaction的方法成员字符串
        },
        "ErrorCode": 0,
        "ErrorMessage": "success"
    }

第二步：
~~~~~~~~~~~~~

调用者随后需要使用自己的ECDSA私钥对encodeTransaction进行签名，并生成signedMessage。

第三步：
~~~~~~~~~~~~~

POST /weIdentity/sendTransaction

调用接口：

.. list-table::
   :header-rows: 1
   :widths: 30 50

   * - 标题
     - 描述
   * - 接口名
     - weIdentity/sendTransaction
   * - Method
     - POST
   * - Content-Type
     - application/json

接口入参：

.. list-table::
   :header-rows: 1
   :widths: 30 60 20

   * - Key
     - Value
     - Required
   * - functionName
     - registerAuthorityIssuer
     - Y
   * - functionArg
     - 
     - N
   * - transactionArg
     - 
     - Y
   * - transactionArg.signedMessage
     - Y
     - 格式为Base64编码后的签名值
   * - transactionArg.nonce
     - 交易随机数，请调用 weidentity-java-client 相应函数生成
     - Y
   * - transactionArg.data
     - 交易特征值，为第一步调用中返回的 respBody.data 值
     - Y
   * - v
     - 版本号
     - Y

接口入参示例：

.. code-block:: java

    {
        "functionName": "registerAuthorityIssuer",
        "functionArg": {
            "signedMessage": Base64字符串定长签名值
        },
        "transactionArg": {
            "nonce": "14616548136584"
            "data": 和第一步中返回值一致
        },
        "v": "1.0.0"
    }


接口返回: application/json


.. list-table::
   :header-rows: 1
   :widths: 30 50

   * - Key
     - Value
   * - ErrorCode
     - 错误码，0表示成功
   * - ErrorMessage
     - 错误信息
   * - respBody
     - True

返回示例：

.. code-block:: java

    {
        "ErrorCode": 0,
        "ErrorMessage": "success",
        "respBody": True
    }


5. 查询AuthorityIssuer
------------------------------

调用接口：

.. list-table::
   :header-rows: 1
   :widths: 30 50

   * - 标题
     - 描述
   * - 接口名
     - weIdentity/InvokeFunction
   * - Method
     - POST
   * - Content-Type
     - application/json

接口入参：

.. list-table::
   :header-rows: 1
   :widths: 30 60 20

   * - Key
     - Value
     - Required
   * - functionName
     - queryAuthorityIssuer
     - Y
   * - functionArg
     - 
     - Y
   * - functionArg.weId
     - WeIdentity DID，与[SDK直接调用的方式入参](https://weidentity.readthedocs.io/projects/javasdk/zh_CN/latest/docs/weidentity-java-sdk-doc.html#id9)一致。
     - Y
   * - transactionArg
     - 
     - N
   * - v
     - 版本号
     - Y

接口入参示例：

.. code-block:: java

    {
        "functionArg": {
            "weId": weId地址，如"did:weid:0x12025448644151248e5c1115b23a3fe55f4158e4153"
        },
        "transactionArg": {
        },
        "functionName": "queryAuthorityIssuer",
        "v": "1.0.0"
    }

接口返回: application/json

.. list-table::
   :header-rows: 1
   :widths: 30 50

   * - Key
     - Value
   * - ErrorCode
     - 错误码，0表示成功
   * - ErrorMessage
     - 错误信息
   * - respBody
     - 完整的Authority Issuer信息


.. code-block:: java

    {
        "respBody": {
            "accValue": ,
            "created": 16845611984115,
            "name": "Sample College",
            "weid": "did:weid:0x12025448644151248e5c1115b23a3fe55f4158e4153"
        }
        "ErrorCode": 0
        "ErrorMessage": "success"
    }


6. 创建CPT
---------------

第一步：
~~~~~~~~~~

调用接口：

.. list-table::
   :header-rows: 1
   :widths: 30 50

   * - 标题
     - 描述
   * - 接口名
     - weIdentity/encodeTransaction
   * - Method
     - POST
   * - Content-Type
     - application/json

接口入参: 

.. list-table::
   :header-rows: 1
   :widths: 30 60 20

   * - Key
     - Value
     - Required
   * - functionName
     - registerCpt
     - Y
   * - functionArg
     - 
     - Y
   * - functionArg.weId
     - CPT创建者，与[SDK直接调用的方式入参](https://weidentity.readthedocs.io/projects/javasdk/zh_CN/latest/docs/weidentity-java-sdk-doc.html#id9)一致，后略
     - Y
   * - functionArg.cptJsonSchema
     - CPT Json Schema
     - Y
   * - functionArg.cptSignature
     - CPT创建者的签名
     - Y
   * - transactionArg
     - 
     - Y
   * - transactionArg.nonce
     - 交易随机数，请调用 weidentity-java-client 相应函数生成
     - Y
   * - transactionArg.data
     - 交易特征值
     - N
   * - v
     - 版本号
     - Y

接口入参示例：

.. code-block:: java

      {
        "functionArg": {
            "weId": "did:weid:0x12025448644151248e5c1115b23a3fe55f4158e4153",
            "cptJsonSchema":{
                "title": "cpt",
                "description": "this is cpt",
                "properties": {
                    "name": {
                        "type": "string",
                        "description": "the name of certificate owner"
                    },
                    "gender": {
                        "enum": [
                            "F",
                            "M"
                        ],
                    "type": "string",
                    "description": "the gender of certificate owner"
                    },
                    "age": {
                        "type": "number",
                        "description": "the age of certificate owner"
                    }
                },
                "required": [
                    "name",
                    "age"
                ]
            },
            "cptSignature": "MTIzNDU2NzgxMjM0NTY3ODEyMzQ1Njc4MTIzNDU2NzgxMjM0NTY3ODEyMzQ1Njc4MTIzNDU2NzgxMjM0NTY3ODU="
        },
        "transactionArg": {
            "nonce": "12321376217856"
        }，
        "functionName": "registerCpt"，
        "v": "1.0.0"
      }

接口返回: application/json

.. list-table::
   :header-rows: 1
   :widths: 30 50

   * - Key
     - Value
   * - ErrorCode
     - 错误码，0表示成功
   * - ErrorMessage
     - 错误信息
   * - respBody
     - 
   * - respBody.encodedTransaction
     - Base64编码的encode交易信息
   * - respBody.data
     - 交易特征值（rawTransaction的方法成员）

返回示例：

.. code-block:: java

    {
        "respBody": {
            "encodedTransaction": Base64字符串的encode交易信息
            "data": rawTransaction的方法成员字符串
        },
        "ErrorCode": 0,
        "ErrorMessage": "success"
    }



第二步：
~~~~~~~~~~~~

调用者随后需要使用自己的ECDSA私钥对encodeTransaction进行签名，并生成signedMessage。

第三步：
~~~~~~~~~~~~~

调用接口：

.. list-table::
   :header-rows: 1
   :widths: 30 50

   * - 标题
     - 描述
   * - 接口名
     - weIdentity/sendTransaction
   * - Method
     - POST
   * - Content-Type
     - application/json

接口入参：

.. list-table::
   :header-rows: 1
   :widths: 30 60 20

   * - Key
     - Value
     - Required
   * - functionName
     - registerCpt
     - Y
   * - functionArg
     - 
     - N
   * - transactionArg
     - 
     - Y
   * - transactionArg.signedMessage
     - Y
     - 格式为Base64编码后的签名值
   * - transactionArg.nonce
     - 交易随机数，请调用 weidentity-java-client 相应函数生成
     - Y
   * - transactionArg.data
     - 交易特征值，为第一步调用中返回的 respBody.data 值
     - Y
   * - v
     - 版本号
     - Y

接口入参示例：

.. code-block:: java

    {
        "functionName": "registerCpt",
        "functionArg": {
            "signedMessage": Base64字符串定长签名值
        },
        "transactionArg": {
            "nonce": "14616548136584"
            "data": 和第一步中返回值一致
        },
        "v": "1.0.0"
    }

接口返回: application/json


.. list-table::
   :header-rows: 1
   :widths: 30 50

   * - Key
     - Value
   * - ErrorCode
     - 错误码，0表示成功
   * - ErrorMessage
     - 错误信息
   * - respBody
     - cptBaseInfo

返回示例：

.. code-block:: java

    {
        "respBody": {
            "cptId": 12,
            "cptVersion": 1
        },
        "ErrorCode": 0,
        "ErrorMessage": "success"
    }


7. 查询CPT
-----------

POST /weIdentity/InvokeFunction

接口入参：Json，cptId

.. code-block:: java

    {
        "functionArg": {
            "cptId": 10,
        },
        "transactionArg": {
        },
        "functionName": "queryCpt",
        "v": "1.0.0"
    }

入参说明与SDK文档一致：https://weidentity.readthedocs.io/projects/javasdk/zh_CN/latest/docs/weidentity-java-sdk-doc.html#querycpt

接口返回：Json，完整的Cpt信息

.. code-block:: java

    {
        "respBody": {
            "cptBaseInfo" : {
                "cptId" : 2000308,
                "cptVersion" : 1
            },
            "cptId" : 2000308,
            "cptJsonSchema" : {
                "$schema" : "http://json-schema.org/draft-04/schema#",
                "title" : "a CPT schema",
                "type" : "object"
            },
            "cptPublisher" : "did:weid:0x104a58c272e8ebde0c29083552ebe78581322908",
            "cptSignature" : "HJPbDmoi39xgZBGi/aj1zB6VQL5QLyt4qTV6GOvQwzfgUJEZTazKZXe1dRg5aCt8Q44GwNF2k+l1rfhpY1hc/ls=",
            "cptVersion" : 1,
            "created" : 1553503354555,
            "metaData" : {
                "cptPublisher" : "did:weid:0x104a58c272e8ebde0c29083552ebe78581322908",
                "cptSignature" : "HJPbDmoi39xgZBGi/aj1zB6VQL5QLyt4qTV6GOvQwzfgUJEZTazKZXe1dRg5aCt8Q44GwNF2k+l1rfhpY1hc/ls=",
                "created" : 1553503354555,
                "updated" : 0
            },
            "updated" : 0
        },
        "ErrorCode": 0,
        "ErrorMessage": "success"
    }


8. 创建Credential
------------------

POST /weIdentity/InvokeFunction

接口入参：Json，以signature代替私钥

.. code-block:: java

    {
        "functionArg": {
            "cptId": 10,
            "issuer": "did:weid:0x12025448644151248e5c1115b23a3fe55f4158e4153",
            "expirationDate": "2019-04-18T21:12:33Z",
            "claim": claimJson结构体, 略去
        },
        "transactionArg": {
        },
        "functionName": "createCredential",
        "v": "1.0.0"
    }

入参说明与SDK文档一致：https://weidentity.readthedocs.io/projects/javasdk/zh_CN/latest/docs/weidentity-java-sdk-doc.html#createcredential

接口返回：Json，完整的Credential，与SDK文档一致

.. code-block:: java

    {
        "respBody": {
            "context": "https://www.w3.org/2018/credentials/v1",
            "cptId": 10,
            "uuid" : "decd7c81-6b41-414d-8323-00161317a38e",
            "issuer": "did:weid:0x12025448644151248e5c1115b23a3fe55f4158e4153",
            "issuranceDate": "2019-03-19T21:12:33Z",
            "expirationDate": "2019-04-18T21:12:33Z",
            "claim": claimJson结构体, 略去
            "signature": "MTIzNDU2NzgxMjM0NTY3ODMzMzM0NDQ0MTIzNDU2NzgxMjM0NTY3ODEyMzQ1Njc4MTIzNDU2NzgxMjM0NTY3ODU="
        },
        "ErrorCode": 0,
        "ErrorMessage": "success"
    }


9. 验证Credential
--------------------

POST /weIdentity/InvokeFunction

接口入参：

.. code-block:: java

    {
        "functionArg": {
            "context": "https://www.w3.org/2018/credentials/v1",
            "cptId": 10,
            "uuid" : "decd7c81-6b41-414d-8323-00161317a38e",
            "issuer": "did:weid:0x12025448644151248e5c1115b23a3fe55f4158e4153",
            "issuranceDate": "2019-03-19T21:12:33Z",
            "expirationDate": "2019-04-18T21:12:33Z",
            "claim": claimJson结构体,
            "signature": "MTIzNDU2NzgxMjM0NTY3ODMzMzM0NDQ0MTIzNDU2NzgxMjM0NTY3ODEyMzQ1Njc4MTIzNDU2NzgxMjM0NTY3ODU="
        },
        "transactionArg": {
        },
        "functionName": "verifyCredential"
        "v": "1.0.0"
    }


入参说明与SDK文档一致：https://weidentity.readthedocs.io/projects/javasdk/zh_CN/latest/docs/weidentity-java-sdk-doc.html#verifycredential

接口返回：

.. code-block:: java

    {
        "respBody": True,
        "ErrorCode": 0,
        "ErrorMessage": "success"
    }
