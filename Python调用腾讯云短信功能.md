---
typora-root-url: assets
---

### Python调用腾讯云短信功能 

#### 1 注册腾讯云并开通云短信功能

![image-20200609110334727](https://github.com/hanfuming/notebook/blob/master/assets/image-20200609110334727.png)

![image-20200609110346788](https://github.com/hanfuming/notebook/blob/master/assets/image-20200609110346788.png)

![image-20200609110419553](https://github.com/hanfuming/notebook/blob/master/assets/image-20200609110419553.png)

#### 2 发送短信

上述的准备工作做完中我们开通相关服务并获取到如下几个值：

- 创建应用，获取到 `appid` 和 `appkey`
- 创建签名，获取 `签名内容`
- 创建模板，获取 `模板ID`

接下来开始使用Python发送短信。

第一步：安装SDK

```shell
pip install qcloudsms_py
```

第二步：基于SDK发送短信

```shell
#!/usr/bin/env python
# -*- coding:utf-8 -*-
import ssl
# ssl._create_default_https_context = ssl._create_unverified_context
from qcloudsms_py import SmsMultiSender, SmsSingleSender
from qcloudsms_py.httpclient import HTTPError
def send_sms_single(phone_num, template_id, template_param_list):
    """
    单条发送短信
    :param phone_num: 手机号
    :param template_id: 腾讯云短信模板ID
    :param template_param_list: 短信模板所需参数列表，例如:【验证码：{1}，描述：{2}】，则传递参数 [888,666]按顺序去格式化模板
    :return:
    """
    appid = 42311  # 自己应用ID
    appkey = "b87123y42"  # 自己应用Key
    sms_sign = "Python"  # 自己腾讯云创建签名时填写的签名内容（使用公众号的话这个值一般是公众号全称或简称）
    sender = SmsSingleSender(appid, appkey)
    try:
        response = sender.send_with_param(86, phone_num, template_id, template_param_list, sign=sms_sign)
    except HTTPError as e:
        response = {'result': 1000, 'errmsg': "网络异常发送失败"}
    return response
def send_sms_multi(phone_num_list, template_id, param_list):
    """
    批量发送短信
    :param phone_num_list:手机号列表
    :param template_id:腾讯云短信模板ID
    :param param_list:短信模板所需参数列表，例如:【验证码：{1}，描述：{2}】，则传递参数 [888,666]按顺序去格式化模板
    :return:
    """
    appid = 112
    appkey = "8cc5b87123y4"
    sms_sign = "Python"
    sender = SmsMultiSender(appid, appkey)
    try:
        response = sender.send_with_param(86, phone_num_list, template_id, param_list, sign=sms_sign)
    except HTTPError as e:
        response = {'result': 1000, 'errmsg': "网络异常发送失败"}
    return response
if __name__ == '__main__':
    result1 = send_sms_single("15112345678", 548760, [666, ])
    print(result1)
    result2 = send_sms_single( ["15112345678", "15112345679", "15112345690", ],548760, [999, ])
    print(result2)
```

#### 3 频率限制

![image-20200609110906265](https://github.com/hanfuming/notebook/blob/master/assets/image-20200609110906265.png)



