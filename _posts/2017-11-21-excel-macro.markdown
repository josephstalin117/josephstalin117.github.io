---
layout: post
title:  "excel macro"
date:   2017-11-21 20:35:35 +0800
categories: vba
---


#### 1. 简单的跨sheet复制

```
Sub Macro()

    Dim x As Long
    Dim m As Long
    Dim i As Long
    i = 1
    For m = 3 To 33 Step 1
        For x = 2006 To 2015 Step 1
            Sheets(CStr(x)).Select
            Range("A" & m & ":AP" & m).Select
            Selection.Copy
            Sheets("Sheet1").Select
            Range("A" & i).Select
            ActiveSheet.Paste
            i = i + 1
        Next
    Next
    
    
End Sub

```
