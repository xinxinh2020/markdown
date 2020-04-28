

[TOC]

# 测试项

## 获取验证码接口

### 获取邮箱验证码

接口：

```shell
localhost:8001/user_verify/get_verification_code
```

POST参数：

```shell
{
	"user_name": "starlight1_1",
	"key": "SGVsbG9Xb3JsZDIwMjA="
}
```

user_name可以是系统用户名，手机号或邮箱，但必须确保该用户绑定了邮箱，否则会发送手机验证码而不是邮箱验证码。

key是固定的口令，仅提供给允许调用该接口的用户，如系统部。

#### 第一次调用

```shell
{
    "uuid": "8fc5a437-9369-4c7d-a8f8-1a7133a653d8",
    "code": 200,
    "info": "",
    "kind": "verification code",
    "total": 0,
    "spec": {
        "verification_code": "145993",
        "email": "****85787@qq.com",
        "telephone": "",
        "actually_send": true
    }
}
```

此时actually_send的值为true，用户的邮箱应收到一封邮件。

#### 3分钟内重复调用

```shell
{
    "uuid": "039c24bb-c7ff-45df-9705-684ba15e75d4",
    "code": 200,
    "info": "",
    "kind": "verification code",
    "total": 0,
    "spec": {
        "verification_code": "145993",
        "email": "****85787@qq.com",
        "telephone": "",
        "actually_send": false
    }
}
```

此时actually_send的值为false，返回的是上一次发送的验证码，不会再次向用户的邮箱发送邮件。



#### 1分钟内重复调用超过20次

```shell
{
    "uuid": "adaf32f6-7d8e-43f7-ba1d-46f808904c8e",
    "code": 1111,
    "info": "您使用的IP地址在单位时间内调用api接口的次数已达到上限",
    "kind": "",
    "total": 0,
    "spec": null
}
```

该接口有调用频率限制，1分钟调用超过20次会报错

#### 3分钟后再次调用

```shell
{
    "uuid": "3fcc3f85-6030-4ade-a495-c1820197dc1d",
    "code": 200,
    "info": "",
    "kind": "verification code",
    "total": 0,
    "spec": {
        "verification_code": "955426",
        "email": "****85787@qq.com",
        "telephone": "",
        "actually_send": true
    }
}
```

此时actually_send的值为true，用户的邮箱应收到一个新的验证码，旧验证码失效（可以暂时不用验证，在二次验证登录那里验证）。



### 获取手机验证码

接口：

```shell
localhost:8001/user_verify/get_verification_code
```

和获取邮件验证码是同个接口。

参数：

```shell
{
	"user_name": "starlight1_1",
	"key": "SGVsbG9Xb3JsZDIwMjA=",
	"by_telephone": true
}
```

user_name可以是系统用户名，手机号或邮箱，但必须确保该用户绑定了手机，否则会发送邮箱验证码而不是手机验证码。

如果用户同时绑定了邮箱和手机，默认会发送邮件验证码，可以通过by_telephone参数显式指定通过手机方式发送验证码。

#### 第一次

```shell
{
    "uuid": "f7fc8781-d750-4ff8-a9e3-296dc13239c9",
    "code": 200,
    "info": "",
    "kind": "verification code",
    "total": 0,
    "spec": {
        "verification_code": "480083",
        "email": "",
        "telephone": "155****6783",
        "actually_send": true
    }
}
```



#### 3分钟内重复调用

```shell
{
    "uuid": "758f2643-e6af-4d49-93a5-6a2cb7a22921",
    "code": 200,
    "info": "",
    "kind": "verification code",
    "total": 0,
    "spec": {
        "verification_code": "480083",
        "email": "",
        "telephone": "155****6783",
        "actually_send": false
    }
}
```



#### 1分钟调用超过20次

```shell
{
    "uuid": "54d5650d-2a3e-430b-a46a-f0b249f1150f",
    "code": 1111,
    "info": "您使用的IP地址在单位时间内调用api接口的次数已达到上限",
    "kind": "",
    "total": 0,
    "spec": null
}
```

#### 3分钟后再次调用

```shell
{
    "uuid": "8941e244-a6f3-4aaf-bcda-1dbd2d1f12bb",
    "code": 200,
    "info": "",
    "kind": "verification code",
    "total": 0,
    "spec": {
        "verification_code": "228575",
        "email": "",
        "telephone": "155****6783",
        "actually_send": true
    }
}
```

#### 1天调用超过6次

当1天调用超过6次以后，当天无论再调用多少次，返回的都是第6次发送的手机验证码，且不会再向用户发送短信。可以通过设置telephone_verification_code表的count_by_day为6，skip_time_check为1来模拟当天发送次数。

