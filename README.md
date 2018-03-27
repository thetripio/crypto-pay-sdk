# Crypto Pay 支付系统集成流程



接口地址：

测试接口： https://test.payment.trip.io

生产接口：https://payment.trip.io



### 主要资源：

Payment：支付对象

Currency：数字货币对象

Transaction：区块链交易对象

Refund：退款申请对象

Config：配置对象



### 认证方式



支持多种数字货币的支付网关。开发者调用接口创建收款请求信息，消费者通过钱包支付到特定地址，收款成功后通过回调通知开发者。消费者支付页面跳转至制定地址。支持 Web 和 Mobile。



如 Header：

``Authentication：apiKey 1fa4Y52SWEhii7CmYiMOcv:4ToXczFz0ZyCgLpgKIkyxA``



### 2. 配置回调与 IP 白名单

接口：config

```
	  paymentWebhookUrl:
        type: "string"
        description: "回调接口，传入 Payment 对象"
        example: "https://app.trip.io/payment/webhook"
      refundWebhookUrl:
        type: "string"
        description: "退款回调接口，传入 Refund 对象"
        required: false
        example: "https://app.trip.io/refund/finished?refundId=821"
      redirectUrl:
        type: "string"
        description: "页面跳转地址，Query 传参 paymentId"
        required: false
        example: "https://app.trip.io/payment/finished?paymentId=823951"
      whitelist:
        type: "array"
        description: "服务器 IP 白名单"
        required: false
        items:
          type: "string"
```



### 2. 创建支付

接口：payments/create

```
	  paymentCurrency:
        type: "string"
        description: "用于支付的数字货币代号"
        examples:
          example: "ETH"
          example1: "TRIO"
      paymentAmount:
        type: "string"
        description: "用于支付的数字货币数量"
        example: "0.24"
      referenceCurrency:
        type: "string"
        description: "用于参考显示的法币/数字货币代号"
        enum:
        - "USD,CNY"
        required: false
        example:
          value: "USD"
          strict: false
      referenceAmount:
        type: "string"
        description: "用于参考显示的法币/数字货币数量"
        required: false
        example: "352"
      orderName:
        type: "string"
        description: "订单内容名称"
        required: false
        example: "万豪住一晚"
      orderId:
        type: "string"
        description: "订单号"
        required: false
        example: "8127389631223"
      orderNotes:
        type: "string"
        description: "订单内容/备注"
        required: false
        example: "前台验证码: 234098"
      timeFrame:
        type: "number"
        description: "支付时间窗口（秒）"
        default: 900
        required: false
        example: 600
      requiredConfirmations:
        type: "integer"
        description: "所需区块确认数，减少确认数可以减少时间"
        default: 12
        required: false
        example: 1
```



返回：支付链接



```
{
    success:true,
    paymentLink: "https://payment.trip.io/bill/x23nosfb323
}
```



### 3. Webhook 回调

```
	  paymentId:
        type: "string"
        description: "支付 ID"
      orderId:
        type: "string"
        description: "订单号"
        required: false
        example: "8127389631223"
      paidPercentage:
        type: "string"
        description: "已支付的比例 。少于1为部分支付，1为完全支付，大于1为超额支付。"
        examples:
          example: "0.6"
          example1: "1"
          example2: "1.2"
      paymentCompleted:
        type: "boolean"
        description: "是否已完成支付（包括超额支付）"
        examples:
          example: true
          example1: false
      createdAt:
        type: "datetime"
        description: "订单创建时间，订单接口自动生成"
        example: "2016-02-28T16:41:41.090Z"
      completedAt:
        type: "datetime"
        description: "订单完成时间，系统自动生成"
        required: false
        example: "2016-02-28T16:41:41.090Z"
      lastTransactionAt:
        type: "string"
        description: "最后一次转账时间，系统自动生成"
        required: false
        example: "2016-02-28T16:41:41.090Z"
```



### 4. 页面跳转



附带参数：paymentId



### 5. 退款



接口：payments/{paymentId}/refund



```
      paymentId:
        type: "string"
        description: "支付 ID"
        required: false
      requestedAt:
        type: "datetime"
        description: "发起退款请求的时间，系统自动记录"
        required: false
      refundToAddress:
        type: "string"
        description: "退款地址，为了解决交易所钱包问题"
        required: false
      fee:
        type: "string"
        description: "退款手续费（待定）"
        required: false
      notes:
        type: "string"
        description: "退款备注"
        required: false
```



### 6. 退款回调



```
      paymentId:
        type: "string"
        description: "支付 ID"
        required: false
      refundToAddress:
        type: "string"
        description: "退款地址，为了解决交易所钱包问题"
        required: false
      completed:
        type: "boolean"
        description: "是否已完成"
        required: false
      completedAt:
        type: "string"
        description: "完成时间"
        required: false
      blockTxId:
        type: "string"
        description: "区块交易号"
        required: false
      webhookCalledAt:
        type: "string"
        description: "退款成功回调时间 (code 200)"
        required: false
```



### 7. 汇率查询

法币汇率：/rates/fiat

```

  displayName: "法币汇率"
  get:
    displayName: "获取法币对数字货币的汇率"
    queryParameters:
      sourceSymbol:
        type: "string"
        description: "法币代号"
        required: false
        examples:
          example: "USD"
          example1: "CNY"
      targetSymbol:
        type: "string"
        description: "数字货币代号"
        required: false
        examples:
          example: "ETH"
          example1: "LTC"
          example2: "TRIO"
      amount:
        type: "string"
        description: "法币数量"
        required: false
        example: "621"
    responses:
      200:
        body:
          application/json:
            type: "object"
            description: "汇率换算结果"
            properties:
              sourceSymbol:
                type: "string"
                description: "法币代号，如: USD"
                required: false
              targetSymbol:
                type: "string"
                description: "数字货币代号，如: ETH"
                required: false
              rate:
                type: "string"
                description: "法币数量，如：125"
                required: false
              targetAmount:
                type: "string"
                description: "换算后的数字货币数量，如：0.23"
                required: false

```

数字货币汇率：

```
  displayName: "数字货币汇率"
  description: "待完善"
  get:
    displayName: "获取数字货币对法币的汇率"
    queryParameters:
      sourceSymbol:
        type: "string"
        description: "数字货币代号"
        example: "ETH"
      targetSymbol:
        type: "string"
        description: "法币代号"
        example: "USD"
      amount:
        type: "string"
        description: "数字货币数量"
        default: "1"
        required: false
```

