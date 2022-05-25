# Dormitory-electricity-inquiry
### 中山大学南方学院
### 使用云函数来实现对宿舍电量的自动查询

### URL：

http://nfu.zhihuianxin.net/electric



### 云函数代码：

```python
# -*- coding: utf-8 -*-
import requests
import datetime
import time
import json
import sys

# 企业微信信息，用于推送消息通知，以查看执行情况
qywx_info = {
    # 企业 ID
    'corid' : '',
    # 应用密钥
    'secret' : '',
    # 应用 ID
    'agentid' : 
}
# 微信推送
def push_wechat(txt):
    # get access_token
    token_url = 'https://qyapi.weixin.qq.com/cgi-bin/gettoken?corpid=%s&corpsecret=%s' % (qywx_info['corid'], qywx_info['secret'])
    r = requests.get(token_url).json()
    assert r['errcode'] == 0, r['errmsg']
    # push
    push_url = 'https://qyapi.weixin.qq.com/cgi-bin/message/send?access_token=' + r['access_token']
    jsondata = {
       "touser" : 'LiangYingJian',
       "msgtype" : "text",
       "agentid" : qywx_info['agentid'],
       "text" : {
           "content" : txt
       }
    }
    r = requests.post(push_url, json = jsondata).json()
    print('push succeed!' if r['errcode'] == 0 else 'push failed! ' + r['errmsg'])

# 主函数
def main():
    url = 'http://nfu.zhihuianxin.net/electric/getData/getReserveAM'
    params = {
        # 宿舍 ID，从最上面URL自行获取
        'roomId': 3135,
    }
    res = requests.get(url,params=params)
    data = json.loads(res.text)
    power = float(data['data']['remainPower'])
    if (power) < 15.00:
        push_txt = (datetime.datetime.utcnow() + datetime.timedelta(hours=8)).isoformat(sep=' ', timespec='seconds')
        # 推送消息，自行更改
        push_txt += "\n东31-428当前电量为【%s】"% (power)
        push_wechat(push_txt)
        print(power)
    else:
        print(power)

def main_handler(event, context):
    return main()
            
if __name__ == '__main__':
    main()

```



