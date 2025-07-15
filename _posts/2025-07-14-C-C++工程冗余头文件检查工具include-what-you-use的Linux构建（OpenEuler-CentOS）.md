---
title: 'C/C++ 工程冗余头文件检查工具 include-what-you-use 的 Linux 构建（OpenEuler/CentOS）'
date: 2025-07-14
permk: /blogs/2025/C-C++工程冗余头文件检查工具include-what-you-use的Linux构建
excerpt:
  Include-what-you-use (iwyu) 是 Google 推出，基于 Clang 的 C/C++ 工程冗余头文件检查工具。本文介绍在 Linux 上构建 iwyu 的过程，以 OpenEuler（属于 RHEL/CentOS 系）为例。 
tags:
  - C/C++
  - Linux
---

[Include-what-you-use](https://github.com/include-what-you-use/include-what-you-use) (iwyu) 是 Google 推出，基于 Clang 的 C/C++ 工程冗余头文件检查工具。

iwyu 默认认为符合如下规则的 #include 语句是合理的：

- 不应该包含不需要的头文件
- 不应该依赖于其他头文件的间接包含
- 尽可能使用前置声明替代 #include 语句（有争议，可以用 `--no_fwd_decls` 选项禁用）

依据上述准则，iwyu 可以自动分析 C/C++ 源代码中的头文件依赖关系，识别出冗余的 #include 语句，并给出修改建议，从而大大加快项目的编译速度、减少编译依赖。

本文介绍在 Linux 上构建 iwyu 的过程，以 OpenEuler（属于 RHEL/CentOS 系）为例。

## 安装 Clang

1. 首先，安装 Clang 及其依赖：
  ```shell
  sudo yum install clang llvm-devel clang-devel
  ```
2. 检查 Clang 版本，iwyu 需要与 Clang 版本匹配：
  ```shell
  clang --version
  ```
  例如，如果 Clang 版本是 17.0.6，那么 iwyu 应该选择 clang_17 分支。

## 安装 iwyu

1. 克隆 iwyu 源码仓库：
  ```shell
  git clone https://github.com/include-what-you-use/include-what-you-use.git
  cd include-what-you-use
  ```
2. 切换与 Clang 版本对应的 iwyu 版本分支：
  ```shell
  git checkout clang_17
  ```
3. 使用系统自带的 LLVM 开发包构建 iwyu：
  ```shell
  mkdir build && cd build
  cmake -G "Unix Makefiles" -DCMAKE_PREFIX_PATH=$(llvm-config --prefix) ..
  make -j$(nproc)
  ```
  此时会在 `build/bin` 目录下生成 `include-what-you-use` 可执行文件。
4. 使用软链接将可执行文件命名为 `iwyu`，方便使用：
  ```shell
  cd bin && ln -s include-what-you-use iwyu
  ```
5. 将 iwyu 所在目录添加到 PATH 环境变量中：
  ```shell
  echo 'export PATH=$HOME/include-what-you-use/build/bin:$PATH' >> ~/.zshrc
  source ~/.zshrc
  ```

## 在 CMake 中使用 iwyu

1. 构建 CMake 项目时，添加以下选项以启用 iwyu：
  ```shell
  cmake -B build \
  -DCMAKE_CXX_COMPILER=clang++ \
  -DCMAKE_EXPORT_COMPILE_COMMANDS=ON \
  -DCMAKE_CXX_INCLUDE_WHAT_YOU_USE="iwyu;-Xiwyu;--no_fwd_decls;-resource-dir=$(clang -print-resource-dir)"
  ```
  其中，`--no_fwd_decls` 用于禁用前向声明检查，`-resource-dir` 指定 Clang 的资源目录。
2. 编译项目的同时，iwyu 会自动分析头文件依赖并生成建议。将终端显示的内容同时写入日志 `/tmp/iwyu.log` 中。
  ```shell
  cmake --build build -j$(nproc) 2>&1 | tee /tmp/iwyu.log
  ```
3. 使用 include-what-you-use 项目根目录下的自动补丁脚本 `fix_includes.py` 分析日志中的修改建议，脚本会自动对源文件做出修改。
  ```shell
  python3 ~/include-what-you-use/fix_includes.py < /tmp/iwyu.log
  ```