```shell
{
    "uuid": "20ef65bf-f009-4f32-b5ff-b694b3884729",
    "code": 200,
    "info": "",
    "kind": "verification code",
    "total": 0,
    "spec": {
        "verification_code": "322299",
        "email": "",
        "telephone": "155****6783",
        "actually_send": false
    }
}
```



## 登录二次验证接口

接口：

```shell
localhost:8000/short_term_token/name
```

和原登录接口一样。

出现以下情况之一，则不能直接使用密码登录，需要进行二次验证：

1. 使用不常用的IP地址登录的
2. 使用密码登录失败尝试次数过多的（大于5次）
4. 距离上次登录成功时间过久的（大于7天）

### 第一次使用密码登录

第一次登录的时候，肯定是需要进行二次验证的，直接使用密码登录会提醒用户改用二次验证登录：

```shell
{
    "uuid": "38e92b24-85b0-47c5-97c1-1c4e00a97bb4",
    "code": 200,
    "info": "请使用验证码进行登录",
    "kind": "verification_code",
    "total": 1,
    "spec": false
}
```

其中，kind等于verification_code表示用户需要二次验证（否则为token），spec用于提示前端告诉用户是通过手机还是通过邮箱发送验证码，true表示发送手机验证码，false表示发送邮件验证码



### 改用验证码登录

先参考上文的获取验证码接口调用一下发送验证码接口，得到验证码后通过验证码进行登录，分别需要测试通过手机验证码和邮件验证码登录的方式。

#### 通过邮件验证码登录

POST请求参数：

```shell
{
    "username": "652085787@qq.com",
    "verification_code": "422823"
}
```

verification_code填入邮件收到的验证码。

##### 获取验证码后马上登录

```shell
{
    "uuid": "02509a99-baff-4be8-bd97-580cb2bd2a10",
    "code": 200,
    "info": "",
    "kind": "token",
    "total": 1,
    "spec": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MjE2MjgsIm9yaV91c2VyX25hbWUiOiIiLCJ1c2VyX25hbWUiOiJzdGFybGlnaHQxXzEiLCJ1aWQiOjY3ODUsImdyb3VwX25hbWUiOiJzdGFybGlnaHQxIiwiZ3JvdXBfaWQiOjgzMTMsInN0YXR1cyI6NSwiZXhwIjoxNTg4MDE5NDQ0fQ.EcCvv4eSl8_7UgvHurBOSIkVm4YJRXPaENapGq6D24Rzngc0FXlBuQIpeTbs3FL1FgFAqBbW8xxiQ9vh8GG-Ng"
}
```



##### 1分钟内使用该验证码都可以成功登录

```shell
{
    "uuid": "5bbb08c7-809f-4971-b56e-7c52a2ccbe01",
    "code": 200,
    "info": "",
    "kind": "token",
    "total": 1,
    "spec": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MjE2MjgsIm9yaV91c2VyX25hbWUiOiIiLCJ1c2VyX25hbWUiOiJzdGFybGlnaHQxXzEiLCJ1aWQiOjY3ODUsImdyb3VwX25hbWUiOiJzdGFybGlnaHQxIiwiZ3JvdXBfaWQiOjgzMTMsInN0YXR1cyI6NSwiZXhwIjoxNTg4MDE5NTI4fQ.LPZdAfmAZIeEldAvR-xI51QPE2gU4bG9qLp6Lm0iCkhh4J8oMAm66ig4_wiqQPfYQ_fP9FX1vm5DoWAuS3J7Pg"
}
```



##### 1分钟内调用超过20次报错

```shell
{
    "uuid": "1472ad63-2869-402e-9b66-0ac02d7b1c65",
    "code": 1111,
    "info": "您使用的IP地址在单位时间内调用api接口的次数已达到上限",
    "kind": "",
    "total": 0,
    "spec": null
}
```



##### 3分钟后使用该验证码会报错

```shell
{
    "uuid": "3f7c3cb4-8a92-4b6e-9e41-6bc88fab3468",
    "code": 10475,
    "info": "验证码错误或已过期",
    "kind": "",
    "total": 0,
    "spec": null
}
```



##### 使用重新发送新的验证码

重新调用一下发送邮件验证码的接口（或者通过界面），获取一下新的邮件验证码，使用新的验证码才能成功登录：

