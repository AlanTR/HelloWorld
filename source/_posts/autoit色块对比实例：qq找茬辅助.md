title: 【转】Autoit色块对比实例：QQ找茬辅助
date: 2013-02-25 09:00:44
tags: 
- Autoit
- 游戏
categories: 
- 笔记
- 转载
---

什么是Autoit？

{% blockquote %}
[Autoit][1]是一种自动控制工具。它可以被用来自动完成任何基于 Windows 或 DOS 的简单任务。它最初被设计用来自动完成安装那些其它方
法不能自动安装的软件。这在PC首次展示时非常有用，那时成百上千的机器需要被自动的安装。尽管有一些程序如 MS Office, Mcafee, IE4 等具有自动安装的组件，可还有太多的程序不具备自动安装的功能。那就是用到 AutoIt 的地方。AutoIt 也可以被用来在你的机器上完成简单
的任务。
{% endblockquote %}

是不是有点罗嗦了？那咱换个方式 - -
<!--more-->
{% blockquote %}
它可以用来创建Microsoft Windows中的工具软件来自动执行常见的任务，例如监视网站，监视网络，磁盘碎片整理以及备份。它还能用来模拟
用户的操作，因此在软件测试中可以使用AutoIt代替手动来“驱动”应用程序。它还常用来开发计算机游戏机器人，用来自动执行游戏中的任务。
{% endblockquote %}

其实有时候知道它能干嘛就够了，[Wiki][2]上的东西足够详细也足够精简。

昨晚上逛论坛的时候发现[komaau3][3]写的一份以色块对比的为基础的QQ美女找茬辅助，果断下载下来学习一下。原理大概就是截取游戏窗口(
窗口?用句柄激活呗)中的起始点坐标+图片大小确认的图片，将图片转换为数组，再依据正则表达式来进行对比。将结果输出在界面上，然后Dll
Call user32.dll 来实现将鼠标点击传到游戏窗口的目的。

小小改了一下，弄了一个QQ大家来找茬的版本，原作者的版本只能在QQ美女找茬里运行。

其实我接触Autoit也没多久，于是对原作者的运行方式是不是这么解读很怀疑 @_@

