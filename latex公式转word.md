```vba
Sub ConvertAllInlineLatexEquations()
    Dim rng As Range, endRng As Range
    Dim formulaText As String
    Dim userResponse As VbMsgBoxResult
    
    ' ==========================================
    ' 第一阶段：扫描并处理 $ ... $ 格式的行内公式
    ' ==========================================
    Set rng = ActiveDocument.Range(0, 0)
    rng.Find.ClearFormatting
    
    With rng.Find
        .Text = "$[!$]{1,}$"
        .MatchWildcards = True
        .Wrap = wdFindStop
        .Forward = True
        
        Do While .Execute
            rng.Select
            formulaText = rng.Text
            
            userResponse = MsgBox("发现公式 ($ 格式): " & vbCrLf & formulaText & vbCrLf & vbCrLf & _
                                  "是否将其转换为 Word 公式？", vbYesNoCancel + vbQuestion, "手动确认转换")
            
            If userResponse = vbCancel Then Exit Sub
            
            If userResponse = vbYes Then
                ' 去除首尾的 1 个字符 ($)
                rng.Text = Mid(formulaText, 2, Len(formulaText) - 2)
                ActiveDocument.OMaths.Add Range:=rng
                On Error Resume Next
                rng.OMaths(1).BuildUp
                On Error GoTo 0
            End If
            ' 无论选是或否，都将光标移至当前公式末尾继续
            rng.Collapse Direction:=wdCollapseEnd
        Loop
    End With
    
    ' ==========================================
    ' 第二阶段：扫描并处理 \( ... \) 格式的行内公式
    ' ==========================================
    Set rng = ActiveDocument.Range(0, 0)
    rng.Find.ClearFormatting
    
    With rng.Find
        .Text = "\("
        .MatchWildcards = False ' 关闭通配符，进行精准文本匹配
        .Wrap = wdFindStop
        .Forward = True
        
        Do While .Execute
            ' 此时 rng 只选中了 "\("，我们需要向后寻找对应的 "\)"
            Set endRng = ActiveDocument.Range(rng.End, ActiveDocument.Content.End)
            
            With endRng.Find
                .ClearFormatting
                .Text = "\)"
                .MatchWildcards = False
                .Wrap = wdFindStop
                .Forward = True
                
                If .Execute Then
                    ' 找到了闭合的 "\)"，将 rng 扩展为包含整个公式
                    rng.End = endRng.End
                    rng.Select
                    formulaText = rng.Text
                    
                    userResponse = MsgBox("发现公式 (\( \) 格式): " & vbCrLf & formulaText & vbCrLf & vbCrLf & _
                                          "是否将其转换为 Word 公式？", vbYesNoCancel + vbQuestion, "手动确认转换")
                    
                    If userResponse = vbCancel Then Exit Sub
                    
                    If userResponse = vbYes Then
                        ' 去除首尾的 2 个字符 (\( 和 \))
                        rng.Text = Mid(formulaText, 3, Len(formulaText) - 4)
                        ActiveDocument.OMaths.Add Range:=rng
                        On Error Resume Next
                        rng.OMaths(1).BuildUp
                        On Error GoTo 0
                    End If
                    ' 移动光标继续查找
                    rng.Collapse Direction:=wdCollapseEnd
                Else
                    ' 如果找不到对应的 "\)"，说明文档可能存在未闭合的语法，跳出此循环
                    Exit Do
                End If
            End With
        Loop
    End With
    
    MsgBox "文档中所有的 $ 和 \( \) 行内公式扫描与转换已完成！", vbInformation, "转换完毕"
End Sub
```