```shell
{
    "uuid": "e94f6886-53d5-4fc0-98f5-edbf34049b3d",
    "code": 200,
    "info": "",
    "kind": "token",
    "total": 1,
    "spec": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MjE2MjgsIm9yaV91c2VyX25hbWUiOiIiLCJ1c2VyX25hbWUiOiJzdGFybGlnaHQxXzEiLCJ1aWQiOjY3ODUsImdyb3VwX25hbWUiOiJzdGFybGlnaHQxIiwiZ3JvdXBfaWQiOjgzMTMsInN0YXR1cyI6NSwiZXhwIjoxNTg4MDE5OTA0fQ.VfTCo89tRZc4yNu8NHKI7XuQoJrbVdBLPJEj5qsRZO6B2gg5XgpZ3M2Ksi6TBi2hFPjgtzxaameMM5QasULNiw"
}
```



#### 通过手机验证码

请求参数：

```shell
{
    "username": "652085787@qq.com",
    "verification_code": "422823"
}
```

verification_code填入手机收到的验证码。

##### 获取验证码后马上登录

此时应能成功获取到token：

```shell
{
    "uuid": "6101f4b6-089e-4403-bbeb-cc15d1c73009",
    "code": 200,
    "info": "",
    "kind": "token",
    "total": 1,
    "spec": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MjE2MjgsIm9yaV91c2VyX25hbWUiOiIiLCJ1c2VyX25hbWUiOiJzdGFybGlnaHQxXzEiLCJ1aWQiOjY3ODUsImdyb3VwX25hbWUiOiJzdGFybGlnaHQxIiwiZ3JvdXBfaWQiOjgzMTMsInN0YXR1cyI6NSwiZXhwIjoxNTg4MDIzNjg0fQ.WJ4t8lnA4Dfhp5Gh-n-o6s2hc_ztaT0ZA8jAK24shT-G5BtRAWUaXvCF_ss039nF-L10oFrcBuJBxfhvK4DtXw"
}
```



##### 1分钟内使用该验证码都可以成功登录

重复上一步操作即可

##### 3分钟后使用该验证码会报错

```shell
{
    "uuid": "f42aaa19-ca2d-4d9f-846a-19139575bc27",
    "code": 10475,
    "info": "验证码错误或已过期",
    "kind": "",
    "total": 0,
    "spec": null
}
```



##### 使用重新发送新的验证码

```shell
{
    "uuid": "33ece17b-0ab7-4ea4-8c44-be4a592283da",
    "code": 200,
    "info": "",
    "kind": "token",
    "total": 1,
    "spec": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MjE2MjgsIm9yaV91c2VyX25hbWUiOiIiLCJ1c2VyX25hbWUiOiJzdGFybGlnaHQxXzEiLCJ1aWQiOjY3ODUsImdyb3VwX25hbWUiOiJzdGFybGlnaHQxIiwiZ3JvdXBfaWQiOjgzMTMsInN0YXR1cyI6NSwiZXhwIjoxNTg4MDIzNzQ1fQ.PKiRKuDeAN3BPzT11lidTRzlItn6e7oofwi1gBt0vflyNUyY2Bio-uaU2ZEqyY6Ux1l8B4_qoqgWsI_FvRyEgQ"
}
```

验证码失效后，重新发送得到新的验证码，使用新的验证码可以登录

##### 当天验证码发送次数达到限制以后当天验证码不失效

当天发送短信验证码达到6次以后，后续登录都使用该验证码，该验证码当天不会过期。可以通过设置telephone_verification_code表的count_by_day为6，skip_time_check为1来模拟当天发送次数达到6次时的情况。在发送完第6个验证码后，等等3分钟以上的时间，使用该验证码进行登录，此时应该能够成功。



##### 当天验证码发送次数达到限制以后，第二天该验证码仍然失效

上一步测试完以后，第二天再重复同样的测试，登录应该失败。



### 当使用二次验证登录成功后，下一次登录使用密码登录

参数：

```shell
{
    "username": "starlight1_1",
    "password": "{{PASS1}}"
}
```



#### 马上使用密码登录

使用二次验证成功登录后，马上使用密码登录，此时应该能够成功登录：

```shell
{
    "uuid": "2274055f-42d4-4428-9f89-2a7802b0a796",
    "code": 200,
    "info": "",
    "kind": "token",
    "total": 1,
    "spec": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MjE2MjgsIm9yaV91c2VyX25hbWUiOiIiLCJ1c2VyX25hbWUiOiJzdGFybGlnaHQxXzEiLCJ1aWQiOjY3ODUsImdyb3VwX25hbWUiOiJzdGFybGlnaHQxIiwiZ3JvdXBfaWQiOjgzMTMsInN0YXR1cyI6NSwiZXhwIjoxNTg4MDI5OTM0fQ.c3kLWWRLAuAlX3Do4f2JXw66CU1O3G4NoDFcx86hgRF58HAmXPFgYJK_wi0As1X-5epn5ggDQ6Z6V8tE1-sSRw"
}
```