贴上代码备用：
{% codeblock QQ美女找茬 lang:au3 http://www.autoitx.com/thread-37444-1-1.html 原帖地址 %}
;#Include <GDIPlus.au3>
#include <ScreenCapture.au3>
;#Include <Array.au3>
#include <GUIConstantsEx.au3>
#include <WindowsConstants.au3>
 
$Form1 = GUICreate("QQ找茬助手 by aopo", 495, 445, -1, -1)
$Button1 = GUICtrlCreateButton("找", 0, 0, 25, 25)
$Pic1 = GUICtrlCreatePic("", 0, 0, 495, 445)
GUISetState(@SW_SHOW)
 
WinSetOnTop($Form1, "", 1)
 
_GDIPlus_Startup()
 
While 1
        $nMsg = GUIGetMsg()
        Switch $nMsg
                Case $GUI_EVENT_CLOSE
                        _GDIPlus_Shutdown()
                        Exit
                Case $Button1
                        GUISetState(@SW_HIDE, $Form1);由于采用的前台截图，所以先隐藏下窗口
                        $t = TimerInit()
                        getMap1()
                        WinSetTitle($Form1, "", "本次找茬耗时:" & Round(TimerDiff($t) / 1000, 2) & "秒")
                        GUISetState(@SW_SHOW, $Form1)
                Case $Pic1
                        Local $hWnd = WinGetHandle("大家来找茬")
                        Opt("MouseCoordMode", 2);相对客户区
                        Local $aPos = MouseGetPos();获取相对坐标
                        ConsoleWrite($aPos[0] & "," & $aPos[1] & @CRLF)
                        _MouseClick($hWnd, $aPos[0]+10, $aPos[1]+195);转换到找茬窗口的坐标
        EndSwitch
WEnd
 
Func getMap1()
        Local $hWnd = WinGetHandle("大家来找茬")
        If Not IsHWnd($hWnd) Then Return
        WinActivate($hWnd)
        $hBitmap = _ScreenCapture_CaptureWnd("", $hWnd, 0, 0, -1, -1, False)
        $hImage = _GDIPlus_BitmapCreateFromHBITMAP($hBitmap)
        ;_GDIPlus_ImageSaveToFile($hImage, @ScriptDir & "\原图.bmp")
        
        ;$hImage = _GDIPlus_ImageLoadFromFile(@ScriptDir & "\原图.bmp");使用本地图片测试
 
        $hClone1 = _GDIPlus_BitmapCloneArea($hImage, 10, 195, 495, 445)
        ;_GDIPlus_ImageSaveToFile($hClone1, @ScriptDir & "\111111.bmp")
        Local $aMap1 = Bitmap2Array($hClone1)
 
        $hClone2 = _GDIPlus_BitmapCloneArea($hImage, 519, 195, 495, 445)
        ;_GDIPlus_ImageSaveToFile($hClone2, @ScriptDir & "\222222.bmp")
        Local $aMap2 = Bitmap2Array($hClone2)
 
        ;GUICtrlSetImage($Pic1, @ScriptDir & "\111111.bmp")
        $hGraphic = _GDIPlus_ImageGetGraphicsContext($hClone1)
        $hPen = _GDIPlus_PenCreate(0xFFFF0000, 2)
        Local $aMatch1, $aMatch2, $iX, $iY, $iW, $iH
        $iH = UBound($aMap1)
        For $iY = 0 To $iH - 1 Step 5;找茬颜色对比无需太严格，每次5行像素对比一次
                If $aMap1[$iY] <> $aMap2[$iY] Then
                        $aMatch1 = StringRegExp($aMap1[$iY], "\w{6}", 3);大图处理正则效率明显提高了很多
                        If @error Then ContinueLoop
                        $aMatch2 = StringRegExp($aMap2[$iY], "\w{6}", 3)
                        If @error Then ContinueLoop
                        $iW = UBound($aMatch1)
                        For $iX = 0 To $iW - 1 Step 10;每行每10像素对比一次，提高对比速度
                                If $aMatch1[$iX] == $aMatch2[$iX] Then ContinueLoop
                                _GDIPlus_GraphicsDrawLine($hGraphic, $iX, $iY + 1, $iX + 5, $iY + 1, $hPen)
                        Next
                EndIf
        Next
        _GDIPlus_ImageSaveToFile($hClone1, @TempDir & "\111111.bmp")
        GUICtrlSetImage($Pic1, @TempDir & "\111111.bmp")
        _GDIPlus_PenDispose($hPen)
        _GDIPlus_GraphicsDispose($hGraphic)
        _GDIPlus_ImageDispose($hClone1)
        _GDIPlus_ImageDispose($hClone2)
        _GDIPlus_ImageDispose($hImage)
EndFunc   ;==>getMap1
 
;ShellExecute(@ScriptDir & "\111111.bmp")
;ShellExecute(@ScriptDir & "\222222.bmp")
 
Func Bitmap2Array($h_Bitmap, $b_Array2d = False)
        Local $t_BITMAPDATA, $p_Scan, $i_Stride, $t_PIXEDATA, $s_PIXEDATA
        $i_Width = _GDIPlus_ImageGetWidth($h_Bitmap)
        $i_Height = _GDIPlus_ImageGetHeight($h_Bitmap)
        $t_BITMAPDATA = _GDIPlus_BitmapLockBits($h_Bitmap, 0, 0, $i_Width, $i_Height, $GDIP_ILMREAD, $GDIP_PXF24RGB)
        
        $i_Stride = DllStructGetData($t_BITMAPDATA, "Stride");Stride - Offset, in bytes, between consecutive scan lines of the bitmap. If the stride is positive, the bitmap is top-down. If the stride is negative, the bitmap is bottom-up.
        $p_Scan = DllStructGetData($t_BITMAPDATA, "Scan0");Scan0 - Pointer to the first (index 0) scan line of the bitmap.
        
        $t_PIXEDATA = DllStructCreate("ubyte lData[" & (Abs($i_Stride) * $i_Height) & "]", $p_Scan)
        
        $s_PIXEDATA = StringTrimLeft(DllStructGetData($t_PIXEDATA, "lData"), 2);去掉头部"0x"
        _GDIPlus_BitmapUnlockBits($h_Bitmap, $t_BITMAPDATA)
 
        If $b_Array2d Then;要求返回二维数组
                Local $a_Ret[$i_Height][$i_Width], $i_X, $i_Y, $s_Temp
                For $i_Y = 0 To $i_Height - 1
                        $s_Temp = StringMid($s_PIXEDATA, $i_Y * ($i_Stride * 2) + 1, $i_Width * 6)
                        $a_Match = StringRegExp($s_Temp, "\w{6}", 3)
                        For $i_X = 0 To $i_Width - 1
                                $a_Ret[$i_Y][$i_X] = Number("0x" & $a_Match[$i_X])
                        Next
                Next
        Else;要求返回一维数组
                Local $a_Ret[$i_Height], $i_Y
                For $i_Y = 0 To $i_Height - 1
                        $a_Ret[$i_Y] = StringMid($s_PIXEDATA, $i_Y * ($i_Stride * 2) + 1, $i_Width * 6)
                Next
        EndIf
        Return $a_Ret
EndFunc   ;==>Bitmap2Array
 
Func _MouseClick($hWnd, $x, $y, $button = 'left', $times = 1, $delay = 250)
        Local $iX
        Local $lParam = BitOR(BitAND($x, 0xFFFF), $y * 0x10000)
        $button = StringLower($button)
        If $button = "left" Then
                For $iX = 1 To $times
                        _PostMessage($hWnd, 0x200, 0, $lParam);WM_MOUSEMOVE
                        _PostMessage($hWnd, 0x201, 1, $lParam);WM_LBUTTONDOWN
                        _PostMessage($hWnd, 0x202, 0, $lParam);WM_LBUTTONUP
                        If $iX < $times Then Sleep($delay)
                Next
        ElseIf $button = "right" Then
                For $iX = 1 To $times
                        _PostMessage($hWnd, 0x200, 0, $lParam);WM_MOUSEMOVE
                        _PostMessage($hWnd, 0x204, 2, $lParam);WM_RBUTTONDOWN
                        _PostMessage($hWnd, 0x205, 0, $lParam);WM_RBUTTONUP
                        If $iX < $times Then Sleep($delay)
                Next
        EndIf
EndFunc   ;==>_MouseClick
 
Func _PostMessage($hWnd, $iMsg, $iwParam, $ilParam)
        DllCall("user32.dll", "bool", "PostMessage", "hwnd", $hWnd, "uint", $iMsg, "wparam", $iwParam, "lparam", $ilParam)
EndFunc   ;==>_PostMessage

{% endcodeblock %}



[1]: http://www.autoitscript.com/site/autoit/
[2]: http://zh.wikipedia.org/zh/AutoIt#.E7.94.A8.E6.B3.95
[3]: http://www.autoitx.com/space.php?uid=7653399