```
Sub ConvertAllInlineLatexEquations()
    Dim rng As Range, endRng As Range
    Dim formulaText As String
    Dim userResponse As VbMsgBoxResult
    
    ' ==========================================
    ' 第一阶段：扫描并处理 $ ... $ 格式的行内公式
    ' ==========================================
    Set rng = ActiveDocument.Range(0, 0)
    rng.Find.ClearFormatting
    
    With rng.Find
        .Text = "$[!$]{1,}$"
        .MatchWildcards = True
        .Wrap = wdFindStop
        .Forward = True
        
        Do While .Execute
            rng.Select
            formulaText = rng.Text
            
            userResponse = MsgBox("发现公式 ($ 格式): " & vbCrLf & formulaText & vbCrLf & vbCrLf & _
                                  "是否将其转换为 Word 公式？", vbYesNoCancel + vbQuestion, "手动确认转换")
            
            If userResponse = vbCancel Then Exit Sub
            
            If userResponse = vbYes Then
                ' 去除首尾的 1 个字符 ($)
                rng.Text = Mid(formulaText, 2, Len(formulaText) - 2)
                ActiveDocument.OMaths.Add Range:=rng
                On Error Resume Next
                rng.OMaths(1).BuildUp
                On Error GoTo 0
            End If
            ' 无论选是或否，都将光标移至当前公式末尾继续
            rng.Collapse Direction:=wdCollapseEnd
        Loop
    End With
    
    ' ==========================================
    ' 第二阶段：扫描并处理 \( ... \) 格式的行内公式
    ' ==========================================
    Set rng = ActiveDocument.Range(0, 0)
    rng.Find.ClearFormatting
    
    With rng.Find
        .Text = "\("
        .MatchWildcards = False ' 关闭通配符，进行精准文本匹配
        .Wrap = wdFindStop
        .Forward = True
        
        Do While .Execute
            ' 此时 rng 只选中了 "\("，我们需要向后寻找对应的 "\)"
            Set endRng = ActiveDocument.Range(rng.End, ActiveDocument.Content.End)
            
            With endRng.Find
                .ClearFormatting
                .Text = "\)"
                .MatchWildcards = False
                .Wrap = wdFindStop
                .Forward = True
                
                If .Execute Then
                    ' 找到了闭合的 "\)"，将 rng 扩展为包含整个公式
                    rng.End = endRng.End
                    rng.Select
                    formulaText = rng.Text
                    
                    userResponse = MsgBox("发现公式 (\( \) 格式): " & vbCrLf & formulaText & vbCrLf & vbCrLf & _
                                          "是否将其转换为 Word 公式？", vbYesNoCancel + vbQuestion, "手动确认转换")
                    
                    If userResponse = vbCancel Then Exit Sub
                    
                    If userResponse = vbYes Then
                        ' 去除首尾的 2 个字符 (\( 和 \))
                        rng.Text = Mid(formulaText, 3, Len(formulaText) - 4)
                        ActiveDocument.OMaths.Add Range:=rng
                        On Error Resume Next
                        rng.OMaths(1).BuildUp
                        On Error GoTo 0
                    End If
                    ' 移动光标继续查找
                    rng.Collapse Direction:=wdCollapseEnd
                Else
                    ' 如果找不到对应的 "\)"，说明文档可能存在未闭合的语法，跳出此循环
                    Exit Do
                End If
            End With
        Loop
    End With
    
    MsgBox "文档中所有的 $ 和 \( \) 行内公式扫描与转换已完成！", vbInformation, "转换完毕"
End Sub
```