#### 1分钟内密码错误次数过多

Todo: 故意在1分钟内把密码输错5次，此时会被系统要求进行二次验证：

```shell
{
    "uuid": "11c8b5e6-8a41-43ab-a474-bac0a058f8c4",
    "code": 200,
    "info": "您的账号密码输入错误次数过多，请使用验证码登录",
    "kind": "verification_code",
    "total": 1,
    "spec": false
}
```

使用验证码登录成功后再使用密码登录，又能正常登录

```shell
{
    "uuid": "fe518020-3b95-434d-9d5b-43123ff94cc5",
    "code": 200,
    "info": "",
    "kind": "token",
    "total": 1,
    "spec": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MjE2MjgsIm9yaV91c2VyX25hbWUiOiIiLCJ1c2VyX25hbWUiOiJzdGFybGlnaHQxXzEiLCJ1aWQiOjY3ODUsImdyb3VwX25hbWUiOiJzdGFybGlnaHQxIiwiZ3JvdXBfaWQiOjgzMTMsInN0YXR1cyI6NSwiZXhwIjoxNTg4MDMyMTE3fQ.IHdL6CIiLq7ozLkfPXGsxvlZ6lgJ5YsLnMYUxTGT5Z3BuzY7nM3c6YT1Whwjix5vg6JXzIyLCZ4_xmKqoeZ6KA"
}
```



#### 累计密码失败次数

只要不在1分钟内连续输错5次，1分钟密码失败次数会重置



#### 换个网段登录需要二次验证

可以在POSTMAN的请求头里修改X-Real-IP的地址，触发该场景。改变网段(24位网段)会被系统要求进行二次验证：

```shell
{
    "uuid": "9cd8b28b-e61f-4c2c-9d4b-ce068794fb0b",
    "code": 200,
    "info": "检测到您登录的网段发生变化，请使用验证码登录",
    "kind": "verification_code",
    "total": 1,
    "spec": false
}
```

使用新网段进行二次验证后再使用密码登录，此时应该能在新网段通过密码登录：

```shell
{
    "uuid": "5201e662-a901-4873-8612-27b4b0e140ba",
    "code": 200,
    "info": "",
    "kind": "token",
    "total": 1,
    "spec": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MjE2MjgsIm9yaV91c2VyX25hbWUiOiIiLCJ1c2VyX25hbWUiOiJzdGFybGlnaHQxXzEiLCJ1aWQiOjY3ODUsImdyb3VwX25hbWUiOiJzdGFybGlnaHQxIiwiZ3JvdXBfaWQiOjgzMTMsInN0YXR1cyI6NSwiZXhwIjoxNTg4MDMwNjYyfQ.b_-MZQH-oFcp6AajN4Hk-69nSYfoaqNVgq_4hlJPtbgH9py2Zq5wy5YSnhkXQDnVxMMuDKhi8NdJXDohVa3Giw"
}
```

二次验证登录成功后，该网段会被记录到最近登录网段中，使用该网段的不同IP地址都不需要二次验证。最多会记录两个网段。

#### 一周未登录需要进行二次验证

修改login_security_enhancement表的last_login字段，把时间改为一周前，此时系统应该要求用户进行二次验证：

```shell
{
    "uuid": "abe8669d-d257-4c72-b5fe-f254dae2214e",
    "code": 200,
    "info": "您已经超过7天没登录系统，请使用验证码登录",
    "kind": "verification_code",
    "total": 1,
    "spec": false
}
```

使用二次验证登录成功后，在使用密码登录能够正常登录：

```shell
{
    "uuid": "9a9ba2ec-c9dc-4cc1-87f7-8d738fd7e223",
    "code": 200,
    "info": "",
    "kind": "token",
    "total": 1,
    "spec": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6MjE2MjgsIm9yaV91c2VyX25hbWUiOiIiLCJ1c2VyX25hbWUiOiJzdGFybGlnaHQxXzEiLCJ1aWQiOjY3ODUsImdyb3VwX25hbWUiOiJzdGFybGlnaHQxIiwiZ3JvdXBfaWQiOjgzMTMsInN0YXR1cyI6NSwiZXhwIjoxNTg4MDMyMzUzfQ.HOyBoReLfMvqcB6IOpY2Fe97NMOA6LEBbvOyFyKA8r_uVPxHSu0OluoIz-gkbVqeXH6bsXS9X-8470gUw4SWMg"
}
```







