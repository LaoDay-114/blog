---
title: LinkGo编译器
---
## 前言
关于LinkGo编译器的文章正式转移到 BitCat114 的博客了
LinkGo编译器真的开发了好久qwq
想要看更多原版LinkGo编译器的信息请观看官方网址：
[LinkGo编译器](https://linkgo.bitcat.dpdns.org) 或 [备用网址](https://linkgo.ashc.qzz.io)
推荐使用备用网址，网页正在从备用网址迁移到正式网址。
如果正式网址出现问题，请链接到备用网址。

另外LinkScript的设计也花了很久）
## 技术栈
使用 Rust 构建，编译独立语言 LinkScript
LinkGo编译代码过程中需要 nasm编译器 参与。

## 使用
抱歉，LinkGo编译器暂未提供预编译版
请从官方网站获取源代码自行编译
获取源代码后请安装 `ansi_term` 库（没错。只有这一个依赖）
之后执行 `cargo build` 编译就可以了

## 原理
LinkGo内置了 LinkScript 解释器
编译过程:
1. 用户使用命令参数向 LinkGo 指定 `.lg` LinkScript代码文件。
2. LinkGo 读取用户指定的代码文件。
3. LinkGo 将读取的代码内容交给 LinkGoparser 组件。
4. LinkGoparser 组件将处理好的生成逻辑交给 LinkGobuild 组件。
5. LinkGobuild 组件生成转换好的 64位 汇编代码。
6. LinkGobuild 组件调用 nasm编译器 编译生成好的64位汇编代码。
7. 完成编译，调用 LinkGolog 组件向用户发送提示
总计耗时 1 秒
> LinkScriptV1 极其简陋qwq
这时候可能就会有读者好奇了 `老猫老猫，你说的这个LinkScriptv1到底长啥样啊？`
emmm
那我就给你们示例吧: (没有展示测试API以及不稳定API)
```LinkScriptV1
use LinkScriptStd
use LinkScriptString
use LinkScriptMath

fnc _Main() {
    @这是一行注释
    var greeting -> int = 20;
    Prints("Hello, World! {}", greeting);

    var str_a, str_b -> int = 20;
    var str_result -> &str;
    LinkString(str_a, str_b, str_result);
    Prints("{}", str_result);

    var add_a, add_b -> int = 20;
    var add_result -> int;
    AddInt(add_a, add_b, add_result);
    Prints("{}", add_result);

    ret LinkScriptDone;
}
```
