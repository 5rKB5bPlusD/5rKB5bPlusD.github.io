---
title: 整理文件
date: 2022-04-20 20:45:19
categories: uTools
tags:
---

uTools官网 http://www.u.tools/

![](https://api.onedrive.com/v1.0/shares/s!AmIICgBbPfPTjmX8t0IZiK-Zl-uR/root/content)
<center style="font-size:14px;color:#C0C0C0;text-decoration:underline">快捷命令-整理文件</center>
<br/>

```javascript
/**
 * 文件整理工具
 * 文件移动，文件夹压缩移动，源文件/文件夹进入回收站
 * 需要安装7z，设置环境变量 https://www.7-zip.org/
 * 需要cmdutils支持回收文件 http://www.maddogsw.com/cmdutils/
 * 以上两个工具都要配置在环境变量中
 */

const logPath = path.join(os.tmpdir(), 'history.log')
const payload = {{payload}}
console.log(logPath)

//文件整理配置
let config = {
    '.*': 'D:/xxx/archive/杂项',
    '.*发票|行程.*': 'D:/xxx/archive/报销'
}

//在当前目录整理
// let current_dir = payload[0].path.slice(0, payload[0].path.lastIndexOf('\\'))
// let config = {
//     '.*': current_dir + '/杂项',
//     '.*发票|行程.*': current_dir + '/报销'
// }

// for (let i in config) {
//     if (!fs.existsSync(config[i])) {
//         fs.mkdirSync(config[i])
//     }
// }

//获取文件夹大小
const getSize = function(nowPath) {
    let size = 0
    const files = fs.readdirSync(nowPath)
    files.forEach(function(fileName, index) {
        console.log(fileName, index)
        const filePath = nowPath + '/' + fileName
        const file = fs.statSync(filePath)
        if (file.isDirectory()) {
            size += getSize(filePath)
        } else {
            size += file.size
        }
    })
    return size
}

//显示执行的操作
const show = function(showList) {
    quickcommand.showSelectList(showList, { optionType: 'json' }).then((choice) => {
        utools.copyText(JSON.stringify(choice))
        utools.shellOpenPath(logPath)
        show(showList)
    })
}

let showList = []
for (let i = 0, len = payload.length; i < len; i++) {
    let obj = {}
    obj['title'] = payload[i].name
    obj['description'] = payload[i].path + ' 正在执行中...'
    showList.push(obj)
}

show(showList)

let logMsg = ''
let destPaths = []
let recyclePaths = []
for (let i = 0, len = payload.length; i < len; i++) {
    let fileName = payload[i].name
    let filePath = payload[i].path
    // let fileCollationConf = JSON.parse(utools.dbStorage.getItem('file_collation_conf'))
    let fileCollationConf = config
    let destPath = ''
    for (let i in fileCollationConf) {
        if (fileName.match(i) != null && i != '.*') {
            destPath = fileCollationConf[i]
        }
    }
    if (destPath == null || destPath == '') {
        destPath = fileCollationConf['.*']
    }
    if (destPath == null || destPath == '') {
        let obj = {}
        obj['title'] = fileName
        obj['description'] = filePath + ' => 未匹配到规则'
        showList[i] = obj
        quickcommand.updateSelectList(obj, i)
        continue
    }

    if (payload[i].isDirectory) {
        destPath = path.join(destPath, fileName + '.7z')
        if (fs.existsSync(destPath)) {
            destPath = destPath.slice(0, destPath.lastIndexOf('\\') + 1) + fileName + '_' + new Date().getTime() + '.7z'
        }

        //计算预计时间
        let size = getSize(filePath)
        let expect = (size / 1024 / 1024 / 20 * 1000).toString().split('.')[0]
        let obj = {}
        obj['title'] = fileName
        obj['description'] = filePath + ' => 文件夹压缩中，预计耗时' + expect + 'ms'
        showList[i] = obj
        quickcommand.updateSelectList(obj, i)

        //需要安装7z并配置环境变量
        child_process.exec('7z a "' + destPath + '" "' + filePath + '"').on('close', (code) => {
            if (code == 0) {
                let obj = {}
                obj['title'] = fileName
                obj['description'] = filePath + ' => ' + destPath
                showList[i] = obj
                quickcommand.updateSelectList(obj, i)
                // utools.showNotification(filePath + ' => ' + destPath)
                //没有cmdutils工具注释掉下面代码
                child_process.exec('Recycle.exe "' + filePath + '"')
            } else {
                destPath = 'err code: ' + code
            }
        })
    } else {
        destPath = path.join(destPath, fileName)
        if (fs.existsSync(destPath)) {
            if (fileName.lastIndexOf('.') > 0) {
                const onlyName = fileName.substring(0, fileName.lastIndexOf('.'))
                const suffix = fileName.substr(fileName.lastIndexOf('.'))
                destPath = destPath.slice(0, destPath.lastIndexOf('\\') + 1) + onlyName + '_' + new Date().getTime() + suffix
            } else {
                destPath = destPath.slice(0, destPath.lastIndexOf('\\') + 1) + fileName + '_' + new Date().getTime()
            }
        }
        fs.writeFileSync(destPath, fs.readFileSync(filePath))

        let obj = {}
        obj['title'] = fileName
        obj['description'] = filePath + ' => ' + destPath
        showList[i] = obj
        quickcommand.updateSelectList(obj, i)
        // utools.showNotification(filePath + ' => ' + destPath)
        //没有cmdutils工具注释掉下面代码
        child_process.exec('Recycle.exe "' + filePath + '"')
    }
    let date = new Date()
    logMsg += date.toLocaleDateString() + ' ' + date.toLocaleTimeString() + ' ' + filePath + '----->' + destPath + '\n'
}

if (fs.existsSync(logPath)) {
    fs.appendFileSync(logPath, logMsg)
} else {
    fs.writeFile(logPath, logMsg, (err) => {})
}
utools.showNotification('整理完成')
```