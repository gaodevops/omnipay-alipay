# Omnipay: Alipay


**Alipay driver for the Omnipay PHP payment processing library**

[Omnipay](https://github.com/omnipay/omnipay) is a framework agnostic, multi-gateway payment
processing library for PHP. This package implements Alipay support for Omnipay.

> Cross-border Alipay payment please use [`lokielse/omnipay-global-alipay`](https://github.com/lokielse/omnipay-global-alipay)
 
> Legacy Version please use [`"lokielse/omnipay-alipay": "dev-legacy"`](https://github.com/lokielse/omnipay-alipay/tree/legacy)

## Installation

Omnipay is installed via [Composer](http://getcomposer.org/). To install, simply add it
to your `composer.json` file:

    "gaodevops/omnipay-alipay": "^2.0",

And run composer to update your dependencies:

    $ composer update -vvv

## Basic Usage

The following gateways are provided by this package:

| Gateway       	    		|         Description             |说明                 | Links |
|:---------------	    	|:---------------------------     |:---------         |:----------:|
| Alipay_AopPage 	    		| Alipay Page Gateway             |电脑网站支付 - new    | [Usage][link-wiki-aop-page] [Doc][link-doc-aop-page] |
| Alipay_AopApp 	    		| Alipay APP Gateway              |APP支付 - new    | [Usage][link-wiki-aop-app] [Doc][link-doc-aop-app] |
| Alipay_AopF2F 	    		| Alipay Face To Face Gateway     |当面付 - new         | [Usage][link-wiki-aop-f2f] [Doc][link-doc-aop-f2f] |
| Alipay_AopWap 	    		| Alipay WAP Gateway              |手机网站支付 - new     | [Usage][link-wiki-aop-wap] [Doc][link-doc-aop-wap] |
| Alipay_AopJs 	    		| Alipay Js Gateway              |JSAPI - new     | [Usage][link-wiki-aop-js] [Doc][link-doc-aop-js] |
| Alipay_LegacyApp 	    	| Alipay Legacy APP Gateway       |APP支付      | [Usage][link-wiki-legacy-app] [Doc][link-doc-legacy-app]      |
| Alipay_LegacyExpress 		| Alipay Legacy Express Gateway   |即时到账    | [Usage][link-wiki-legacy-express] [Doc][link-doc-legacy-express]|
| Alipay_LegacyWap      	| Alipay Legacy WAP Gateway   |手机网站支付     | [Usage][link-wiki-legacy-wap] [Doc][link-doc-legacy-wap]       |

## Usage

### Purchase (购买)

```php
/**
 * @var AopAppGateway $gateway
 */
$gateway = Omnipay::create('Alipay_AopPage');
$gateway->setSignType('RSA2'); // RSA/RSA2/MD5
$gateway->setAppId('the_app_id');
$gateway->setPrivateKey('the_app_private_key');
$gateway->setAlipayPublicKey('the_alipay_public_key');
$gateway->setReturnUrl('https://www.example.com/return');
$gateway->setNotifyUrl('https://www.example.com/notify');

/**
 * @var AopTradePagePayResponse $response
 */
$response = $gateway->purchase()->setBizContent([
    'subject'      => 'test',
    'out_trade_no' => date('YmdHis') . mt_rand(1000, 9999),
    'total_amount' => '0.01',
    'product_code' => 'FAST_INSTANT_TRADE_PAY',
])->send();

$url = $response->getRedirectUrl();
```

For general usage instructions, please see the main [Omnipay](https://github.com/omnipay/omnipay)
repository.

### Refund (退款)
```
    /**
     * 支付宝退款 (有密)
     * @param Request $request
     * @return mixed
     */
    public function aliRefund(Request $request)
    {
        $params = $request->all();
        $order_number = array_get($params, 'order_number');
        $out_trade_no = array_get($params, 'out_trade_no');
        $amount = array_get($params, 'amount');
        $gateway = Omnipay::create('Alipay_LegacyExpress');
        $gateway->setSignType(config('config.AliPay.sign_type'));
        $gateway->setReturnUrl(config('config.AliPay.return_url')); //同步通知地址
        $gateway->setNotifyUrl(config('config.AliPay.notify_url')); //异步通知地址
        $gateway->setSellerEmail(config('config.AliWebPay.seller_email'));
        $gateway->setPartner(config('config.AliWebPay.partner'));
        $gateway->setKey(config('config.AliWebPay.key'));
        $data = [
            'refund_date' => date('Y-m-d H:i:s'),
            "seller_user_id" => trim(config('config.AliWebPay.seller_id')),
            'batch_no' => $order_number,
            'batch_num' => 1,//退款笔数与refund_items数组中保持一致
            '_input_charset' => 'UTF-8',
            'refund_items' => [
                [
                    'out_trade_no' => $out_trade_no,
                    'amount' => $amount / 100.0,
                    'reason' => 'User_refund'
                ]
            ],
        ];
        $request = $gateway->refund($data);
        $response = $request->send();
        $url = $response->getRedirectUrl();
        return ['url' => $url]; //在浏览器中打开该地址输入密码进行退款
    }

    /**
     * 支付宝退款 （无密）
     * @param Request $request
     * @return mixed
     */
    public function aliRefundNoPwd(Request $request)
    {
        $params = $request->all();
        $order_number = array_get($params, 'order_number');
        $out_trade_no = array_get($params, 'out_trade_no');
        $amount = array_get($params, 'amount');
        $gateway = Omnipay::create('Alipay_LegacyExpress');
        $gateway->setSignType(config('config.AliPay.sign_type'));
        $gateway->setReturnUrl(config('config.AliPay.return_url')); //同步通知地址
        $gateway->setNotifyUrl(config('config.AliPay.notify_url')); //异步通知地址
        $gateway->setSellerEmail(config('config.AliWebPay.seller_email'));
        $gateway->setPartner(config('config.AliWebPay.partner'));
        $gateway->setKey(config('config.AliWebPay.key'));
        $data = [
            'refund_date' => date('Y-m-d H:i:s'),
            "seller_user_id" => trim(config('config.AliWebPay.seller_id')),
            'batch_no' => $order_number,
            'batch_num' => 1,//退款笔数与refund_items数组中保持一致
            '_input_charset' => 'UTF-8',
            'refund_items' => [
                [
                    'out_trade_no' => $out_trade_no,
                    'amount' => $amount / 100.0,
                    'reason' => 'User_refund'
                ]
            ],
        ];
        $request = $gateway->refundNoPwd($data);
        $response = $request->send();
        $url = $response->getRedirectUrl();
        $html = file_get_contents($url);
        $rs = $this->xmlToArray($html);
        if ($rs && array_get($rs, 'is_success') == 'T') {
            return ['msg' => '申请退款成功'];
        } else {
            if(array_get($rs, 'error') == 'DUPLICATE_BATCH_NO'){
                return '请勿重复申请 '. array_get($rs, 'error');
            }else{
                return '申请退款失败 '. array_get($rs, 'error');
            }
        }
    }
    
```
## Pay Notify （支付回调）
```
    /**
     * 支付宝支付回调(同步/异步) 验签  并返回回调数据
     * @param $params
     * @return bool|mixed
     */
    private function verifyAndGetAliDataForPay($params)
    {
        $gateway = Omnipay::create('Alipay_AopApp');
        $publicKey = config('config.AliPay.alipay_public_key');  //获取支付宝公钥
        $privateKey = config('config.AliPay.merchant_private_key');  //获取支付宝私钥
        $gateway->setSignType(config('config.AliPay.sign_type')); // RSA/RSA2/MD5
        $gateway->setAppId(config('config.AliPay.app_id'));
        $gateway->setPrivateKey($privateKey);
        $gateway->setAlipayPublicKey($publicKey);
        try {
            $response = $gateway->completePurchase(['params' => $params])->send(); //验签
            $data = $response->getData();
            return $data;
        } catch (InvalidRequestException $ex) {
            \Log::info('支付宝支付回调失败:' . $ex->getMessage());
            return false;
        }
    }
```

## Refund Notify （退款回调）
```
   /**
     * 支付宝退款回调异步 验签 并返回回调数据
     * @param $params
     * @return bool|mixed
     */
    private function verifyAndGetAliDataForRefund($params)
    {
        $gateway = Omnipay::create('Alipay_LegacyExpress');
        $gateway->setSignType(config('config.AliWebPay.sign_type')); // MD5
        $gateway->setPartner(config('config.AliWebPay.partner'));
        $gateway->setKey(config('config.AliWebPay.key'));
        try {
            $response = $gateway->completePurchase(['params' => $params])->send(); //验签
            $data = $response->getData();
            return $data;
        } catch (InvalidRequestException $ex) {
            \Log::info('支付宝退款回调失败:' . $ex->getMessage());
            return false;
        }
    }
```

## Related

- [Laravel-Omnipay](https://github.com/ignited/laravel-omnipay)
- [Omnipay-GlobalAlipay](https://github.com/lokielse/omnipay-global-alipay)
- [Omnipay-WechatPay](https://github.com/lokielse/omnipay-wechatpay)
- [Omnipay-UnionPay](https://github.com/lokielse/omnipay-unionpay)

## Support

If you are having general issues with Omnipay, we suggest posting on
[Stack Overflow](http://stackoverflow.com/). Be sure to add the
[omnipay tag](http://stackoverflow.com/questions/tagged/omnipay) so it can be easily found.

If you want to keep up to date with release anouncements, discuss ideas for the project,
or ask more detailed questions, there is also a [mailing list](https://groups.google.com/forum/#!forum/omnipay) which
you can subscribe to.

If you believe you have found a bug, please report it using the [GitHub issue tracker](https://github.com/lokielse/omnipay-alipay/issues),
or better yet, fork the library and submit a pull request.
