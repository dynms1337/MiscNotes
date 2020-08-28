
# すべてのシートをCSV出力するエクセルマクロ（UTF-8対応）

- 以下のエクセルマクロを実行すると、実行したエクセルファイルに含まれるすべてのシートを、UTF-8でCSV出力します。
- Shift-JISで出力したいときは、`Encoding = "Shift-JIS"`の箇所を有効に変更すればよいです。
- 事前にライブラリの参照設定が必要です。
  - Visual Basic Editor のメニューから［ツール］→［参照設定］を選び，［参照可能なライブラリファイル］の中から "Microsoft ActiveX Data Objects x.x Library" にチェックを入れます。

```
Sub writeCSV_UTF8_noBOM_CRLF()
'
'エクセルファイルと同じフォルダに全シートをBOMなしUTF-8のCSVとして保存
'

Encoding = "UTF-8"
'Encoding = "Shift-JIS"

Dim ws As Worksheet, savePath As String
' ブックが保存済みでない場合は保存
If ActiveWorkbook.Saved = False Then
    If MsgBox("ブックがまだ保存されていません。保存しますか？", vbYesNo) = vbNo Then
        Exit Sub
    Else
        ActiveWorkbook.Save
    End If
End If
'全シート保存
Application.DisplayAlerts = False
Application.ScreenUpdating = False
For Each ws In ActiveWorkbook.Worksheets
    ws.Activate
    savePath = ActiveWorkbook.Path & "\" & ws.Name & ".csv"
    'ADODB.Streamオブジェクトを生成
    Dim adoSt As Object
    Set adoSt = CreateObject("ADODB.Stream")
    
    Dim strLine As String
    Dim i As Long, j As Long
    Dim content As String
    i = 1
    Do While ws.Cells(i, 1).Text <> ""
        strLine = ""
        j = 1
        Do While True
            If ws.Cells(i, j + 1).Text = "" Then Exit Do
            strLine = strLine & ws.Cells(i, j).Text & ","
            j = j + 1
        Loop
        strLine = strLine & ws.Cells(i, j).Text & vbCrLf  ' 改行文字：\r\n
        content = content & strLine
        i = i + 1
    Loop
    tmp = SaveText(savePath, content, Encoding)
Next
End Sub

Function SaveText(Filename, Text As String, Optional Encoding = "UTF-8") As Boolean
    Const adTypeBinary = 1
    Const adTypeText = 2
    Const adSaveCreateOverWrite = 2
    Dim Position
    Dim Charset As String
    Dim Bytes
    Position = 0
    Select Case UCase(Encoding)
        Case "UTF-8"
            Charset = "utf-8"
            Position = 3
        Case Else
            Charset = Encoding
    End Select
    
    On Error Resume Next
    
    With CreateObject("ADODB.Stream")
        .Type = adTypeText
        .Charset = Charset
        .Open
        .WriteText Text
        .Position = 0
        .Type = adTypeBinary
        .Position = Position
        Bytes = .Read
        .Close
    End With
    With CreateObject("ADODB.Stream")
        .Type = adTypeBinary
        .Open
        .Position = 0
        .Write Bytes
        .SaveToFile Filename, adSaveCreateOverWrite
        .Close
    End With
    
    If Err.Number = 0 Then
        SaveText = True
    Else
        SaveText = False
    End If
End Function
```

