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

#### 2.判断

```
Sub Macro2()

'Dim lastrow As Long
Dim xWs As Worksheet

For Each xWs In ThisWorkbook.Worksheets

    Dim lastrow As Long
    If xWs.Name <> "Sheet1" Then
        Range("A3:AP3").Select
        Selection.Copy
        Sheets("Sheet1").Select
        'lastrow = ActiveSheet.Cells(Rows.Count, "A").End(xlUp).Row + 1
        ActiveSheet.Paste
        'ActiveSheet.Cells(lastrow, "A").Value = "test"
        'ActiveSheet.Cells(lastrow, "A").Select
   End If
Next xWs

End Sub
```
