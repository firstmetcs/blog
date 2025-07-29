---
title: C#设置软件开机自启动
excerpt: 通过C#操作注册表来实现软件开机自启动，并解决无法以管理员权限运行的问题
date: 2025-06-27 15:10:20
index_img: https://fsmt-blog.oss-cn-beijing.aliyuncs.com/cover/run_on_start_up.jpeg
banner_img: https://fsmt-blog.oss-cn-beijing.aliyuncs.com/cover/run_on_start_up.jpeg
tags:
- C#
- 注册表
- 开机启动
- winform
categories:
- C#
---
# 如何实现

通过C#操作注册表来实现，直接调用方法：
```csharp
private void AutoStart(bool isAuto = true)       
 {            
	 if (isAuto == true)            
	 {               
		RegistryKey local = Registry.CurrentUser;               
		RegistryKey run = local.CreateSubKey(@"SOFTWARE\Microsoft\Windows\CurrentVersion\Run");                
		run.SetValue("ProName", System.Windows.Forms.Application.ExecutablePath);                
		run.Close();                
		local.Close();            
	   }           
	else           
	{                
		RegistryKey local = Registry.CurrentUser;               
		RegistryKey run = local.CreateSubKey(@"SOFTWARE\Microsoft\Windows\CurrentVersion\Run");                
		run.DeleteValue("ProName", false);                
		run.Close();               
		local.Close();           
 	 }        
}
```

# 无法运行
我想让它开机运行，就把它写到了开机注册表中：
```text
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
```
结果不管是注销还是重启，程序都不会启动。

于是我又把它写到了用户表中：
```text
HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Run
```
好像还是不行。

在我印象中，将程序路径加入这两个注册表下，就可以了实现自启。

我用其它程序试了一下，是可以的.

## 问题原因
一番测试之后发现，居然是因为给程序加入了运行时“请求以管理员权限运行”。

把它取消后可以开机自启，但是程序的一些功能需要管理员权限才行，因此这个方法在这里行不通。

## 解决方法
写到这个路径下即可（64位系统才有）：
```text
HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Run
```
这个注册表路径下的程序，无论是否有“请求以管理员权限运行”功能，都可以正常自启。

# 拓展
## 问题背景：
某个c#程序需要已管理员权限启动，并且有开机启动的功能。

## 解决方案：
1. 已管理员权限启动参考[c#程序以管理员权限运行](https://www.cnblogs.com/MarcLiu/p/13877279.html)。

2. 开机启动，参考代码
    ```csharp
    public static void SetAutoRun(bool isAutoRun)
    {
        //设置是否自动启动
        if (isAutoRun)
        {
            string path = System.Windows.Forms.Application.ExecutablePath;
            Microsoft.Win32.RegistryKey rk2 = Microsoft.Win32.Registry.LocalMachine.CreateSubKey(@"Software\WOW6432Node\Microsoft\Windows\CurrentVersion\Run");
            rk2.SetValue("App", @"""" + path + @"""");
            rk2.Close();
        }
        else
        {
            Microsoft.Win32.RegistryKey rk2 = Microsoft.Win32.Registry.LocalMachine.CreateSubKey(@"Software\WOW6432Node\Microsoft\Windows\CurrentVersion\Run");
            rk2.DeleteValue("App", false);
            rk2.Close();
        }
    }
    ```

以上是最终解决方案，下面列出遇到的一些问题

1. 为什么使用注册表`HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Run`。

    答：开机启动常用的方法是添加注册表开机启动项，可用的注册表位置包括
    
    `HKEY_CURRENT_USER\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Run`
    
    `HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`
    
    `HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Run`
    
    `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run`
    
    其中带WOW6432Node节点的是表示32位程序的
    
    其中如果程序需要已管理员权限启动，则需要写入到`HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Run`，如果过不需要已管理员权限启动，则写入`HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\Run `

2. 其他开机启动方式
    参考博文[C# 将程序添加开机启动的三种方式](https://blog.csdn.net/arrowzz/article/details/69808761)


{% note info %}
本文参考自以下博文，版权归原作者所有。

原文作者：李同学L
原文地址：[C#设置软件开机自启动](https://blog.csdn.net/weixin_45053845/article/details/131101855)

原文作者：skyyx2002
原文地址：[程序无法通过开机注册表运行的解决方法](https://blog.csdn.net/s806903/article/details/128204897)

原文作者：六镇2012
原文地址：[c#程序已管理员权限启动、开机自动启动相关问题](https://www.cnblogs.com/MarcLiu/p/13885607.html)
{% endnote %}