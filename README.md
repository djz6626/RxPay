![](https://github.com/Cuieney/RxPay/blob/master/img/logo.png)


## What's RxPay ?

**让支付从此简单下去，一键支付功能，支持支付宝支付，微信支付**

## 使用步骤
### step 1（Gradle）
#### java 项目配置

```
dependencies {
    	compile 'com.cuieney:rxpay-api:2.1.2'
    	annotationProcessor 'com.cuieney:rxpay-compiler:2.1.0'
        //如果你项目配置了kotlin请忽略下面这行的配置（否则会报错 Failed resolution of: Lkotlin/jvm/internal/Intrinsics）
        compile "org.jetbrains.kotlin:kotlin-stdlib-jre7:$kotlin_version"
}

```

#### kotlin 项目配置

```

apply plugin: 'kotlin-kapt'

dependencies {
    compile 'com.cuieney:rxpay-api:2.1.2'
    kapt 'com.cuieney:rxpay-compiler:2.1.0'
    ...
}

```


### step 2
在你的AndroidManifest文件中添加权限
#### AndroidManifest
```

<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

```
### step 3
如果你需要用到微信支付的话，请仔细看一下步骤
1.在你要使用微信支付的地方添加一下注解

```
@WX(packageName = "微信支付注册keystore时候的包名")
public class MainActivity extends AppCompatActivity
```
2.在AndroidManifest添加你微信支付的appid 和商户号，apiKey（商户平台设置的密钥key）

```
   <meta-data
            android:name="WX_APPID"
            android:value="xxxxx"/>
   //非必填项，此处填写后，请求json的partnerId字段就可以不填
   <meta-data
            android:name="PARTNER_ID"
            android:value="xxxx"/>
    //非必填项，此处填写后，请求json的sign字段就可以不填（即App端签名）
   <meta-data
            android:name="API_KEY"
            android:value="xxxxx"/>

```
3.在AndroidManifest的微信支付回调页面的Activity

```
     <activity
            android:name="xxx.xxx.xxx.wxapi.WXPayEntryActivity"
            android:exported="true"
            android:screenOrientation="portrait"
            android:windowSoftInputMode="adjustPan"
            />

```
上面的xxx.xxx.xxx就是你微信支付注册keystore时候的包名，报错没关系，编译会生成对应的Activity。

对应的支付宝支付AndroidManifest需要添加的是

```
<activity
    android:name="com.alipay.sdk.app.H5PayActivity"
    android:configChanges="orientation|keyboardHidden|navigation|screenSize"
    android:exported="false"
    android:screenOrientation="behind"
    android:windowSoftInputMode="adjustResize|stateHidden" >
</activity>
 <activity
    android:name="com.alipay.sdk.app.H5AuthActivity"
    android:configChanges="orientation|keyboardHidden|navigation"
    android:exported="false"
    android:screenOrientation="behind"
    android:windowSoftInputMode="adjustResize|stateHidden" >
</activity>

```


### step 4
发起支付功能

1.发起支付宝支付请求

```
 rxPay.requestAlipay("服务器产生的订单信息")
                .subscribe(new Consumer<Boolean>() {
                    @Override
                    public void accept(Boolean aBoolean) throws Exception {
                        payState.setText("阿里支付状态："+aBoolean);
                    }
                }, new Consumer<Throwable>() {
                    @Override
                    public void accept(Throwable throwable) throws Exception {
                        payState.setText("阿里支付状态："+throwable.getMessage());
                    }
                });

```

2.发起微信支付请求

```
 rxPay.requestWXpay(new JSONObject(“服务器生成订单的后拼接成下图这种json”))
                .subscribe(new Consumer<Boolean>() {
                    @Override
                    public void accept(Boolean aBoolean) throws Exception {
                        payState.setText("微信支付状态："+aBoolean);
                    }
                }, new Consumer<Throwable>() {
                    @Override
                    public void accept(Throwable throwable) throws Exception {
                        payState.setText("微信支付状态："+throwable.getMessage());
                    }
                });
```
对应的json格式参考


```
{
    "nonceStr": "非必填项",
    "partnerId": "非必填项(如果不填此选项，必须在AndroidManifest配置PARTNER_ID)",
    "packageValue": "非必填项",
    "prepayId": "必填项",
    "sign": "非必填项（如果不填此选项，必须在AndroidManifest配置API_KEY）",
    "timeStamp": "非必填项"
}


```


[code sample](https://github.com/Cuieney/RxPay/tree/master/app/src/main/java/com/cuieney/android/rxpay)

#### 混淆

```
-keep class com.alipay.android.app.IAlixPay{*;}
-keep class com.alipay.android.app.IAlixPay$Stub{*;}
-keep class com.alipay.android.app.IRemoteServiceCallback{*;}
-keep class com.alipay.android.app.IRemoteServiceCallback$Stub{*;}
-keep class com.alipay.sdk.app.PayTask{ public *;}
-keep class com.alipay.sdk.app.AuthTask{ public *;}
-keep class com.alipay.sdk.app.H5PayCallback {
    <fields>;
    <methods>;
}
-keep class com.alipay.android.phone.mrpc.core.** { *; }
-keep class com.alipay.apmobilesecuritysdk.** { *; }
-keep class com.alipay.mobile.framework.service.annotation.** { *; }
-keep class com.alipay.mobilesecuritysdk.face.** { *; }
-keep class com.alipay.tscenter.biz.rpc.** { *; }
-keep class org.json.alipay.** { *; }
-keep class com.alipay.tscenter.** { *; }
-keep class com.ta.utdid2.** { *;}
-keep class com.ut.device.** { *;}

-dontwarn com.alipay.android.phone.mrpc.core.**

```
#### Tips
* 如果你的项目中有之前集成了支付宝，请记得删除了alipaySdk-xxxxxxxx.jar，不然会冲突。
* 对于调起微信支付的json的字段请参考以上的json

#### 问题
发现bug或好的建议欢迎 [issues](https://github.com/Cuieney/RxPay/issues) or
Email <cuieney@163.com>

#### 微信交流群
![](https://github.com/Cuieney/RxPay/blob/master/img/wechat.png)

### License

> ```
> Copyright 2017 Cuieney
>
> Licensed under the Apache License, Version 2.0 (the "License");
> you may not use this file except in compliance with the License.
> You may obtain a copy of the License at
>
>    http://www.apache.org/licenses/LICENSE-2.0
>
> Unless required by applicable law or agreed to in writing, software
> distributed under the License is distributed on an "AS IS" BASIS,
> WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
> See the License for the specific language governing permissions and
> limitations under the License.
