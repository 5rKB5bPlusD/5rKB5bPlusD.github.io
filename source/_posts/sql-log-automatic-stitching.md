---
title: sql日志自动拼接
date: 2022-04-20 20:46:06
categories: uTools
tags:
---

uTools官网 http://www.u.tools/

![](https://api.onedrive.com/v1.0/shares/s!AmIICgBbPfPTjmiViXlRo261rW17/root/content)
<center style="font-size:14px;color:#C0C0C0;text-decoration:underline">快捷命令-sql日志自动拼接</center>
<br/>

```javascript
let input = `{{payload}}`
let sql = input.slice(input.indexOf('Preparing:') + 11, input.lastIndexOf('DEBUG') - 24)
let params = input.slice(input.indexOf('Parameters:') + 12)
if (sql == '' || params == '') { return }
let sqlArr = sql.split('?')
let paramArr = params.split(', ').map((item) => {
    return item.replace(/.(?<=\()(?!.*\().*/, '').trim()
})
let res = ''
for (let i = 0; i < sqlArr.length - 1; i++) {
    if (sqlArr[i].endsWith('limit ') || sqlArr[i].endsWith('LIMIT ')) {
        res = res + sqlArr[i] + paramArr[i]
    } else {
        res = res + sqlArr[i] + '\'' + paramArr[i] + '\''
    }
}
res += sqlArr[sqlArr.length - 1]
if (!res.endsWith(';')) {
    res += ';'
}
utools.copyText(res)
utools.showNotification('已复制结果')
```