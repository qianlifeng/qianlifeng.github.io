title: python设置windows桌面壁纸
date: 2012-05-10 11:25
tags: [python]
---

每天一个新壁纸，天天好心情。

<!--more-->

```
# -*- coding: UTF-8 -*- 

from __future__ import unicode_literals
import Image
import datetime
import win32gui,win32con,win32api
import re
from HttpWrapper import SendRequest

StoreFolder = "c:\\dayImage"

def setWallpaperFromBMP(imagepath):
    k = win32api.RegOpenKeyEx(win32con.HKEY_CURRENT_USER,"Control Panel\\Desktop",0,win32con.KEY_SET_VALUE)
    win32api.RegSetValueEx(k, "WallpaperStyle", 0, win32con.REG_SZ, "2") #2拉伸适应桌面,0桌面居中
    win32api.RegSetValueEx(k, "TileWallpaper", 0, win32con.REG_SZ, "0")
    win32gui.SystemParametersInfo(win32con.SPI_SETDESKWALLPAPER,imagepath, 1+2)

def setWallPaper(imagePath):
    """
    Given a path to an image, convert it to bmp and set it as wallpaper
    """
    bmpImage = Image.open(imagePath)
    newPath = StoreFolder + '\\mywallpaper.bmp'
    bmpImage.save(newPath, "BMP")
    setWallpaperFromBMP(newPath)

def getPicture():
    url = "http://photography.nationalgeographic.com/photography/photo-of-the-day/"
    h = SendRequest(url)
    if h.GetSource():
        r = re.findall('<div class="download_link"><a href="(.*?)">Download',h.GetSource())
        if r:
            return SendRequest(r[0]).GetSource()
        else:
            print "解析图片地址出错，请检查正则表达式是否正确"
            return None


def setWallpaperOfToday():
    img = getPicture()
    if img:
        path = StoreFolder + "\\%s.jpg" % datetime.date.today()
        f = open(path,"wb")
        f.write(img)
        f.close()
        setWallPaper(path)

setWallpaperOfToday()
print 'Wallpaper set ok!'
```

其中的httpwrapper是我写的一个http访问的封装：
```
#!/usr/bin/env python 
# -*- coding: UTF-8 -*-
#-------------------------------------------------------------------------------
# Name: 对http访问的封装
#
# Author: qianlifeng
#
# Created: 10-02-2012
#-------------------------------------------------------------------------------

import base64
import urllib
import urllib2
import time
import re
import sys

class SendRequest:
  """
  网页请求增强类
  SendRequest('http://xxx.com',data=dict, type='POST', auth='base',user='xxx', password='xxx')
  """
  def __init__(self, url, data=None, method='GET', auth=None, user=None, password=None, cookie = None, **header):
    """
    url: 请求的url，不能为空
    date: 需要post的内容，必须是字典
    method: Get 或者 Post，默认为Get
    auth: 'base' 或者 'cookie'
    user: 用于base认证的用户名
    password: 用于base认证的密码
    cookie: 请求附带的cookie，一般用于登录后的认证
    其他头信息:
    e.g. referer='www.sina.com.cn'
    """

    self.url = url
    self.data = data
    self.method = method
    self.auth = auth
    self.user = user
    self.password = password
    self.cookie = cookie

    if 'referer' in header:
        self.referer = header[referer]
    else:
        self.referer = None

    if 'user-agent' in header:
        self.user_agent = header[user-agent]
    else:
## self.user_agent = 'Mozilla/5.0 (Windows NT 5.1; rv:8.0) Gecko/20100101 Firefox/8.0'
        self.user_agent = 'Mozilla/5.0 (iPhone; U; CPU iPhone OS 3_0 like Mac OS X; en-us) AppleWebKit/528.18 (KHTML, like Gecko) Version/4.0 Mobile/7A341 Safari/528.16'

    self.__SetupRequest()
    self.__SendRequest()

  def __SetupRequest(self):

    if self.url is None or self.url == '':
        raise 'url 不能为空!'

    #访问方式设置
    if self.method.lower() == 'post':
        self.Req = urllib2.Request(self.url, urllib.urlencode(self.data))

    elif self.method.lower() == 'get':
        if self.data == None:
            self.Req = urllib2.Request(self.url)
        else:
            self.Req = urllib2.Request(self.url + '?' + urllib.urlencode(self.data))

    #设置认证信息
    if self.auth == 'base':
        if self.user == None or self.password == None:
            raise 'The user or password was not given!'
        else:
            auth_info = base64.encodestring(self.user + ':' + self.password).replace('\n','')
            auth_info = 'Basic ' + auth_info
            self.Req.add_header("Authorization", auth_info)

    elif self.auth == 'cookie':
        if self.cookie == None:
            raise 'The cookie was not given!'
        else:
            self.Req.add_header("Cookie", self.cookie)


    if self.referer:
        self.Req.add_header('referer', self.referer)
    if self.user_agent:
        self.Req.add_header('user-agent', self.user_agent)


  def __SendRequest(self):

    try:
      self.Res = urllib2.urlopen(self.Req)
      self.source = self.Res.read()
      self.code = self.Res.getcode()
      self.head_dict = self.Res.info().dict
      self.Res.close()
    except:
      print "Error: HttpWrapper=>_SendRequest ", sys.exc_info()[1]


  def GetResponseCode(self):
    """
    得到服务器返回的状态码(200表示成功,404网页不存在)
    """
    return self.code

  def GetSource(self):
    """
    得到网页源代码，需要解码后在使用
    """
    if "source" in dir(self):
        return self.source
    return u''

  def GetHeaderInfo(self):
    """
    u'得到响应头信息'
    """
    return self.head_dict

  def GetCookie(self):
    """
    得到服务器返回的Cookie，一般用于登录后续操作
    """
    if 'set-cookie' in self.head_dict:
      return self.head_dict['set-cookie']
    else:
      return None

  def GetContentType(self):
    """
    得到返回类型
    """
    if 'content-type' in self.head_dict:
      return self.head_dict['content-type']
    else:
      return None

  def GetCharset(self):
    """
    尝试得到网页的编码
    如果得不到返回None
    """
    contentType = self.GetContentType()
    if contentType is not None:
        index = contentType.find("charset")
        if index > 0:
           return contentType[index+8:]
    return None

  def GetExpiresTime(self):
    """
    得到网页过期时间
    """
    if 'expires' in self.head_dict:
      return self.head_dict['expires']
    else:
      return None

  def GetServerName(self):
    """
    得到服务器名字
    """
    if 'server' in self.head_dict:
      return self.head_dict['server']
    else:
      return None

__all__ = [SendRequest,]

if __name__ == '__main__':
    b = SendRequest("http://www.baidu.com")
    print b.GetSource()
```
