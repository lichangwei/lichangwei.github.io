---
title: 使用页面跳转完成登录和认证
---

在上篇文章我们提到如何使用WWW-Authenticate实现登录和认证。这一次我们讲一下另一种实现方式。服务器端提供一个登录认证页面，浏览器端需要跳转到该页面，用户输入PIN码之后，发送请求，浏览器验证之后跳转当前页面来。以下是更详细的步骤。  

1. 浏览器端点击认证按钮，然后请求电视机（服务器端，下同）提供的一个认证页面。
```
GET http://192.168.1.199:7999/webauth/auth_default?app_name=Sample%20Web%20App&app_url=http%3A%2F%2F192.168.1.199%3A3000%2Ftest%2Fbravia.html&return_url=http%3A%2F%2F192.168.1.199%3A3000%2Ftest%2Fbravia.html%3Fserver%3D192.168.1.199%26port%3D7999%26device_id%3D3%26x%3D1%26y%3D2%23zzz&auth_level=generic HTTP/1.1
```
该请求中包含一些重要的参数信息：  
app_name 该应用的名称，显示给用户确认。  
app_url 该应用的网址，显示给用户确认。  
return_url 认证完成之后，通过该网址跳回本应用。  
auth_level 认证等级，将会体现到该应用对电视机操控权限上。  
  
2. 电视机接收到认证页面请求，创建PIN码并显示在电视机上（如下图），返回该页面源码。  
![电视机上显示PIN码](../images/Redirect-1.png)  
```
HTTP/1.1 200 OK
<!DOCTYPE HTML>
```  
  
3. 浏览器端显示该页面（如下图），用户输入PIN码并发送认证请求。  
![浏览器上显示认证页面](../images/Redirect-2.png)  
```
POST http://192.168.1.199:7999/webauth/auth_default_submit HTTP/1.1
Content-Type: application/x-www-form-urlencoded
pin_code=5184&response=allow&token=1378462608431&app_name=Sample+Web+App&app_url=http%3A%2F%2F192.168.1.199%3A3000%2Ftest%2Fbravia.html&return_url=http%3A%2F%2F192.168.1.199%3A3000%2Ftest%2Fbravia.html%3Fserver%3D192.168.1.199%26port%3D7999%26device_id%3D3%26x%3D1%26y%3D2%23zzz&auth_level=generic
```  
  
4. 电视机收到认证请求，判断PIN码是否正确，如果不正确，则再次回到第二步中的认证页面，并提示错误，如下图。  
![PIN码输入错误](../images/Redirect-error-case.png)  
如果正确，则跳转到return_url页面。  
```
HTTP/1.1 302 Found
Location: http://192.168.1.199:3000/test/bravia.html?server=192.168.1.199&port=7999&device_id=3&x=1&y=2&result=authorized#zzz
Set-Cookie: auth=9336226396260589431055355817107579663142096251022247731126844883; path=/; max-age=1209600; expires=Fri, 20 Sep 2013 10:49:56 GMT
```

