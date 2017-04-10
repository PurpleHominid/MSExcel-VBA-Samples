# MSExcel-VBA-Samples
The follow snippets are for reference with regards to adding controls to worksheets and processing the updates via VBA

## Open a WWW page using a cell value as the URL
```
Sub ButtonURLOpen_Click()
    'ActiveWorkbook.FollowHyperlink Address:="http://google.co.uk", NewWindow:=True

    'This function gets the row and column number related to the position of the button
    Dim b As Object, RowNo As Integer, ColNo As Integer
    Dim sAddress As String 'Create a variable for aour address

    Set b = ActiveSheet.Buttons(Application.Caller) 'locate the row and column of the button
    With b.TopLeftCell
        ColNo = .Column
        RowNo = .Row
    End With
    Set b = Nothing

    'MsgBox "Column Number " & ColNo
    'MsgBox "Row Number " & RowNo


    sAddress = ActiveSheet.Cells(RowNo, ColNo - 1).Value
    If (URLExists(sAddress)) Then
        ActiveWorkbook.FollowHyperlink _
            address:=sAddress, _
            NewWindow:=True, _
            AddHistory:=False
        'update the related cell data with a colour indicator
        ActiveSheet.Cells(RowNo, ColNo - 1).Font.ColorIndex = 1
        ActiveSheet.Cells(RowNo, ColNo - 1).Interior.ColorIndex = 4

    Else
        'MsgBox (sAddress & " does not seem to exist")
        'update the related cell data with a colour indicator
        ActiveSheet.Cells(RowNo, ColNo - 1).Font.ColorIndex = 2
        ActiveSheet.Cells(RowNo, ColNo - 1).Interior.ColorIndex = 3
    End If

End Sub
```

## Return TRUE/FALSE is the URL provide exists (needs a little work, WinHttp.WinHttpRequest is not available on Mac MSExcel, plus the editor is junk!)
```
Function URLExists(sURL As String) As Boolean
    Dim Request As Object
    Dim sPostData As String

    Dim sResponse As Variant
    Dim bExists As Boolean

    Dim lTimeOutSec As Integer

    bExists = False
    lTimeOutSec = 2

    On Error GoTo EndNow
        Set Request = CreateObject("WinHttp.WinHttpRequest.5.1")

        'Set timeouts
        ' http://msdn.microsoft.com/en-us/library/windows/desktop/aa384061(v=vs.85).aspx
        ' SetTimeouts(resolveTimeout, ConnectTimeout, SendTimeout, ReceiveTimeout)
 
        'Open http verb (method), url, false to force syncronous
        ' open(http method, absolute uri to request, async (true: async, false: sync)

        'WinHttpRequestOption enumeration
        '(0) WinHttpRequestOption_UserAgentString
        '(1) WinHttpRequestOption_URL
        '(2) WinHttpRequestOption_URLCodePage
        '(3) WinHttpRequestOption_EscapePercentInURL
        '(4) WinHttpRequestOption_SslErrorIgnoreFlags
        '(5) WinHttpRequestOption_SelectCertificate
        '(6) WinHttpRequestOption_EnableRedirects
        '(7) WinHttpRequestOption_UrlEscapeDisable
        '(8) WinHttpRequestOption_UrlEscapeDisableQuery
        '(9) WinHttpRequestOption_SecureProtocols
        '(10) WinHttpRequestOption_EnableTracing
        '(11) WinHttpRequestOption_RevertImpersonationOverSsl
        '(12) WinHttpRequestOption_EnableHttpsToHttpRedirects
        '(13) WinHttpRequestOption_EnablePassportAuthentication
        '(14) WinHttpRequestOption_MaxAutomaticRedirects
        '(15) WinHttpRequestOption_MaxResponseHeaderSize
        '(16) WinHttpRequestOption_MaxResponseDrainSize
        '(17) WinHttpRequestOption_EnableHttp1_1
        '(18) WinHttpRequestOption_EnableCertificateRevocationCheck

        With Request
            .SetTimeouts (lTimeOutSec * 1000), (lTimeOutSec * 1000), (lTimeOutSec * 1000), (lTimeOutSec * 1000)
            .Open "GET", sURL, False

            .Option(4) = 13056 'Set SSL Ignore State, 13056=ignore errors, 0=break on errors
            .Option(6) = True 'Set redirects
            .Option(12) = True 'Allow http to redirect to https

            .Send sPostData
            sResponse = .StatusText
        End With

        MsgBox (sResponse)

        If sResponse = "OK" Then
            bExists = True
        End If

EndNow:
    'MsgBox ("Error # " & Str(Err.Number) & " was generated by " _
    '     & Err.Source & Chr(13) & "Error Line: " & Erl & Chr(13) & Err.Description)

    Set Request = Nothing
    URLExists = bExists

End Function
```

## Returns the value of a ComboBox and the worksheet ROW/COL
```
Sub RepoCommits_Change()
    'get the value of a combobox and identify the objects location as row, column
    Dim b As Object, vName As Variant, RowNo As Integer, ColNo As Integer
    Set b = ActiveSheet.Shapes(Application.Caller)
    With b
        vName = .Name 'Get the name of the object

        With b.TopLeftCell
            ColNo = .Column
            RowNo = .Row
        End With

    End With
    Set b = Nothing

    MsgBox ("Check box Row: " & RowNo)

    'Use the name of the object to get the details
    With ActiveSheet.Shapes(vName).ControlFormat
        MsgBox "Index = " & .Value
        MsgBox "Value = " & .List(.Value)
    End With
    
End Sub
```

## Determin the value of a CheckBox using the control name and identifies the ROW/COL
```
Sub ImproveJSValidation_Click()
    Dim b As Object, vName As Variant, RowNo As Integer, ColNo As Integer, lValue As Long, bState As Boolean
    lValue = 0
    bState = False

    Set b = ActiveSheet.Shapes(Application.Caller)
    With b
        vName = .Name 'Get the name of the object
        With b.TopLeftCell
            ColNo = .Column
            RowNo = .Row
        End With

    End With
    Set b = Nothing


    'Use the name of the object to get the details
    With ActiveSheet.Shapes(vName).ControlFormat
        lValue = CLng(.Value)
        If (lValue > 0) Then
            bState = True
        End If

    End With
    

    If (bState) Then
        'Improvements to JSValidation required
        MsgBox ("Improve JS Required Row: " & RowNo)
    Else
        'No improvements to JSValidation required
        MsgBox ("Improve JS Not Required Row: " & RowNo)

    End If


End Sub
```


