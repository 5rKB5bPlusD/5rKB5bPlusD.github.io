---
title: get参数解析成json
date: 2022-04-20 20:45:43
categories: uTools
tags:
---

uTools官网 http://www.u.tools/

![](https://api.onedrive.com/v1.0/shares/s!AmIICgBbPfPTjmdJt05MEH3YBeWI/root/content)
<center style="font-size:14px;color:#C0C0C0;text-decoration:underline">快捷命令-get参数解析成json</center>
<br/>

```javascript
let input = quickcommand.enterData.payload
let obj = {}

if (input.indexOf('?') > 0) {
    let url = input.substring(0, input.indexOf('?'))
    obj['url'] = url
    let param = input.substr(input.indexOf('?') + 1)
    let arr = param.split('&')
    let tempObj = {}
    arr.forEach(function(value) {
        let tempArr = value.split('=')
        if (tempArr.length == 2) {
            tempObj[tempArr[0]] = tempArr[1]
        } else {
            tempObj[tempArr[0]] = ''
        }
    })
    obj['param'] = tempObj
} else {
    let arr = input.split('&')
    let tempObj = {}
    arr.forEach(function(value) {
        let tempArr = value.split('=')
        if (tempArr.length == 2) {
            tempObj[tempArr[0]] = tempArr[1]
        } else {
            tempObj[tempArr[0]] = ''
        }
    })
    obj['param'] = tempObj
}
utools.copyText(JSON.stringify(obj))
utools.redirect('json', JSON.stringify(obj))
```
