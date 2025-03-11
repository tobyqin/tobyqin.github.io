---
title: 各平台免费翻译API
categories: [Tech]
tags: [free, api, translator]
date: 2020-02-10
---

收集一下，用的上。

<!-- more -->

## Google

> https://translate.google.cn/translate_a/single?client=gtx&dt=t&dj=1&ie=UTF-8&sl=auto&tl=zh_CN&q=hello
>
> https://translate.google.cn/translate_a/single?client=gtx&dt=t&dj=1&ie=UTF-8&sl=auto&tl=en_US&q=你好

```javascript
{
    "sentences": [
        {
            "trans": "你好",
            "orig": "hello",
            "backend": 1
        }
    ],
    "src": "en",
    "confidence": 1,
    "spell": {},
    "ld_result": {
        "srclangs": [
            "en"
        ],
        "srclangs_confidences": [
            1
        ],
        "extended_srclangs": [
            "en"
        ]
    }
}
```

## BING 必应

> https://api.microsofttranslator.com/v2/Http.svc/Translate?appId=AFC76A66CF4F434ED080D245C30CF1E71C22959C&from=&to=zh&text=hello
>
> https://api.microsofttranslator.com/v2/Http.svc/Translate?appId=AFC76A66CF4F434ED080D245C30CF1E71C22959C&from=&to=en&text=你好

```html
<string xmlns="http://schemas.microsoft.com/2003/10/Serialization/"
  >How are you doing</string
>
```

## Youdao 有道

> https://fanyi.youdao.com/translate?&doctype=json&type=AUTO&i=hello
>
> https://fanyi.youdao.com/translate?&doctype=json&type=AUTO&i=你好

```javascript
{
    "type": "ZH_CN2EN",
    "errorCode": 0,
    "elapsedTime": 1,
    "translateResult": [
        [
            {
                "src": "你好",
                "tgt": "hello"
            }
        ]
    ]
}
```

type 类型：

```
ZH_CN2EN 中文　 　英语
ZH_CN2JA 中文　 　日语
ZH_CN2KR 中文　 　韩语
ZH_CN2FR 中文　 　法语
ZH_CN2RU 中文　 　俄语
ZH_CN2SP 中文　 　西语
EN2ZH_CN 英语　 　中文
JA2ZH_CN 日语　 　中文
KR2ZH_CN 韩语　 　中文
FR2ZH_CN 法语　 　中文
RU2ZH_CN 俄语　 　中文
SP2ZH_CN 西语　 　中文
```
