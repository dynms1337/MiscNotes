
# すべてのシートをCSV出力するエクセルマクロ（UTF-8対応）

- 以下のエクセルマクロを実行すると、実行したエクセルファイルに含まれるすべてのシートを、UTF-8でCSV出力します。
- Shift-JISで出力したいときは、`.Charset = "UTF-8"`の箇所を`.Charset = "Shift-JIS"`に変更すればよいです。

```
Sub writeCSV_UTF8_noBOM_CRLF()
'
'エクセルファイルと同じフォルダに全シートをBOMなしUTF-8のCSVとして保存
'

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
    i = 1
    With adoSt
        .Charset = "UTF-8"
        .LineSeparator = adLF
        .Open

        Do While ws.Cells(i, 1).Text <> ""
            strLine = ""
            j = 1
            Do While True
                If ws.Cells(i, j + 1).Text = "" Then Exit Do
                strLine = strLine & ws.Cells(i, j).Text & ","
                j = j + 1
            Loop
            strLine = strLine & ws.Cells(i, j).Text & vbCrLf  ' 改行文字：\r\n
            .WriteText strLine
            i = i + 1
        Loop

        .Position = 0 'ストリームの位置を0にする
        .Type = adTypeBinary 'データの種類をバイナリデータに変更
        .Position = 3 'ストリームの位置を3にする

        Dim byteData() As Byte '一時格納用
        byteData = .Read 'ストリームの内容を一時格納用変数に保存
        .Close '一旦ストリームを閉じる（リセット）

        .Open 'ストリームを開く
        .Write byteData 'ストリームに一時格納したデータを流し込む
        .SaveToFile savePath, adSaveCreateOverWrite  'savePath:フルパス　adSaveCreateOverWrite:指定したファイルが既に存在する場合、上書き
        .Close
    End With

Next

End Sub
```

