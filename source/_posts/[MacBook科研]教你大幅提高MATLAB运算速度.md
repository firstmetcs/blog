---
title: 【MacBook科研】教你大幅提高MATLAB运算速度
excerpt: 苹果的M芯片内置了AMX单元，可以加速矩阵运算，提高科研效率。现在MATLAB也可以通过调用Apple Accelerate来使用AMX单元加速运算，只需改变MATLAB调用的BLAS库即可。
date: 2024-10-10 08:30:00
index_img: https://fsmt-blog.oss-cn-beijing.aliyuncs.com/cover/m-chip-macbook-matlab.webp
banner_img: http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/10/10/1040g008318m94sgpjq0g5nap35j0beajmv4erg8nddftwgthw.webp
tags:
- Matlab
- MacBook
- 科研
categories:
- Matlab
---
{% note info %}
本文由 Fluid 用户授权转载，版权归原作者所有。

本文作者：达达城城主
原文地址：http://xhslink.com/a/lURVCu1NuEpX
{% endnote %}

苹果的M芯片内置了AMX单元，可以加速矩阵运算，提高科研效率。现在MATLAB也可以通过调用Apple Accelerate来使用AMX单元加速运算，只需改变MATLAB调用的BLAS库即可。

# 结论
先说结论：我写了一个包含矩阵乘法、求解线性方程组、特征值分解、奇异值分解、FFT、矩阵求逆、Cholesky分解的测试程序。调用Accelerate运行测试程序可以加速50%以上，除了FFT运算速度略有下降外，其它运算的速度都有很大的提升。运行Matlab内置的Benchmark，前后分数差别不大。
所以，如果你在科研中常用的运算是矩阵操作，则可以尝试开启Accelerate加速。而如果你经常进行FFT运算，则不建议开启。
测试程序在文末附上

# 教程
下面开始教程：
1. 确认Apple Silicon的Accelerate框架库位置。我的电脑是M3 MacBook Air，macOS15.0，路径如下：
	`/System/Library/Frameworks/Accelerate.framework/Accelerate`
2. 设置Matlab启动项，使其调用苹果的BLAS库。
    1. 打开Matlab，确认当前使用的BLAS库。在命令行中输入：`version -blas` ，m芯片的Mac默认为OpenBLAS.
    2. 在命令行中输入：`edit setup.m`
    3. 在打开的setup.m文件中输入：
`setenv('BLAS_VERSION', '/System/Library/Frameworks/Accelerate.framework/Accelerate')`;
若你的电脑中Accelerate框架位置不同，则将代码中的路径替换为你的路径。
    4. 保存后重启Matlab
    5. 重启Matlab后，再次输入`version -blas`，若返回‘Apple Accelerate BLAS’则成功调用Accelerate。

![调用Apple Accelerate BLAS](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/10/10/1040g008318m94sgpjq0g5nap35j0beajmv4erg8nddftwgthw.webp)


![使用默认BLAS库](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/10/10/1040g008318m94sgpjq6g5nap35j0beajo93p7s8nddftwgthw.webp)
# 补充
下面是一些补充：
在写测试程序时，虽然计算的是随机矩阵，但随机数是用固定的种子生成的，所以无需担心可重复性。在同一台设备上重复运行测试程序，有10%的浮动是很正常的，建议在关闭其它应用后再进行测试。

![1040g008318m94sgpjq3g5nap35j0beaje2hsuhg!nd_dft_wgth_webp_3](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/10/10/1040g008318m94sgpjq3g5nap35j0beaje2hsuhgnddftwgthw.webp)

# 测试程序
本程序用于测试MATLAB矩阵运算性能，包含矩阵乘法、求解线性方程组、特征值分解、奇异值分解、FFT、矩阵求逆、Cholesky分解几个部分，并在运行后输出计算用时，用时越短代表性能越好。
写本程序是为了测试M芯片的Mac运行MATLAB时，调用AMX单元前后的性能对比。这个程序也可以用于测试电脑性能

![测试MATLAB矩阵运算性能的跑分程序](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/10/10/1040g008318mi2f7bjq1g5nap35j0beajvv1f9conddftwgthw.webp)
![测试MATLAB矩阵运算性能的跑分程序](http://fsmt-blog.oss-cn-beijing.aliyuncs.com/2024/10/10/1040g008318mi2f7bjq205nap35j0beaj6rei92onddftwgthw.webp)