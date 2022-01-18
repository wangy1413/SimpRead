> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [blog.csdn.net](https://blog.csdn.net/qq285744011/article/details/122468863?utm_medium=distribute.pc_feed_v2.none-task-blog-personrec_tag-3.pc_personrecdepth_1-utm_source=distribute.pc_feed_v2.none-task-blog-personrec_tag-3.pc_personrec)

效果图
---

![](https://img-blog.csdnimg.cn/0ef3c40a71824b89a6a897ec2ad4de0e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAUnVkb27mu6jmtbfmuJTmnZE=,size_18,color_FFFFFF,t_70,g_se,x_16)

步骤
--

1.  安装 electronjs 和打包神器 electron-packager  
    [Win10 安装 electronjs 和打包神器 electron-packager_Rudon 滨海渔村的博客 - CSDN 博客步骤 1. 安装 Node.jsWindows10 系统安装 npm_Rudon 滨海渔村的博客 - CSDN 博客 2. 安装 Git bashGit - Downloads3. 安装 electronjs 和其打包神器 electron-packagernpm -vnpm config set registry http://registry.npm.taobao.org/// 全局安装 electronnpm install electron -g// 全局安装打包神器 electron-package..![](https://g.csdnimg.cn/static/logo/favicon32.ico)https://rudon.blog.csdn.net/article/details/122467263](https://rudon.blog.csdn.net/article/details/122467263 "Win10安装electronjs和打包神器electron-packager_Rudon滨海渔村的博客-CSDN博客")
2.   安装 VScode  
    [Visual Studio Code - Code Editing. RedefinedVisual Studio Code is a code editor redefined and optimized for building and debugging modern web and cloud applications.  Visual Studio Code is free and available on your favorite platform - Linux, macOS, and Windows.![](https://code.visualstudio.com/favicon.ico)https://code.visualstudio.com/](https://code.visualstudio.com/ "Visual Studio Code - Code Editing. Redefined")
3.  安装 git bash 并在 D 盘创建文件夹 D:\Projects\note  
    mkdir -p /D/Projects/note/  
    cd /D/  
    cd Projects/note/
4.  初始化 electron 项目  
     npm init -y
5.  使用 VScode 打开项目目录  
    ![](https://img-blog.csdnimg.cn/a82dee032d594fd19252ced97b1f5755.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAUnVkb27mu6jmtbfmuJTmnZE=,size_20,color_FFFFFF,t_70,g_se,x_16)
6.  查看 electron、electron-packager 版本  
    electron -v  
    electron-packager --version
7.  修改 package.json 内容为
    
    ```
    {
      "name": "note",
      "productName": "Note",
      "author": "Rudon",
      "version": "1.0.0",
      "description": "Note",
      "main": "main.js",
      "scripts": {
        "start": "electron ."
      },
      "license": "ISC",
      "devDependencies": {
        "electron": "^16.0.7"
      },
      "dependencies": {
        "electron-packager": "^15.4.0",
        "jquery": "^3.4.1"
      }
    }
    ```
    
    注意：要根据自身的 [electron](https://so.csdn.net/so/search?q=electron&spm=1001.2101.3001.7020)、electron-packager 版本进行修改
    
8.   命令行 cd 到项目根目录，再安装各种包  
    cd /D/Projects/note/  
    npm install  
    根目录下会自动生成 / package-lock.json， /node_modules/
9.  使用 vscode 创建如下文件：  
    event.js
    
    ```
    const remote = require('electron').remote;
    const ipcRenderer = require('electron').ipcRenderer;
    const dialog = remote.dialog;
    const Menu = remote.Menu;
    const fs = require('fs');
    var $ = require('jquery');
    var os  = require('os');
    let platform = os.platform();
     
     
    let filePath = ''
    let fileDocument = document.getElementById('newText')
    let isSave = true
     
     
    // 用来解释主进程和渲染进程的实例
    // 流程：先在主进程中监听窗口的close事件，然后当发生点击时，将消息从主进程发送到渲染进程。渲染进程收到消息后执行某些操作后，将消息发回主进程，由主进程执行剩下的操作
    function eventQuit() {
        var options = {};
        // options.title = '确定退出吗？';
        options.message = '确定退出吗？';
        options.type = 'none';
        options.buttons = ['Yes', 'No'];
        dialog.showMessageBox(options, (response) => {
            console.log('当前被单击的按钮索引是' + response);
            if (response == 0) {
                ipcRenderer.send('reqaction', 'exit');
            }
        })
    }
     
    function setNew() {
        document.getElementById('newText').value = ''
        filePath = ''
        window.document.title = '无标题文档'
    }
     
    function askChoice(ask_type) {
        if(ask_type == 0) {
            setNew()
        } else if (ask_type == 1) {
            filePath = openFile()
            if (filePath) {
                readFile(filePath)
                document.title = returnFileName(filePath)
            }
        } else {
            ipcRenderer.send('reqaction', 'exit');
        }
    }
     
    // 询问是否保存当前文档 ask_type: 0 newfile 1 openfile 2 exit
    function askSave(ask_type) {
        if (isSave == false) {
            var options = {};
            options.title = '是否将当前文档保存?';
            options.message = '是否将当前文档保存?';
            options.type = 'none';
            options.buttons = ['Yes', 'No'];
            dialog.showMessageBox(options, (response) => {
                if (response == 0) {
                    if (filePath == '') {
                        // 没有被保存过的新文档，需要先打开保存对话框
                        filePath = openSaveDialog()
                        writeFile(filePath, fileDocument.value)
                        askChoice(ask_type)
                    } else {
                        // 已经被保存过的，存在路径，直接保存文件
                        writeFile(filePath, fileDocument.value)
                        askChoice(ask_type)
                    }
                } else{
                    askChoice(ask_type)
                }
            })
        }
    }
     
    // 获取文件名
    function returnFileName(filePath) {
        if (platform == 'linux' || platform == 'darwin') {
            var fileList = filePath.split('/')
        } else {
            var fileList = filePath.split('\\')
        }
        return fileList[fileList.length - 1]
    }
     
    // 写入文档
    function writeFile(filePath, fileData) {
        fs.writeFileSync(filePath, fileData);
        isSave = true
    }
     
    // 读取文档
    function readFile(filePath) {
        window.document.getElementById('newText').value = fs.readFileSync(filePath, 'utf8');
    }
     
    // 打开保存对话框并返回路径
    function openSaveDialog() {
        var options = {};
        options.title = '保存文件';
        options.buttonLabel = '保存';
        options.defaultPath = '.';
        options.nameFieldLabel = '保存文件';
        options.showsTagField = false;
        options.filters = [
            {name: '文本文件', extensions: ['txt','js','html','md']},
            {name: '所有文件', extensions: ['*']}
        ]
        // 保存成功返回一个路径，否则返回 undefined
        var path = dialog.showSaveDialog(options)
        return path == undefined ? false : path
      }
     
     
    // 打开打开对话框并返回路径
    function openFile(){
        var options = {};
        options.title = '打开文件';
        options.buttonLabel = '打开';
        options.message = '打开文件';
        options.defaultPath = '.';
        options.properties = ['openFile'];
        options.filters = [
            {name: '文本文件', extensions: ['txt','js','html','md']}
        ]
        // 打开成功返回一个数组，第一个元素为打开的路径,否则返回 undefined
        var path = dialog.showOpenDialog(options)
        return path == undefined ? false : path[0]
      }
     
    //监听与主进程的通信
    ipcRenderer.on('action', (event, arg) => {
        switch (arg) {
            case 'exiting':
                if (fileDocument.value == '' || fileDocument.value == null || isSave == true) {
                    eventQuit()
                } else {
                    askSave(2)
                }
                break;
            case 'newfile':
                if (fileDocument.value == '' || fileDocument.value == null || isSave == true) {
                    setNew()
                } else {
                    askSave(0)
                }
                break;
            case 'openfile':
                if (fileDocument.value == '' || fileDocument.value == null || isSave == true) {
                    filePath = openFile()
                    if (filePath) {
                        readFile(filePath)
                        document.title = returnFileName(filePath)
                    }
                } else {
                    askSave(1)
                }
                break;
            case 'savefile':
                if (!filePath) filePath = openSaveDialog()
                if (filePath) {
                    writeFile(filePath, fileDocument.value)
                    document.title = returnFileName(filePath)
                }
                break;
        }
    });
     
     
    // 当打开页面时就会执行 onload。当用户进入后及离开页面时，会触发 onload 和 onunload 事件。
    window.onload = function() {
        console.log('开始',platform)
        let newText = document.getElementById('newText')
        const contextMenuTemplate = [
            { label: '复制', role: 'copy' }, 
            { label: '剪切', role: 'cut' }, 
            { label: '粘贴', role: 'paste' },
            { label: '删除', role: 'delete' }
          ];
        const contextMenu = Menu.buildFromTemplate(contextMenuTemplate);
        newText.addEventListener('contextmenu',function(event) {
            event.preventDefault();
            contextMenu.popup(remote.getCurrentWindow());
        })
        
        document.getElementById('newText').onkeyup = function(event) {
            switch (event.keyCode) {
                case 9:
                    newText.value += '    '
                    break;
            }
        }
        document.getElementById('newText').oninput = function (event) {
            if (isSave) document.title += " *"
            isSave = false
        }
    }
     
    // 监听浏览器窗口变化时执行的函数
    window.onresize = function(){
        document.getElementsByTagName('body')[0].style.height = window.innerHeight+'px';
        document.getElementById('newText').focus();
    }
    ```
    
      
    index.html
    
    ```
    <!DOCTYPE html>
    <html>
    <head>
        <meta charset="UTF-8">
        <title>e-text</title>
        <link rel="stylesheet" href="static/css/font-awesome.min.css">
        <style type="text/css">
            body {
                margin: 0;
                padding: 0;
                width: 100%;
                overflow-x: hidden;
                overflow-y: hidden;
            }
            #newText {
                width: 100%;
                height: 100%;
                margin: 0;
                padding: 0;
                outline: none;
                border-width: 0;
                resize: none;
                font-size: 18px;
            }
            #willshow:hover {
                cursor: pointer;
            }
        </style>
    </head>
    <body>
        <textarea id="newText"></textarea>
        <script src="event.js"></script>
    </body>
    </html>
    ```
    
      
    main.js
    
    ```
    const { app, BrowserWindow, Menu, Tray, ipcMain } = require('electron')  // 引入electron
     
    // 保持对window对象的全局引用，如果不这么做的话，当JavaScript对象被
    // 垃圾回收的时候，window对象将会自动的关闭
    let win
    // 托盘应用需要的
    let tray;
    let contextMenu
     
    function createWindow () {
      win = new BrowserWindow({
        // 设置窗口大小
        width: 800,
        height: 500,
        webPreferences: {
          nodeIntegration: true
        }
      })
      console.log('system edition:', process.platform)  // darwin:表示macos；linux:表示linux；win32:表示Windows；
      // 使用模版创建菜单
      const template = [
        {
          label: '文件',
          submenu: [    
            {
              label: '新建文件',
              click:()=>{
                win.webContents.send('action', 'newfile');
              }
            },
            {
              type: 'separator'
            },
            {
              label: '打开文件',
              click:()=>{
                win.webContents.send('action', 'openfile');
              }
            },
            {
              type: 'separator'
            },
            {
              label: '保存文件',
              click:()=>{
                win.webContents.send('action', 'savefile');
              }
            },
            {
              type: 'separator'
            },
            {
              label: '关闭',
              accelerator: 'Ctrl+Q',      // 设置菜单快捷键
              click: ()=>{win.close()}
            }
          ]
        },
        {
          label: '编辑',
          submenu: [
            {
              label: '复制',
              role:'copy',
              click:()=>{win.webContents.copy()} // 在点击时执行复制命令
            },
            {
              label: '粘贴',
              role:'paste',
              click:()=>{win.webContents.paste()} // 在点击时执行粘贴命令
            },
            {
              label: '剪切',
              role:'cut',
              click:()=>{win.webContents.cut()}
            },
            {
              type:'separator'   // 设置菜单项分隔条
            },
            {
              label: '撤销',
              role:'undo',
              click:()=>{win.webContents.undo()} // 在点击时执行撤销命令
            },
            {
              label: '重做',
              role:'redo',
              click:()=>{win.webContents.redo()} // 在点击时执行重做命令
            }
          ]
        }
        ];
     
      const menu = Menu.buildFromTemplate(template);
      //  开始设置菜单
      Menu.setApplicationMenu(menu);
     
      // 加载index.html文件
      win.loadFile('index.html')
     
      // 打开调试工具
      // win.webContents.openDevTools()
      
     
      // 监听窗口关闭的事件，监听到时将一个消息发送给渲染进程
      win.on('close', (e) => {
        e.preventDefault();
        // 给渲染进程发消息
        win.webContents.send('action', 'exiting');
      });
     
      // 当 window 被关闭，这个事件会被触发。
      win.on('closed', () => {
        // 取消引用 window 对象，如果你的应用支持多窗口的话，
        // 通常会把多个 window 对象存放在一个数组里面，
        // 与此同时，你应该删除相应的元素。
        win = null
      })
    }
     
    // Electron 会在初始化后并准备
    // 创建浏览器窗口时，调用这个函数。
    // 部分 API 在 ready 事件触发后才能使用。
    app.on('ready', createWindow)
     
    // 当全部窗口关闭时退出。
    app.on('window-all-closed', () => {
      // 在 macOS 上，除非用户用 Cmd + Q 确定地退出，
      // 否则绝大部分应用及其菜单栏会保持激活。
      if (process.platform !== 'darwin') {
        app.quit()
      }
    })
     
    app.on('activate', () => {
      // 在macOS上，当单击dock图标并且没有其他窗口打开时，
      // 通常在应用程序中重新创建一个窗口。
      if (win === null) {
        createWindow()
      }
    })
     
    // 监听与渲染进程的通讯，监听来自渲染进程的消息，当监听到确定关闭时，将所有窗口退出
    ipcMain.on('reqaction', (event, arg) => {
      console.log('zhu jin cheng:', arg)
      switch(arg){
        case 'exit':
          app.exit()  // 退出所有窗口，注意这里使用 app.quit() 无效
          break;
      }
    });
    ```
    
10.  开始启动测试，命令行：  
    cd /D/Projects/note/  
    electron .  
    即可弹出自己写的记事本  
    ![](https://img-blog.csdnimg.cn/588fa774c28d44fda0e8ef5f408c9cda.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAUnVkb27mu6jmtbfmuJTmnZE=,size_20,color_FFFFFF,t_70,g_se,x_16)
11.  打包！开始编译为 exe 可执行文件，命令行：  
    cd /D/Projects/note/  
    electron-packager .
12.  你的绿色版软件已经打包存放在  
    D:\Projects\note\Note-win32-x64 （整个文件夹）

![](https://img-blog.csdnimg.cn/63185e6bf1804af884db7098bad9d684.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAUnVkb27mu6jmtbfmuJTmnZE=,size_20,color_FFFFFF,t_70,g_se,x_16)

 有 Bug。。。。 无法关闭！研究中。。。

Idea 转发自 [https://blog.csdn.net/haeasringnar/article/details/94852324](https://blog.csdn.net/haeasringnar/article/details/94852324 "https://blog.csdn.net/haeasringnar/article/details/94852324")