title: 安装vim插件YouCompleteMe
date: 2016-03-06 17:59:39
tags: [Linux,vim,YouCompleteMe]
---
`YouCompleteMe`是`vim`上非常好用的代码自动补全的插件。但是必须自己本机编译他，提高了门槛。
我的环境是`ubuntu15.10`，照着教程来：
```shell
git submodule add https://github.com/Valloric/YouCompleteMe bundle/YouCompleteMe
git submodule update --init --recursive -- bundle/YouCompleteMe
```
然后编译：
```c
cd bundle/YouCompleteMe
./install.sh --clang-completer
```
但是我编译的时候出现了错误：
```
[ 87%] Building CXX object ycm/CMakeFiles/ycm_core.dir/ClangCompleter/ClangCompleter.cpp.o
In file included from /home/kai/.vim/bundle/YouCompleteMe/third_party/ycmd/cpp/BoostParts/boost/type_traits/ice.hpp:15:0,
                 from /home/kai/.vim/bundle/YouCompleteMe/third_party/ycmd/cpp/BoostParts/boost/python/detail/def_helper.hpp:9,
                 from /home/kai/.vim/bundle/YouCompleteMe/third_party/ycmd/cpp/BoostParts/boost/python/class.hpp:29,
                 from /home/kai/.vim/bundle/YouCompleteMe/third_party/ycmd/cpp/BoostParts/boost/python.hpp:18,
                 from /home/kai/.vim/bundle/YouCompleteMe/third_party/ycmd/cpp/ycm/ReleaseGil.h:21,
                 from /home/kai/.vim/bundle/YouCompleteMe/third_party/ycmd/cpp/ycm/ClangCompleter/ClangCompleter.cpp:28:
/home/kai/.vim/bundle/YouCompleteMe/third_party/ycmd/cpp/BoostParts/boost/type_traits/detail/ice_or.hpp:17:71: note: #pragma message: NOTE: Use of this header (ice_or.hpp) is deprecated
 # pragma message("NOTE: Use of this header (ice_or.hpp) is deprecated")
                                                                       ^
```

然后一直到最后。
这个是因为在编译`boost`时，出现了错误。

在这之前，脚本会自动下载`clang`，然后编译编译`boost`，而且编译出现了错误。这时我想为什么不用本机上已经安装好的的`clang`和`boost`呢？
```shell
sudo apt-get instll clang
sudo apt-get install libboost-all-dev
```

然后执行
```shell
./install.sh --clang-completer --system-libclang --system-boost
```

而这一切都在[官方文档里有说明](https://github.com/Valloric/YouCompleteMe#freebsdopenbsd)，我折腾完才看到，所以认真读说明是很重要的。 :(