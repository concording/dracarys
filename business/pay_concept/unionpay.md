从2020年10月10日开始，支付产品技术群启动新一轮的技术分享。 本次分享的主题是 银联云闪付。 

为了支持在线直播，以及保留交流记录和文档，这一轮分享转移到企业级IM工具上进行。对比了企业微信、钉钉、飞书等软件后，确定使用飞书来做分享。 这些软件在用户数超过500之后都要求做企业认证，需要办理营业执照，预计费用在3000元左右。 为此从本期开始，**文章打赏费用将积累用于营业执照的申办。也期待大家多打赏，多支持， 感谢先。** 

![Back-end Roadmap](../../img/unionpay/ia_100000627.jpg)

![Back-end Roadmap](../../img/unionpay/ia_100000626.jpg)

![Back-end Roadmap](../../img/unionpay/ia_100000628.jpg)

![Back-end Roadmap](../../img/unionpay/ia_100000629.jpg)

问：这里哪些算传统无卡，哪些是新型无卡？打卡支付就是有卡支付吗？

答：传统无卡指代银联在线，网上银行这种传统网上支付。新型无卡主要体现在移动互联网支付，手机扫码，手持电子设备碰一碰（NFC），刷脸， 无感支付…… 打卡是指代有卡支付，主要有IC芯片卡，磁条卡，闪付卡，是我们最早接触的一种支付方式，包括ETC都是有卡范畴内，卡随“车”携带。

问： 老无卡和新无卡主要是产品形态上的区别嘛？还有其他的不一样嘛？

答： 简而言之 银联老无卡的业务基本是全球各大卡组织都会做的业务； 银联新无卡业务是中国特色，里面的业务只有网联和银联做

![Back-end Roadmap](../../img/unionpay/ia_100000630.jpg)

![Back-end Roadmap](../../img/unionpay/ia_100000631.jpg)

![Back-end Roadmap](../../img/unionpay/ia_100000632.jpg)

![Back-end Roadmap](../../img/unionpay/ia_100000633.jpg)

![Back-end Roadmap](../../img/unionpay/ia_100000634.jpg)

![Back-end Roadmap](../../img/unionpay/ia_100000635.jpg)

![Back-end Roadmap](../../img/unionpay/ia_100000636.jpg)

问：一张银行卡可以绑定多个手机，生成多个虚拟卡号Token吧？

答：是的。关联同一个主账号（PAN）,每个设备token不通，还有，每次解绑再绑定token都会变化，即使是同一设备。

![Back-end Roadmap](../../img/unionpay/ia_100000637.jpg)

问： 图中的PAN是什么？

答： PAN:Private Account Number ，银行卡主账号，也就是卡号。对应的有PIN:Personal Identification Number，持卡人密码

问： 图中扣款成功后，发卡行返回的授权信息又指的是？

答：就是扣款成功的应答信息。

![Back-end Roadmap](../../img/unionpay/ia_100000638.jpg)

![Back-end Roadmap](../../img/unionpay/ia_100000639.jpg)

![Back-end Roadmap](../../img/unionpay/ia_100000640.jpg)

问： 那么支付宝、微信 都是跟哪家银行合作的啊

答： 没get到您的点。支付宝和微信都是间联银行，你的合作指代是什么？

问：我以为支付宝、微信都是间接银行，没办法存储余额，要跟某个银行合作，把余额备用金存储起来

答： @李XX 以前网联是这种模式，后来在央行统筹下，第三方支付机构都需要在人行开立备付金账户。余额的钱实际就是存在这里的。我们是不是跑题了。

![Back-end Roadmap](../../img/unionpay/ia_100000642.jpg)

![Back-end Roadmap](../../img/unionpay/ia_100000641.jpg)

![Back-end Roadmap](../../img/unionpay/ia_100000643.jpg)

![Back-end Roadmap](../../img/unionpay/ia_100000644.jpg)

![Back-end Roadmap](../../img/unionpay/ia_100000645.jpg)


问：这里提到的各场景的支付限额是多少？

答：《条码支付业务规范（试行）》条文显示，条码支付额度将实行分级管理，使用静态条码的，单卡单日同商户累计限额500元；动态条码支付的限额分为三档：1000元、5000元和不限额。如果想要不限额，在扫码支付的时候需要采用数字证书或电子签证加上指纹或密码组合验证，这样单日累计限额由支付机构与你通过协议自主约定。如果没有数字证书和电子签名，只是通过密码、手机短信验证码或指纹等中的两种方式组合验证而实现的扫码支付，每天的额度是5000元。如果只用密码、指纹或手机短信三种方式中的一个进行安全验证，单日扫码支付的限额是1000元。

问：什么是跨屏支付？场景？

答： 简单理解，就是你用手机扫二维码，而这个二维码是在电子屏幕上的。显然这个解释很浅显，深层次的就是根据订单可以在电子屏幕动态显示二维码（二维码中包含了本次订单相关信息），再由客户去扫码，这就是跨屏支付的定义；银联提供了这个二维码系统，各入网机构可以利用这个“二维码支付”产品做自己的跨屏支付。

问： 屏上的二维码，是不是可以简单的理解为动态收款二维码

答： 是的

![Back-end Roadmap](../../img/unionpay/ia_100000646.jpg)

![Back-end Roadmap](../../img/unionpay/ia_100000647.jpg)

![Back-end Roadmap](../../img/unionpay/ia_100000648.jpg)

![Back-end Roadmap](../../img/unionpay/ia_100000649.jpg)

![Back-end Roadmap](../../img/unionpay/ia_100000650.jpg)




问：  这个等式是指等式右侧的这些费用都由商户承担？

答： 我认为是的，银联提供网络，银行提供卡服务，收单提供入口，商户售卖盈利，客户是消费群体。整体分析，只有商户是那个羔羊，因为商户需要收单+银行+银联组成的这个资金结算渠道，他缴费很合理。

![Back-end Roadmap](../../img/unionpay/ia_100000651.jpg)

![Back-end Roadmap](../../img/unionpay/ia_100000652.jpg)

![Back-end Roadmap](../../img/unionpay/ia_100000653.jpg)

![Back-end Roadmap](../../img/unionpay/ia_100000654.jpg)

![Back-end Roadmap](../../img/unionpay/ia_100000655.jpg)

![Back-end Roadmap](../../img/unionpay/ia_100000656.jpg)

![Back-end Roadmap](../../img/unionpay/ia_100000657.jpg)


文章已于2020-10-12修改