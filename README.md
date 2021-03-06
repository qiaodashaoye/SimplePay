# SimplePay

[![License](https://img.shields.io/badge/license-Apache%202-green.svg)](https://www.apache.org/licenses/LICENSE-2.0)
[![Download](https://api.bintray.com/packages/qpglibs/maven/SimplePay/images/download.svg)](https://bintray.com/qpglibs/maven/SimplePay/_latestVersion)
集成微信和支付宝支付，微信支付遇到的坑，我来帮你填，App内嵌支付如此简单。
# 一、集成步骤

> implementation 'com.qpg:paylib:1.0.6'

 ```
allprojects {
    repositories {
        jcenter()
        google()
        //支付宝需加入
        flatDir {
            dirs 'libs', '../paylib/libs'
        }
    }

}
 ```  

# 二、使用
### 微信支付使用

```java
        //1.创建微信支付请求
        WechatPayReq wechatPayReq = new WechatPayReq.Builder()
                .with(this) //activity实例
                .setAppId(appid) //微信支付AppID
                .setPartnerId(partnerid)//微信支付商户号
                .setPrepayId(prepayid)//预支付码
//								.setPackageValue(wechatPayReq.get)//"Sign=WXPay"
                .setNonceStr(noncestr)
                .setTimeStamp(timestamp)//时间戳
                .setSign(sign)//签名
                .create();
        //2.发送微信支付请求
        PayAPI.getInstance().sendPayRequest(wechatPayReq);

```
>注意：prepayid(预支付码)一定要正确，里面包含订单的金额，。

> 关于微信支付完成（失败、成功、取消）后的回调，必须在与包名同级下建立名为
wxapi的文件夹，并在文件夹下建立WXPayEntryActivity类并实现IWXAPIEventHandler接口，
所建的类名必须是WXPayEntryActivity，不然不会执行onResp()回调方法。

### 支付宝支付使用
 > 写在前面：
 ```java
  /** 商户私钥，pkcs8格式 */
  /** 如下私钥，RSA_PRIVATE 或者 RSA2_PRIVATE 只需要填入一个 */
  /** 如果商户两个都设置了，优先使用 RSA2_PRIVATE */
  /** RSA2_PRIVATE 可以保证商户交易在更加安全的环境下进行，建议使用 RSA2_PRIVATE */
  /** 获取 RSA2_PRIVATE，建议使用支付宝提供的公私钥生成工具生成， */
  /** 工具地址：https://doc.open.alipay.com/docs/doc.htm?treeId=291&articleId=106097&docType=1 */
```
                  
#### 支付宝支付第一种方式(不建议用这种方式，商户私钥暴露在客户端，极其危险，推荐用第二种支付方式)
```java

        //1.创建支付宝支付配置
        AliPayAPI.Config config = new AliPayAPI.Config.Builder()
                  .setRsaPrivate("") //设置RSA私钥
                  .setRsa2Private("") //设置RSA2私钥
                  .setRsaPublic("")//设置公钥
                  .setAppid(PayConstants.Appid)//设置appid
                  .create();

        //2.创建支付宝支付请求
        AliPayReq aliPayReq = new AliPayReq.Builder()
                .with(activity)//Activity实例
                .apply(config)//支付宝支付通用配置
                .setOutTradeNo(outTradeNo)//设置唯一订单号
                .setPrice(price)//设置订单价格
                .setSubject(orderSubject)//设置订单标题
                .setBody(orderBody)//设置订单内容 订单详情
                .setCallbackUrl(callbackUrl)//设置回调地址
                .create()//
                .setOnAliPayListener(new OnAliPayListener);//支付宝支付的回调，若不需要回调，可直接传null

        //3.发送支付宝支付请求
        PayAPI.getInstance().sendPayRequest(aliPayReq);

```

#### 支付宝支付第二种方式(**强烈推荐**)

```java
         AliPayReq2 aliPayReq = new AliPayReq2.Builder()
                 .with(MainActivity.this)//Activity实例
                 .setSignedAliPayOrderInfo(info) //后台返回的RSA加密后的支付宝支付订单信息
                 .create()//
                 .setOnAliPayListener(new OnAliPayListener);//支付宝支付的回调，若不需要回调，可直接传null
         PayAPI.getInstance().sendPayRequest(aliPayReq);
```

## 帮助

### 微信支付官方文档 支付流程
https://pay.weixin.qq.com/wiki/doc/api/app/app.php?chapter=8_5

### 支付宝支付官方文档 支付流程
https://doc.open.alipay.com/docs/doc.htm?spm=a219a.7629140.0.0.sdGXaH&treeId=204&articleId=105296&docType=1

### 支付宝支付的密钥处理体系文档。
https://doc.open.alipay.com/docs/doc.htm?spm=a219a.7629140.0.0.1wPnBT&treeId=204&articleId=106079&docType=1