```
Sub ConvertAllInlineLatexEquations()
    Dim rng As Range, endRng As Range
    Dim formulaText As String
    Dim userResponse As VbMsgBoxResult
    Dim autoProcess As Boolean
    
    autoProcess = False
    
    ' 询问用户是要逐个确认还是全部自动转换
    userResponse = MsgBox("是否开启【自动全部转换】模式？" & vbCrLf & _
                          "点击“是”：自动转换所有公式并应用粗体。" & vbCrLf & _
                          "点击“否”：逐个手动确认。", vbYesNoCancel + vbQuestion, "选择运行模式")
    
    If userResponse = vbCancel Then Exit Sub
    If userResponse = vbYes Then autoProcess = True
    
    ' ==========================================
    ' 第一阶段：扫描并处理 $ ... $ 格式
    ' ==========================================
    Set rng = ActiveDocument.Range(0, 0)
    rng.Find.ClearFormatting
    
    With rng.Find
        .Text = "$[!$]{1,}$"
        .MatchWildcards = True
        .Wrap = wdFindStop
        .Forward = True
        
        Do While .Execute
            If Not autoProcess Then
                rng.Select
                userResponse = MsgBox("发现公式: " & rng.Text & vbCrLf & "是否转换？", vbYesNoCancel + vbQuestion)
                If userResponse = vbCancel Then Exit Sub
                If userResponse = vbNo Then GoTo NextIter1
            End If
            
            ' 处理逻辑
            formulaText = rng.Text
            rng.Text = Mid(formulaText, 2, Len(formulaText) - 2)
            ProcessLatexBold rng ' 调用下方定义的粗体处理函数
            ActiveDocument.OMaths.Add Range:=rng
            On Error Resume Next
            rng.OMaths(1).BuildUp
            On Error GoTo 0
            
NextIter1:
            rng.Collapse Direction:=wdCollapseEnd
        Loop
    End With
    
    ' ==========================================
    ' 第二阶段：扫描并处理 \( ... \) 格式
    ' ==========================================
    Set rng = ActiveDocument.Range(0, 0)
    rng.Find.ClearFormatting
    
    With rng.Find
        .Text = "\("
        .MatchWildcards = False
        .Wrap = wdFindStop
        .Forward = True
        
        Do While .Execute
            Set endRng = ActiveDocument.Range(rng.End, ActiveDocument.Content.End)
            With endRng.Find
                .Text = "\)"
                .MatchWildcards = False
                .Wrap = wdFindStop
                If .Execute Then
                    rng.End = endRng.End
                    
                    If Not autoProcess Then
                        rng.Select
                        userResponse = MsgBox("发现公式: " & rng.Text & vbCrLf & "是否转换？", vbYesNoCancel + vbQuestion)
                        If userResponse = vbCancel Then Exit Sub
                        If userResponse = vbNo Then GoTo NextIter2
                    End If
                    
                    formulaText = rng.Text
                    rng.Text = Mid(formulaText, 3, Len(formulaText) - 4)
                    ProcessLatexBold rng
                    ActiveDocument.OMaths.Add Range:=rng
                    On Error Resume Next
                    rng.OMaths(1).BuildUp
                    On Error GoTo 0
                Else
                    Exit Do
                End If
            End With
NextIter2:
            rng.Collapse Direction:=wdCollapseEnd
        Loop
    End With
    
    MsgBox "所有公式处理完毕！", vbInformation
End Sub

' ==========================================
' 核心辅助函数：处理 \mathbf{...} 和 \bm{...}
' ==========================================
Sub ProcessLatexBold(ByRef targetRange As Range)
    Dim searchRng As Range
    Dim startPos As Long, endPos As Long
    
    ' 循环查找并替换所有的 \mathbf{...} 或 \bm{...}
    Do
        Set searchRng = targetRange.Duplicate
        With searchRng.Find
            .ClearFormatting
            .Text = "\\(mathbf|bm)\{(*)\}"
            .MatchWildcards = True
            .Forward = True
            If .Execute Then
                ' 确保找到的在当前处理的公式范围内
                If searchRng.Start >= targetRange.Start And searchRng.End <= targetRange.End Then
                    Dim content As String
                    content = searchRng.Text
                    startPos = InStr(content, "{")
                    endPos = InStrRev(content, "}")
                    
                    ' 提取中间内容并设为粗体
                    searchRng.Text = Mid(content, startPos + 1, endPos - startPos - 1)
                    searchRng.Font.Bold = True
                Else
                    Exit Do
                End If
            Else
                Exit Do
            End If
        End With
    Loop
End Sub
```