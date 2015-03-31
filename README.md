# weixin-pay
微信支付 for node.js

## Installation
```
npm install weixin-pay
```

## Usage

创建统一支付订单
```js
var WXPay = require('weixin-pay');

var wxpay = WXPay({
	appid: 'xxxxxxxx',
	mch_id: '1234567890',
	partner_key: 'xxxxxxxxxxxxxxxxx',
	pfx: fs.readFileSync('./wxpay_cert.p12'),
});

wxpay.createUnifiedOrder({
	body: '扫码支付测试',
	out_trade_no: '20140703'+Math.random().toString().substr(2, 10),
	total_fee: 1,
	spbill_create_ip: '192.168.2.210',
	notify_url: 'http://wxpay_notify_url',
	trade_type: 'NATIVE',
	product_id: '1234567890'
}, function(err, result){
	console.log(result);
});
```

### 原生支付 (NATIVE)
#### 模式一
提供一个生成支付二维码链接的函数，把url生成二维码给用户扫。
```js
var url = wxpay.createMerchantPrepayUrl({ product_id: '123456' });
```

商户后台收到微信的回调之后，调用 createUnifiedOrder() 生成预支付交易单，将结果的XML数据返回给微信。

[什么是模式一？](http://pay.weixin.qq.com/wiki/doc/api/native.php?chapter=6_4)

#### 模式二
直接调用 createUnifiedOrder() 函数生成预支付交易单，将结果中的 code_url 生成二维码给用户扫。

[什么是模式二？]()

### 公众号支付 (JS API)

生成JS API支付参数，发给页面
```js
wxpay.getBrandWCPayRequestParams({
	openid: '微信用户 openid',
	body: '公众号支付测试',
    detail: '公众号支付测试',
	out_trade_no: '20150331'+Math.random().toString().substr(2, 10),
	total_fee: 1,
	spbill_create_ip: '192.168.2.210',
	notify_url: 'http://wxpay_notify_url'
}, function(err, result){
	// in express
    res.render('wxpay/jsapi', { payargs:result })
});
```

网页调用参数（以ejs为例）
```js
WeixinJSBridge.invoke(
	"getBrandWCPayRequest", <%-JSON.Stringify(payargs)%>, function(res){
		if(res.err_msg == "get_brand_wcpay_request:ok" ) {
    		// success
    	}
});
```

### 中间件

商户服务端处理微信的回调（express为例）
```js
var router = express.Router();
var wxpay = require('weixin-pay');

// 原生支付回调
router.use('/wxpay/native/callback', wxpay.useWXCallback(function(msg, req, res, next){
	// msg: 微信回调发送的数据
}));

// 支付结果异步通知
router.use('/wxpay/notify', wxpay.useWXCallback(function(msg, req, res, next){

}));
```
