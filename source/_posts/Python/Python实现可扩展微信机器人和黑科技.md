---
title: Python实现可扩展微信机器人和黑科技
toc: true
tags:
  - Python
abbrlink: 56855
date: 2019-05-24 15:30:23
categories:
description:
---

![](https://tva1.sinaimg.com/large/e3bf8736gy1g3d7qhjp6tj21900u0qv6.jpg)

<!-- more-->

### 介绍

&emsp;&emsp;为了方便和效率，这里我使用了一个在开源社区比较流行的框架`itchat`，缺少的库请自行安装，我的开发环境比较乱，就不导出`requirement.txt`了。我也有自己分析的一套登陆以及消息轮询的代码，但是我是懒癌晚期，不想维护和提供长期服务什么的。这里使用开源框架做示例也比较方便你自己去查文档。

&emsp;&emsp;我也看到网上很多人有很多种有趣的玩法，一些不是玩网一族的人看不出来这个能实现什么功能。它就像微信给的小程序一样，微信提供的只是接口，各种功能都是开发者想出来，我几年前做QQ机器人，也是用web端协议，当时集成的有一言接口，图灵接口，日历接口，文字游戏接口，图片回复接口，猜谜等第三方接口，还实现了后台牛牛系统(赌博系统)，信息管理系统，定时通知功能等。所以说如果不去开发它，就没什么意义了。

### 示例代码解析

```python
# coding=utf-8
import re
import traceback

import itchat, os, time, cv2
from itchat.content import *

# 说明：可以撤回的有文本文字、语音、视频、图片、位置、名片、分享、附件

# {msg_id:(msg_from,msg_to,msg_time,msg_time_rec,msg_type,msg_content,msg_share_url)}
msg_dict = {}

# 文件存储临时目录
rev_tmp_dir = r'C:\Users\Administrator\Desktop\ARMProjects\WeChat\\'
if not os.path.exists(rev_tmp_dir): os.mkdir(rev_tmp_dir)

# 表情有一个问题 | 接受信息和接受note的msg_id不一致 巧合解决方案
face_bug = None

sendMsg = u"{消息助手}：微信助手开启，主人正在休息，稍后回复。"
usageMsg = u"使用方法：\n1.运行CMD命令：cmd xxx (xxx为命令)\n" \
           u"-例如关机命令:\ncmd shutdown -s -t 0 \n" \
           u"2.获取当前电脑用户：cap\n3.启用消息助手(默认关闭)：ast\n" \
           u"4.关闭消息助手：astc"
flag = 0  # 消息助手开关
filename = "{}.txt".format(time.strftime('%Y-%m-%d', time.localtime(time.time())))


def log(text):
    with open(filename, 'a', encoding='utf-8') as f:
        f.seek(0)
        f.write('-' * 20 + '\n')
        f.write(text + '\n')
        f.write('-' * 20 + '\n')


@itchat.msg_register('Text', isGroupChat=False)
def text_reply(msg):
    global flag
    global face_bug
    message = msg['Text']
    fromName = msg['FromUserName']
    toName = msg['ToUserName']
    msg_from = (itchat.search_friends(userName=msg['FromUserName']))["NickName"]

    # 获取的是本地时间戳并格式化本地时间戳 e: 2017-04-21 21:30:08
    msg_time_rec = time.strftime("%Y-%m-%d %H:%M:%S", time.localtime(time.time()))
    # 消息ID
    msg_id = msg['MsgId']
    # 消息时间
    msg_time = msg['CreateTime']
    # 消息发送人昵称 | 这里也可以使用RemarkName备注　但是自己或者没有备注的人为None
    msg_from = (itchat.search_friends(userName=msg['FromUserName']))["NickName"]
    # 消息内容
    msg_content = msg['Text']
    # 分享的链接
    msg_share_url = None

    face_bug = msg_content
    # 更新字典
    msg_dict.update(
        {
            msg_id: {
                "msg_from": msg_from, "msg_time": msg_time, "msg_time_rec": msg_time_rec,
                "msg_type": msg["Type"],
                "msg_content": msg_content, "msg_share_url": msg_share_url
            }
        }
    )

    if '昌哥' in msg['Text'] or '阿昌' in msg['Text'] or 'DAOJI' in msg['Text'].upper():
        try:
            msg.user.send_image('./190427-130223.png')
            # itchat.send_image('./190427-130223.png', toName)
        except:
            traceback.print_exc()

    if toName == "filehelper":
        if message == "cap":
            cap = cv2.VideoCapture(0)
            ret, img = cap.read()
            cv2.imwrite("weixinTemp.jpg", img)
            itchat.send('@img@%s' % u'weixinTemp.jpg', 'filehelper')
            cap.release()
        if message[0:3] == "cmd":
            os.system(message.strip(message[0:4]))
        if message == "ast":
            flag = 1
            itchat.send("消息助手已开启", "filehelper")
        if message == "astc":
            flag = 0
            itchat.send("消息助手已关闭", "filehelper")
    elif flag == 1:
        itchat.send(sendMsg, fromName)
        log('{0}    {1}:\n\n{2}'.format(msg_time_rec, msg_from, message))


# TODO(Daoji) 2019/4/27 群聊艾特
@itchat.msg_register(TEXT, isGroupChat=True)
def text_reply(msg):
    if msg.isAt:
        msg.user.send(u'@%s\u2005I received: %s' % (
            msg.actualNickName, msg.text))


# 将接收到的消息存放在字典中，当接收到新消息时对字典中超时的消息进行清理 | 不接受不具有撤回功能的信息
# [TEXT, PICTURE, MAP, CARD, SHARING, RECORDING, ATTACHMENT, VIDEO, FRIENDS, NOTE]
@itchat.msg_register([PICTURE, MAP, CARD, SHARING, RECORDING, ATTACHMENT, VIDEO])
def handler_receive_msg(msg):
    global face_bug
    # 获取的是本地时间戳并格式化本地时间戳 e: 2017-04-21 21:30:08
    msg_time_rec = time.strftime("%Y-%m-%d %H:%M:%S", time.localtime())
    # 消息ID
    msg_id = msg['MsgId']
    # 消息时间
    msg_time = msg['CreateTime']
    # 消息发送人昵称 | 这里也可以使用RemarkName备注　但是自己或者没有备注的人为None
    msg_from = (itchat.search_friends(userName=msg['FromUserName']))["NickName"]
    # 消息内容
    msg_content = None
    # 分享的链接
    msg_share_url = None

    '''
            接收视频和图片和文件，自动保存以及回发
    '''
    # msg.download(msg.fileName)
    # typeSymbol = {
    #     PICTURE: 'img',
    #     VIDEO: 'vid', }.get(msg.type, 'fil')
    # return '@%s@%s' % (typeSymbol, msg.fileName)

    if msg['Type'] == 'Text' \
            or msg['Type'] == 'Friends':
        msg_content = msg['Text']
    elif msg['Type'] == 'Recording' \
            or msg['Type'] == 'Attachment' \
            or msg['Type'] == 'Video' \
            or msg['Type'] == 'Picture':
        msg_content = r"" + msg['FileName']
        # 保存文件
        msg['Text'](rev_tmp_dir + msg['FileName'])
    elif msg['Type'] == 'Card':
        msg_content = msg['RecommendInfo']['NickName'] + r" 的名片"
    elif msg['Type'] == 'Map':
        x, y, location = re.search(
            "<location x=\"(.*?)\" y=\"(.*?)\".*label=\"(.*?)\".*", msg['OriContent']).group(1, 2, 3)
        if location is None:
            msg_content = r"纬度->" + x.__str__() + " 经度->" + y.__str__()
        else:
            msg_content = r"" + location
    elif msg['Type'] == 'Sharing':
        msg_content = msg['Text']
        msg_share_url = msg['Url']
    face_bug = msg_content
    # 更新字典
    msg_dict.update(
        {
            msg_id: {
                "msg_from": msg_from, "msg_time": msg_time, "msg_time_rec": msg_time_rec,
                "msg_type": msg["Type"],
                "msg_content": msg_content, "msg_share_url": msg_share_url
            }
        }
    )


# 收到note通知类消息，判断是不是撤回并进行相应操作
@itchat.msg_register([NOTE])
def send_msg_helper(msg):
    global face_bug
    if re.search(r"<!\[CDATA\[.*撤回了一条消息\]\]>", msg['Content']) is not None:
        # 获取消息的id
        old_msg_id = re.search(r"<msgid>(.*?)</msgid>", msg['Content']).group(1)
        old_msg = msg_dict.get(old_msg_id, {})
        if len(old_msg_id) < 11:
            itchat.send_file(rev_tmp_dir + face_bug, toUserName='filehelper')
            os.remove(rev_tmp_dir + face_bug)
        else:
            msg_body = "告诉你一个秘密~" + "\n" \
                       + old_msg.get('msg_from') + " 撤回了 " + old_msg.get("msg_type") + " 消息" + "\n" \
                       + old_msg.get('msg_time_rec') + "\n" \
                       + "撤回了什么 ⇣" + "\n" \
                       + r"" + old_msg.get('msg_content')
            # 如果是分享存在链接
            if old_msg['msg_type'] == "Sharing": msg_body += "\n就是这个链接➣ " + old_msg.get('msg_share_url')
            # 将撤回消息发送到文件助手
            itchat.send(msg_body, toUserName='filehelper')
            # 有文件的话也要将文件发送回去
            if old_msg["msg_type"] == "Picture" \
                    or old_msg["msg_type"] == "Recording" \
                    or old_msg["msg_type"] == "Video" \
                    or old_msg["msg_type"] == "Attachment":
                file = '@fil@%s' % (rev_tmp_dir + old_msg['msg_content'])
                itchat.send(msg=file, toUserName='filehelper')
                os.remove(rev_tmp_dir + old_msg['msg_content'])
            # 删除字典旧消息
            msg_dict.pop(old_msg_id)


@itchat.msg_register(FRIENDS)
def add_friend(msg):
    msg.user.verify()
    msg.user.send('机器人启动，自动同意好友添加。')


if __name__ == '__main__':
    itchat.auto_login(hotReload=True, enableCmdQR=False)
    itchat.send(usageMsg, "filehelper")
    itchat.run()

```

&emsp;&emsp;`@itchat.msg_register('Text', isGroupChat=False)`这个是框架的语法，可以查文档，这个语法糖是绑定一个函数监听`Text`即文本消息，`isGroupChat=False`判断是不是群聊。这里不能重复绑定函数，不然有一个会失效。`@itchat.msg_register([PICTURE, MAP, CARD, SHARING, RECORDING, ATTACHMENT, VIDEO])` `@itchat.msg_register(FRIENDS)` `@itchat.msg_register([NOTE])`这个也是，写法不同，参照官方源码示例，好像文档没给出，但是影响、意义不大。

```python
	message = msg['Text']
    fromName = msg['FromUserName']
    toName = msg['ToUserName']
    msg_from = (itchat.search_friends(userName=msg['FromUserName']))["NickName"]
```



&emsp;&emsp;这里是绑定的函数接收一个`msg`形参，是一个字典，可以取出你接收到的一条消息里面包含的所有内容，有消息文本，发送者的微信ID，接受者的微信ID，以及发送者的昵称，还有更多的信息，要开发的话要去查官方资料了解一下，这里昵称通过框架的另外一个查找api查询微信ID对应的用户昵称。

```python
    if toName == "filehelper":
        if message == "cap":
            cap = cv2.VideoCapture(0)
            ret, img = cap.read()
            cv2.imwrite("weixinTemp.jpg", img)
            itchat.send('@img@%s' % u'weixinTemp.jpg', 'filehelper')
            cap.release()
        if message[0:3] == "cmd":
            os.system(message.strip(message[0:4]))
        if message == "ast":
            flag = 1
            itchat.send("消息助手已开启", "filehelper")
        if message == "astc":
            flag = 0
            itchat.send("消息助手已关闭", "filehelper")
    elif flag == 1:
        itchat.send(sendMsg, fromName)
        log('{0}    {1}:\n\n{2}'.format(msg_time_rec, msg_from, message))
```

&emsp;&emsp;我这里判断消息来自微信的文件传输助手，就进行逻辑操作，这里也调用了`OpenCV`进行拍照，可以远程传输到手机上，监控当前电脑前的用户相貌，还有`cmd`远程执行Power Shell命令，还有自动回复开关。

### 总结

&emsp;&emsp;还有群聊艾特取消息自动回复，消息防撤回，一些已注释的是不同的API写法，和我平时不使用的功能，这里我只是进行笼统的讲解和给出参考代码，因为也没有时间进行大篇幅的讲解，和讲述一些基础知识。有问题的通过我的网盘链接加群吧，基础知识会有人帮我讲解，他们解决不了的我会讲，代码就贴下面了，安装完模块就能进行测试了。

### 源码

```python
if __name__ == '__main__':
    itchat.auto_login(hotReload=True, enableCmdQR=False)
    itchat.send(usageMsg, "filehelper")
    itchat.run()
```

Linux端使用要把`enableCmdQR=False`改成`True`，基本用linux也不要我多说了，试试就知道。

地址：[源码](http://od.daoji.ml/)