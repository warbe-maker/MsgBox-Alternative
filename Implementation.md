# Implementation
## General
- The implementation of the _Message Form_ (the UserForm _fMsg_) is mostly design driven. I.e. the number of available _Message Sections_, the number of reply _Buttons_, and the number of reply  _Buttons Rows_ is primarily a matter of the design and it's change requires only moderate code change.
As a consequence the implementation relies on the hierarchical order of the frames and controls therein. Controls object names  are used only where unavoidable as is for the click events of the reply _Buttons_ (at least I havenˋt found a way to avoid this).

- In order to keep the number of the to-be-installed modules at minimum (_mMsg_ and _fMsg_) the implementation does not make use of any Class Module - though they would have allowed a more elegant code.

- All controls (frames, text boxes, and command buttons) are collected at the _Message Form's_ initialization and these collections are used throughout the implementation.

```vbscript
' Returns all controls of type (ctltype) which do have a parent (fromparent)
' as collection (into) by assigning the an initial height (ctlheight) and width (ctlwidth).
' -----------------------------------------------------------------------------------------
Private Sub Collect(ByRef into As Collection, _
                    ByVal fromparent As Variant, _
                    ByVal ctltype As String, _
                    ByVal ctlheight As Single, _
                    ByVal ctlwidth As Single)

    Dim ctl As MSForms.Control
    Dim v   As Variant
     
    On Error GoTo on_error
    
    Set into = Nothing: Set into = New Collection
    Select Case TypeName(fromparent)
        Case "Collection"
            '~~ Parent is each frame in the collection
            For Each v In fromparent
                For Each ctl In Me.Controls
                    If TypeName(ctl) = ctltype And ctl.Parent Is v Then
                        With ctl
                            .Visible = False
                            .Height = ctlheight
                            .width = ctlwidth
                        End With
                        into.Add ctl
                    End If
               Next ctl
            Next v
        Case Else
            For Each ctl In Me.Controls
                If TypeName(ctl) = ctltype And ctl.Parent Is fromparent Then
                    With ctl
                        .Visible = False
                        .Height = ctlheight
                        .width = ctlwidth
                    End With
                    into.Add ctl
                End If
            Next ctl
    End Select
exit_proc:
    Exit Sub
    
on_error:
    Debug.Print Err.Description: Stop: Resume Next
End Sub
```
## Width Adjustments
### Width Increments
The _Message Form_ is initialized with the specified minimum message form width (see [Default Value Constants](#default-value-constants) which may be modified via the public property _MaxFormWidthPrcntgOfSceenSize_ (see [Public Properties of the _Message Form_](#public-properties-of-the-message-form)). A width increment may be triggered by the setup (in the outlined sequence) of the following the width determining elements:
  1. **Title**  
When the **Title** exceeds the specified  maximum message form width some text will be truncated. However, with a default maximum message form width of 80 % of the screen width that will happen pretty unlikely.
  2. **Mono-spaced message section** followed by **Replies Rows**  
When either of the two exceeds the maximum message form width it will get a horizontal scroll bar.
  3. **Proportional spaced message sections**  
are setup at last and adjusted to the (by then) final message form width.

### Width Decrements
When the _Message Area's_ plus the _Button Area's_ height take more than the available maximum specified _Message Form_ height vertical scroll bars come into play. However, the space required for the scroll bar requires an reduction of the _Proportional Spaced_ Message sections and/or the _Button Rows_. In the first case this additionally - increases the _Message Section's_ height, in the second case the Button Rows may require a horizontal scroll bar.

```vbscript
' Re-adjust width of message section text and
' adjust frames height accordingly
' ---------------------------------------------
Private Sub MsgSectionDecrementWidth( _
            ByVal section As Long, _
            ByVal newwidth As Single)

    Dim s As String
    Dim siNewHeight As Single
     
    With MsgSectionText(section)
        s = .Value
        .Value = vbNullstring
         .AutoSize = False
        .Width = newwidth
        .MultiLine = True
        .AutoSize = True
        .Value = s
        MsgSectionTextFrame.Height = .Height + F_MARGIN
        MsgSectionTextFrame.Width = .Width + F_MARGIN
    End With
    
End Sub
```

## Height Adjustments
### Height Increments
The height of the _Message Form_ is incremented along with the setup of the _Message Sections_ and the setup of the _Reply Buttons_ and the _Reply Button Rows_ respectively.

These height increments are at first done without considering any specified maximum _Message Form_ height.

### Height Decrements
When all elements are setup and the message form exceeds the specified maximum height (see [Default Value Constants](#default-value-constants) which may be modified via the [Public Property of the _Message Form_](#public-properties-of-the-message-form) _MaxFormHeightPrcntgOfSceenSize_, the height of the _Message Area_ frame and/or the _Buttons Area_ frame is reduced to fit and provided with a vertical scroll bar.

|Case|Action|
|----|------|
| The _Message Area_ takes 60% or more of the total areas height | The _Message Area_ gets a vertical scrollbar|
|The _Buttons Area_ takes 60% or more if the total areas height|The _Message Area_ gets a vertical scrollbar |
| Otherwise | Both areas get a vertical scroll bar|
 

## Vertical Re-positioning  
Adjusting the top position of displayed elements is due when all elements are setup. Subsequent re-positioning is due when the [width is decreased](#width-decrease) in order to make space for a vertical scroll bar. Together with the adjustment of the top position of the bottom-most element the new height of the message form is set - limited by the specified maximum form height.

Note: This re-positioning may be done at any time for testing in order to display the current setup state.

## Default Value Constants 

| Constant | Meaning |
| -------- | ------- |
| MONOSPACED_FONT_NAME | Default Font Name for mono-spaced message section text |
| MONOSPACED_FONT_SIZE | Default Font Size for mono-spaced message section text|
| FORM_WIDTH_MIN | Minimum _Message Form_ wird in pt|            | FORM_WIDTH_MAX_POW | Maximum _Message Form_ width as % of the screen size |
| FORM_HEIGHT_MAX_POW | |
| MIN_WIDTH_REPLY_BUTTON | Minimum width of a _Reply Button_ |


## Common public properies

<small>[See chapter in README](<README.md#properties-of-the-fmsg-userform>)</small>                         

## Public properties for advanced usage of the message form
 
|          Property               | R/W | Default |
| ------------------------------- |:---:|:-------:|
| MaxFormHeightPrcntgOfScreenSize | R/W |  90 %   |
| MaxFormWidthPrcntgOfScreenSize  | R/W |  80 %   |
| MaxFormHeight                   | R   | <small>derived from the<br>corresponding percentage value</small>  |
| MaxFormWidth                    | R   | <small>derived from the<br> corresponding  percentage value</small>  |
| MinButtonWidth                  | R/W |   70pt  |
| MinFormWidth                    | R/W |  300pt  |
| HmarginButtons                  | W   |    4pt  |


## Public Properties for test only
|       Property        | R/W | Meaning |
| --------------------- |:---:| ------- |
| _TestFramesWithCaption_ | W   | Defaults to False. Frames are displayed with their "test" caption |
| _TestFramesWithBorder_  | W   | Defaults to False. Frames are displayed with a visible border |
| _HmarginFrames_         | W   | Defaults to 0. May be used when borders are displayed to make their positioning transparent|
