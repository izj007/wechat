#  wasm 初探，写个 Hello World

原创 西瓜  [ 前端西瓜哥 ](javascript:void\(0\);)

**前端西瓜哥** ![]()

微信号 fe_watermelon

功能介绍 专注前端图形编辑器领域，输出原创内容，欢迎关注。

____

___发表于_

收录于合集 #wasm 2个

大家好，我是前端西瓜哥。

我们来入门一下 wasm。

## wasm 是什么

wasm 是 WebAssembly 的缩写。

wasm 并不是传统意义上汇编语言（Assembly），而是一种中间编译的字节码，可以在浏览器上运行非 JavaScript 语言，只要它能被编译成
wasm。

wasm 的优点：

  1. 可以使用 C/C++、Rust等语言编写代码，这个是 wasm 最大的价值所在；
  2. 高效快速，二进制文件，以接近原生的速度运行；
  3. 安全，和 JS 有相同的沙盒环境和安全策略，比如同源策略；
  4. 绝大多数主流浏览器支持。另外可移植，非浏览器环境也能支持（塞个 v8 进去，比如 nodejs）；
  5. 使用其他语言的轮子。比如 Canvas 底层调用的 Skia C++ 库，就通过 wasm 技术提供了一个名为 CanvasKit 的 NPM 包给开发者用 JS 开发。

缺点：

  1. 适用场景较少，适合 CPU 密集型的场景（比如 3D 渲染）；
  2. 提升并没有非常高（几十倍），通常可能就两三倍的样子？但对普通前端来说学习成本太高，还得看投入产出比；
  3. 和 JS 有通信的成本，通信频繁或数据量大会降低性能。

## 安装

首先我们需要用到 Emscripten。Emscripten 是一个编译器工具链，使用 LLVM 去编译出 wasm。

先安装 Emscripten SDK。

我选择官网推荐的方式进行安装。西瓜哥我用的系统是 MacOS.

    
    
    # 拉取仓库  
    git clone https://github.com/emscripten-core/emsdk.git  
      
    # 进入目录  
    cd emsdk  
      
    # 下载最新 SDK 工具  
    ./emsdk install latest  
      
    # 版本设置为最新  
    ./emsdk activate latest  
      
    # 将相关命令行工具加入到 PATH 环境变量中（临时）  
    source ./emsdk_env.sh  
    

> 下载那里我一开始失败了几次，后来用了程序员都懂的那个东西才下载成功。

看看是不是成功安装了。

    
    
    emcc -v  
    

如果正确输出版本相关信息，就是安装成功了。

需要注意的是，每次打开新的终端，都要执行一下 `source ./emsdk_env.sh` 去临时更新 PATH 变量。

如果不想每次都要执行这玩意，可以在 .zshrc（或 .bashrc）中加上：

    
    
    # 需使用 emsdk_env.sh 文件的绝对路径  
    source /path/to/emsdk_env.sh &> /dev/null  
    

## Hello World

接下来我们要选择一门高级语言。

语言不能有 GC（自动垃圾回收机制）特性，比如 Java、Python。

（不过可以通过一些非官方的工具转成 wasm，就是问题比较多）

写 wasm，最流行的是 Rust 和 C/C++。

C/C++ 的轮子比较丰富，比如 Skia（Canvas 底层调用的库）就是 C++ 写的。可惜的是 C/C++ 没有包管理工具。

而当下最炙手可热的当属 Rust，我不得不说它真的很酷，有包管理工具，工具链也很完善。就是学习曲线过于陡峭，太难上手。

本文选择使用 C/C++ 语言。

先创建一个 `hello.c` 文件：

    
    
    #include <stdio.h>  
      
    int main() {  
      printf("Hello, world!\n");  
      return 0;  
    }  
    

运行下面命令编译成 wasm。

    
    
    emcc hello.c  
    

然后看到多了两个文件：`a.out.js` 和 `a.out.wasm`。

![]()

其中 js 文件是胶水代码，用来加载和执行 wasm 的，wasm 不能直接作为入口文件使用。

我们用 nodejs 运行一下 `a.out.js`，可以看到成功输出了 "Hello, world!"。

![]()

当然我们也可以创建一个 html 文件，引入这个 `a.out.js` 文件，也可以看到控制台能够正确输出输出。

![]()

看下资源请求，可以看到 html 引入了 `a.out.js`，然后 `a.out.js` 再引入 `a.out.wasm`。

![]()

## HTML 模板

为了方便大家调试，emscripten 还很贴心地提供了额外生成 index.html 的方式，并会引用上编译出来的 js 文件。

我们需要不上 `-o <文件名>.html` 指定输出的 html。

    
    
    emcc hello.cpp -o hello.html  
    

会生成 `hello.html`、`hello.js` 和 `hello.wasm` 三个文件。

打开 hello.html，我们可以看到一个界面，中间是一个 Canvas，显示 wasm 的渲染结果。下面则是控制台的输出。

![]()

## 文件系统

出于安全考虑，wasm 最终是要在浏览器的沙箱内运行的，是无法读取本地文件的。

但我们还是可以使用 C++ 的读取文件的方法的，只是它会被转换为从虚拟文件系统里读取。

`hello_world_file.cpp` 文件：

    
    
    #include <stdio.h>  
      
    int main() {  
      FILE *file = fopen("./hello_world_file.txt", "rb");  
      if (!file) {  
        printf("cannot open file\n");  
        return 1;  
      }  
      while (!feof(file)) {  
        char c = fgetc(file);  
        if (c != EOF) {  
          putchar(c);  
        }  
      }  
      fclose (file);  
      
      printf("\n");  
      
      return 0;  
    }  
    

需要读取的文本文件 `hello_world_file.txt` 为：

    
    
    ==  
    This data has been read from a file.  
    The file is readable as if it were at the same location in the filesystem, including directories, as in the local filesystem where you compiled the source.  
    前端西瓜哥  
    ==  
    

使用 `--preload-file`  选项，指定要预加载的资源文件。

    
    
    emcc hello_world_file.cpp -o hello.html --preload-file hello_world_file.txt  
    

结果：

![]()

## 代码优化

和编译 C 一样，为了提高开发时的编译效率，默认编译（默认为 `-O1`）出来的 wasm 是没有进行优化的。

下面命令会用优化等级 2 进行编译。

    
    
    emcc -O2 hello.c  
    

还有其他的优化等级，对应编译时间会变长，编译出来的文件尺寸会变大。

## 结尾

本文简单入门了一下 wasm。

wasm 是 JS 的补充，解决了 JS 的一些短板，不过总的来说大多数场景是用不上的，但它还在不断发展，我还是挺看好的。未来可期。

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

