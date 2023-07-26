<p align="center">
<a href="https://packagist.org/packages/code-lives/byte" target="_blank"><img src="https://img.shields.io/packagist/v/code-lives/byte?include_prereleases" alt="GitHub forks"></a>
<a href="https://packagist.org/packages/code-lives/byte" target="_blank"><img src="https://img.shields.io/github/stars/code-lives/byte?style=social" alt="GitHub forks"></a>
<a href="https://github.com/code-lives/byte/fork" target="_blank"><img src="https://img.shields.io/github/forks/code-lives/byte?style=social" alt="GitHub forks"></a>

</p>

|                                               第三方                                                | token | openid | 支付 | 回调 | 退款 | 订单查询 | 解密手机号 | 分账 | 模版消息 |
| :-------------------------------------------------------------------------------------------------: | :---: | :----: | :--: | :--: | :--: | :------: | :--------: | :--: | :------: |
| [抖音小程序](https://developer.open-douyin.com/docs/resource/zh-CN/mini-app/introduction/overview/) |   ✓   |   ✓    |  ✓   |  ✓   |  ✓   |    ✓     |     ✓      |  ✓   |    ✓     |

## 安装

```php
composer require code-lives/byte 1.0.0
```

### ⚠️ 注意

> 金额单位分 100=1 元

> 返回结果 array 由开发者自行判断

> 抖音小程序由字节小程序转变而来，支持多端（头条、抖音、今日头条等关联应用）

# 预下单

```php
<?php

//引入命名空间
use Applet\Assemble\Byte;

$pay= Byte::init($config)->set("订单号","金额","描述","描述")->getParam();

```

# 抖音小程序

### Config

| 参数名字    | 类型   | 必须 | 说明                              |
| ----------- | ------ | ---- | --------------------------------- |
| token       | string | 是   | 担保交易回调的 Token(令牌)        |
| salt        | string | 是   | 担保交易的 SALT                   |
| merchant_id | string | 是   | 担保交易的商户号                  |
| app_id      | int    | 是   | 小程序的 APP_ID                   |
| secret      | string | 是   | 小程序的 APP_SECRET               |
| notify_url  | string | 是   | 支付回调 url                      |
| settle_url  | string | 否   | 分账回调 url,没有默认支付回调 url |

### Token

```php
$data= Byte::init($config)->getToken();
```

| 返回参数     | 类型   | 必须 | 说明                   |
| ------------ | ------ | ---- | ---------------------- |
| expires_in   | string | 是   | 凭证有效时间，单位：秒 |
| access_token | string | 是   | 获取到的凭证           |

### Openid

```php
$code="";
$anonymous_code="";//可以不传
$data= Byte::init($config)->getOpenid($code,$anonymous_code);
```

| 返回参数    | 类型   | 必须 | 说明        |
| ----------- | ------ | ---- | ----------- |
| session_key | string | 是   | session_key |
| openid      | string | 是   | 用户 openid |
| unionid     | string | 是   | unionid     |

### 解密手机号

```php
$data= Byte::init($config)->decryptPhone($session_key, $iv, $encryptedData);
echo $phone['phoneNumber'];
```

### 订单查询

```php
$data = Byte::init($config)->findOrder("订单号");
```

### 分账

| 参数名字      | 类型   | 必须 | 说明                           |
| ------------- | ------ | ---- | ------------------------------ |
| out_order_no  | string | 是   | 平台订单号                     |
| out_settle_no | string | 是   | 自定义订单号                   |
| settle_desc   | int    | 是   | 分账描述                       |
| cp_extra      | string | 是   | 开发者自定义字段，回调原样回传 |

```php
$data = Byte::init($config)->settle($order);
```

### 退款

| 参数名字      | 类型   | 必须 | 说明         |
| ------------- | ------ | ---- | ------------ |
| out_order_no  | string | 是   | 平台订单号   |
| out_refund_no | int    | 是   | 自定义订单号 |
| reason        | int    | 是   | 退款说明     |
| refund_amount | string | 是   | 退款金额     |

```php
$order = [
        'out_order_no' => '',
        'out_refund_no' => time() . 'refund',
        'reason' => '就想退款，咋滴',
        'refund_amount' => 1, //退款金额
    ];
$data= Byte::init($config)->applyOrderRefund($order);
```

### 模版消息

```php
$data = [
        'tpl_id' =>  "模版id",
        "open_id" => $parm['openid'],
        'data' => [
            '律师' => "张三",
            "回复时间" => date('Y-m-d H:i:s', time()),
            "回复内容" => "我回复你啦",
        ],
        "page" => "pages/index/index",
    ];
$data= Byte::init($config)->sendMsg($data,$token);

```

### 支付回调

```php
$byte = Byte::init($config);
$status = $byte->notifyCheck(); //验证
if ($status) {
    $orderSn = $byte->getNotifyOrder(); //订单数据$orderSn['msg']['cp_orderno'] $orderSn['msg']['seller_uid']
    switch ($orderSn['type']) {
        case 'payment': // 支付相关回调
            /**
             *业务处理
            */
            echo json_encode(['err_no' => 0, 'err_tips' => 'success']);exit; // 操作成功需要给头条返回的信息
            break;
        case 'refund': // 退款相关回调
            /**
             *业务处理
            */
            echo json_encode(['err_no' => 0, 'err_tips' => 'success']);exit; // 操作成功需要给头条返回的信息
            break;
        case 'settle': // 分账相关回调
            /**
             *业务处理
            */
            echo json_encode(['err_no' => 0, 'err_tips' => 'success']);exit; // 操作成功需要给头条返回的信息
            break;
        default: // 未知数据
            return '数据异常';
    }
}
```
