# 统一支付API接口

## 统一下单支付

### 接口地址 <https://pay.51xshi.com/unify>

***

#### 请求参数类型

- ##### 支付渠道列表

| channel | 渠道       | method |
| ------- | ---------- | ------ |
| alipay  | 支付宝支付 | POST   |
| wechat  | 微信支付   | POST   |

- ##### api列表

| method  | 说明         | 参数         | 返回值 | 平台差异说明     |
| ------- | ------------ | ------------ | ------ | ---------------- |
| mp      | 公众号支付   | array $order | Json   | 微信支付         |
| wap     | 手机网站支付 | array $order | Json   | 微信支付  支付宝 |
| app     | APP 支付     | array $order | Json   | 微信支付  支付宝 |
| web     | 电脑支付     | array $order | Json   | 支付宝           |
| scan    | 扫码支付     | array $order | Json   | 微信支付  支付宝 |
| miniapp | 小程序支付   | array $order | Json   | 微信支付  支付宝 |

- ##### 系统参数

| key         | 类型   | 必选/可选 | 说明                                                                                      |
| ----------- | ------ | --------- | ----------------------------------------------------------------------------------------- |
| app_key     | String | *必选     | 请求来源系统的名称                                                                        |
| app_version | String | 可选      | 用于统计客户端的版本号                                                                    |
| timestamp   | long   | *必选     | 时间戳，格林威治时间起至现在的总秒数（13位时间戳）本系统允许客户端请求最大时间误差为5分钟 |
| sign        | String | *必选     | API输入参数签名结果，签名算法介绍参照下面的介绍                                           |

- ##### 请求参数

| key          | 类型（最大长度） | 必选/可选 | 说明                                                                                             | 平台差异说明                   |
| ------------ | ---------------- | --------- | ------------------------------------------------------------------------------------------------ | ------------------------------ |
| out_trade_no | String(32)       | *必选     | 订单号 全局唯一                                                                                  | 微信支付  支付宝               |
| body         | String(128)      | *必选     | 商品描述                                                                                         | 微信支付                       |
| subject      | String           | *必选     | 商品描述 注意：不可使用特殊字符，如 /，=，& 等。                                                 | 支付宝                         |
| total_fee    | Decimail         | *必选     | 支付金额 微信支付 单位/分 支付宝支付 单位/元                                                     | 微信支付  支付宝               |
| http_method  | GET              | 可选      | 仅（web/wap）模式有效 如果想在 wap 支付时使用 GET 方式提交，请加上此参数。默认使用 POST 方式提交 | 支付宝                         |
| detail       | String(6000)     | 可选      | 商品详情                                                                                         | 微信支付                       |
| goods_detail | GoodsDetail[]    | 可选      | 订单包含的商品列表信息，json格式，其它说明详见商品明细说明                                       | 支付宝支付                     |
| └goods_id    | String(32)       | *必选     | 传入 goods_detail 必选 商品编号                                                                  | 支付宝支付                     |
| └goods_name  | String(32)       | *必选     | 传入 goods_detail 必选 商品名称                                                                  | 支付宝支付                     |
| └quantity    | Number(10)       | *必选     | 传入 goods_detail 必选 商品数量                                                                  | 支付宝支付                     |
| └price       | Price(9)         | *必选     | 传入 goods_detail 必选 商品单价，单位为元                                                        | 支付宝支付                     |
| openid       | String           | 可选      | 用户openid                                                                                       | 微信支付（小程序、公众号）必选 |
| notify_url   | String           | 可选      | 结果回调通知地址                                                                                 | 微信支付 支付宝                |
| return_url   | String           | 可选      | 请求回调地址 HTTP/HTTPS开头字符串                                                                | 支付宝                         |

***

#### 签名算法

- ##### 1.根据请求参数，对签名进行验证，签名不合法的请求将会被拒绝，生成签名的步骤如下

  - 将所有业务请求参数及系统参数（app_key和timestamp），按字母先后顺序排序
  - 参数名称和参数值链接成一个字符串A
  - 在字符串A的首尾加上app_secret组成一个新字符串B
  - 对字符串B进行md5,取32位小写，得到签名sign

