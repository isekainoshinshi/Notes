# 在配置GVim过程中遇到的一些问题

感觉在Windows上配置gvim的环境相比ubuntu上配置要相对有一些难度，
我在配置过程中感觉每个步骤都遇到了或多或少的问题。
现在，我想通过这篇小小的笔记来说明一下在整个过程中所遇到的问题以及如何解决的。

## 需求说明

我搭建gVim的环境主要是为了写C/C++，
以及配置的差不多之后又有了写Markdown的需求。

## 插件配置

请阅读[韦易笑在知乎关于配置vim用于C/C++环境的回答](https://www.zhihu.com/question/47691414/answer/373700711)。

## 遇到的困难

在整个过程中我分别在一下几个步骤中遇到了问题，按我遇到的先后顺序排列。

- 启动YCM过程中
- 本地编译gVim过程中
- 使用YCM过程中

### YCM崩溃

`ycmd server SHUT DOWN (restart with ':YcmRestartServer')`

通过查阅资料发现可能需要本地进行一些编译，
所以解决办法是进到插件的目录执行下面的命令。

```bash
cd ~/.vim/bundle/youcompleteme
python install.py --clang-completer
```

我看非常多的解决方法调用的第二条指令为`python3`，
在我这里没有这么做的原因是：
当时输入`python3`时，
会弹出windos store，
而没有在命令行启动python3.6，
但是输入`python`反而能够正常启动。

进行完以上步骤，
之后进入gvim我本来期盼着就可以正常使用了，
结果就出现了下面一个问题。

参考阅读：

[When I enter VIM I get The ycmd server SHUT DOWN (restart with ':YcmRestartServer'). Unexpected error while loading the YCM core library. Type ':YcmToggleLogs ycmd_49739_stderr_rbhx3wqr.log' to check the logs ](https://github.com/ycm-core/YouCompleteMe/issues/3236)

["The ycmd server SHUT DOWN (restart with :YcmRestartServer)"](https://github.com/ycm-core/YouCompleteMe/issues/914)

---

### YCM安装完提示无法加载python36.dll

`YouCompleteMe unavailable: unable to load Python.`

出现这个问题首先需要检查你本地是否有安装python3.6或更新的版本。
如果安装了还出现了这个问题（也是我所遇到的），
那么一定要检查已经安装的gvim是32位的还是64位的，
还需要检查是否支持python3的特性，
这些都可以通过在gvim中执行`:version`来完成。

![version]( version.png )

![feature]( feature.png )

如果发现是32位的，但是你安装的python3.6是64位的，
那么你有两种方法可以正常使用python3特性：

- 安装32位python3
- 安装64位gVim

我选择的是第二种方式（当时查资料发现可能第一种方法更简单一些）。
本地编译gVim其实并不难，
只需要按照[vim/vim](https://github.com/vim/vim)上给出的流程来就迎刃而解了。
但是我在编译过程中还是遇到了一些问题。

---

### 无法找到Ruby的一些文件

`具体问题是编译过程中无法找到Ruby\Ruby27-x64\include\ruby-2.7.0\x64-mswin64_140路径下的一些文件`

遇到这个问题时我重新安装了好几次，
但是仍然没有出现这个文件夹，
后来在阅读gvim源文件中给出的`installpc.txt`文件中提到的一句话，
大意是需要本地对Ruby源代码进行编译，
获得`x64-mswin64_140`命名的文件夹，
再将这个文件夹放到上述的目录中就可以了。
这个文件名跟使用的MSVC版本有关。

---

### YCM无法正常补全头文件和一些函数

到这个时候，
我惊喜的发现进入gvim不冒红字了，
而且能够有一些补全了，
但是后来在知乎看到看大佬韦易笑介绍这个插件使用的gif图，
我才发现我用的好像不是一个插件，
这时候才发现了这个问题。

参考阅读：

[韦易笑在知乎关于配置vim用于C/C++环境的回答](https://www.zhihu.com/question/47691414/answer/373700711),


这时候发现完成YCM的安装还不算结束，
还需要配置`.ycm_extra_config`文件才能正常的实现补全功能。
这个文件在`~\.vim\plugged\YouCompleteMe\third_party\ycmd`路径下就有，
具体如何配置才能实现补全功能，
总结一句话就是在该文件的flag变量中添加你电脑中库的目录。

- 对于补全工作目录下的头文件、函数等参考[该文](https://www.jb51.cc/c/114392.html)。
- 对于补全标准库，实现方法与上文相似，需要将`-I`改为`'-isystem'`。

当然即使是这一步我也出现了问题，
我在完成上面这个步骤后发现依然不管用，
发现问题出现在目录的摆放顺序上。

```
flags = [
'-Wall',
'-Wextra',
'-Werror',
'-Wno-long-long',
'-Wno-variadic-macros',
'-fexceptions',
'-DNDEBUG',
# You 100% do NOT need -DUSE_CLANG_COMPLETER and/or -DYCM_EXPORT in your flags;
# only the YCM source code needs it.
'-DUSE_CLANG_COMPLETER',
'-DYCM_EXPORT=',
# THIS IS IMPORTANT! Without the '-x' flag, Clang won't know which language to
# use when compiling headers. So it will guess. Badly. So C++ headers will be
# compiled as C headers. You don't want that so ALWAYS specify the '-x' flag.
# For a C project, you would set this to 'c' instead of 'c++'.
'-x',
'c++',
'-isystem',
'cpp/pybind11',
'-isystem',
'cpp/whereami',
'-isystem',
'cpp/BoostParts',
'-isystem',
get_python_inc(),
'-isystem',
'cpp/llvm/include',
'-isystem',
'cpp/llvm/tools/clang/include',
'-I',
'cpp/ycm',
'-I',
'cpp/ycm/ClangCompleter',
'-isystem',
'cpp/ycm/tests/gmock/googlemock/include',
'-isystem',
'cpp/ycm/tests/gmock/googletest/include',
'-isystem',
'cpp/ycm/benchmarks/benchmark/include',
'-std=c++17',
]
```

原始的`.ycm_extra_config`文件flags变量是这样，
需要将添加的系统目录放到`'-std=c++17'`前面，
而我放到了之后，
所以导致一直没有办法正常补全。

---

## 总结

在最后小小的自我反省一下，
整个配置gVim过程中花费了大量时间去解决这几个问题，
但我没有想着去直接看说明文档，
而是寄希望于有人遇到过并且将现成答案步骤放上来，
并且在半天没查到时，
我才反过头来去看那些让我去看说明文档的回答。

## 2020.01.07补充

### gutentags无法正常使用

具体的报错信息没有截图保存，
大概第一行如下，
同时还会报错大意是`没有找到ctags，已经自动把gutentags_enable置为0，如果想重新启动请把那个变量置为1`。

`error detected while processing bufread auto commands gutentags`

这个问题是由于在Windows下没有安装ctags导致的，
那么只需要到[exuberant ctags官网](http://ctags.sourceforge.net/)下载ctags，
并将解压后的目录添加到系统环境变量中即可。

### markdown-preview编辑公式时公式总会重新渲染导致预览页面乱跳

这个问题我一开始以为是由于游览器导致的，
我从firefox转到chrome都没有解决，
后来去看了同一作者的`markdown-preview.nvim`(原先是markdown-preview.vim),
发现这个插件也支持vim8.1以上的版本，
尝试了一下，
发现没有了这个问题。