- ##### 2.举例

  - 系统参数：app_key=littlezov，app_secret=4e686d87ae838d7a448d8ae0dd417662
  - 支付渠道：channel=wechat
  - 支付类型：method=app
  - 时间戳：timestamp=1501035945348
  - 请求的业务参数为：out_trade_no=20201111010855001,body=会员购买,total_fee=990
  - 请求地址：<https://pay.51xshi.com/unify/${channel}/${method}>
  - 签名生成如下
  
     1. 排序后为：app_key=littlezov, app_secret=4e686d87ae838d7a448d8ae0dd417662, body=会员购买, channel=wechat, method=scan, out_trade_no=20201111010855001, timestamp=1605030459932, total_fee=990

    2. 参数名称和参数值链接成一个字符串string=app_keylittlezovapp_secret4e686d87ae838d7a448d8ae0dd417662body会员购买channelwechatmethodscanout_trade_no20201111010855001timestamp1605030459932total_fee990

    3.在字符串string的首尾加上app_secret的值组成一个新字符串signString=4e686d87ae838d7a448d8ae0dd417662app_keylittlezovapp_secret4e686d87ae838d7a448d8ae0dd417662body会员购买channelwechatmethodscanout_trade_no20201111010855001timestamp1605030459932total_fee990typewechat4e686d87ae838d7a448d8ae0dd417662

    4.对字符串signString进行两次md5得到签名sign签名sign=md5(md5(4e686d87ae838d7a448d8ae0dd417662app_keylittlezovapp_secret4e686d87ae838d7a448d8ae0dd417662body会员购买channelwechatmethodscanout_trade_no20201111010855001timestamp1605030459932total_fee990typewechat4e686d87ae838d7a448d8ae0dd417662)) , 即：72768d348d9d4dcce48772dd65e83286

    5.最终的请求地址为 ： <https://pay.51xshi.com/unify/wechat/app>

    6.请求参数为

    ``` 
     {
        "app_key": "littlezov",
        "body": "会员购买",
        "channel": "wechat",
        "method": "scan",
        "out_trade_no": 20201111010855001,
        "timestamp": 1605030459932,
        "total_fee": 990,
        "type": "wechat"
        "sign": "72768d348d9d4dcce48772dd65e83286",
    }
    ```

#### 返回参数code字典表

- ##### 1.安全认证相关的错误码code字典表
  
| code   | 含义                                               |
| ------ | -------------------------------------------------- |
| 10001  | 非法请求，缺少系统级参数（app_key,sign,timestamp） |
| 10002  | 非法请求，未知的调用方                             |
| 10003  | 非法请求，请求过期                                 |
| 10004  | 非法请求，签名sign验证失败                         |
| 100011 | 非法请求，缺少Token                                |
| 100012 | 非法请求，错误的Token                              |

- ##### 2.业务接口通用错误码code字典表
  
| code | 含义                                                                                                                     |
| ---- | ------------------------------------------------------------------------------------------------------------------------ |
| 0    | 处理成功，空数据场景：提交给服务方的业务处理完成，或者查询数据成功；但无返回数据  对应的message 信息，可以给终端用户展示 |
| 200  | 处理成功，场景：提交给服务方的业务处理完成，或者查询数据成功；调用方可以直接进行跳转成功页面或数据展示                   |
| 300  | 多种选择，多用户请求接口返回了接口列表或者服务ip列表，后续可从列表中取有个地址进行请求，大部分未后端多地址场景           |
| 301  | 永久移动，接口地址已经变更本地址不在提供服务，请使用新地址请求                                                           |
| 302  | 临时移动，多用户主服务宕机发生地址临时变更场景                                                                           |
| 303  | 查看其他位置，多用于如请求回调地址调整，如授权登录成功后跳转，支付成功后跳转                                             |
| 304  | 未修改，请求更新不生效 多用于更新数据，但数据和旧的数据一直未发生更改                                                    |
| 400  | 非法请求，默认非法请求返回                                                                                               |
| 401  | 非法请求未授权，缺少授权参数                                                                                             |
| 409  | 冲突，发生重复请求，统一参数多次请求                                                                                     |
| 417  | 参数非法，场景：缺少参数、参数非空验证失败、参数格式错误、业务逻辑验证失败等，对应的message 信息，可以给终端用户展示     |
| 417  | 参数非法，场景：缺少参数、参数非空验证失败、参数格式错误、业务逻辑验证失败等，对应的message 信息，可以给终端用户展示     |
| 500  | 接口发送异常: 请联系管理员                                                                                               |
| 其他 | 其他code,参考具体的接口定义，多用于客户端特殊分支逻辑处理 或异常页面跳转                                                 |

#### 返回参数

- ##### json 对象

| 参数      | 类型    | 含义                           |
| --------- | ------- | ------------------------------ |
| code      | Integer | 结果码，0 or 200成功，其他失败 |
| message   | String  | 结果消息                       |
| timestamp | long    | 数据返回时间戳                 |
| result    | Array   | 数组，具体参考下面的result对象 |

- ##### result对象

  ***支付类***
  
  [微信支付 返回结果](https://pay.weixin.qq.com/wiki/doc/api/native.php?chapter=9_1)

  [支付宝支付 响应示例](https://opendocs.alipay.com/apis/api_1/alipay.trade.app.pay)

  ***授权类***
   >**尽请期待**
  