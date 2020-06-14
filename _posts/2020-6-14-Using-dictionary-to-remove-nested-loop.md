---
title: Replacing nested loops with dictionary
tags: [Software Performance Tuning, Software Performance Optimization, Performance Tuning Techniques, Real world software performance]
style: border 
color: light 
description: Let us look at an interesting example of situation where I could gain some CPU at cost of some RAM.
---

Let us look at a software performance problem/slowness which I fixed in the VB6 code.
The slowness was when opening a form that showed all the administrators and then rendered a selection of those administrators. You could select a few administrators and those will show as selected when you open the same form again. 
Somehow the administrator user’s information and the information about who all are selected were stored in different database tables. 
Original code worked fine for a few years but, once the amount of data grew if the user had selected a few thousand administrators; loading the screen took a few minutes. 

The original code was written something like below:

```vb

Private Sub DisplayAdministrators()
    Const myProcName = "DisplayAdministrators"
    
    'define columns in administrator array
    Const f_AdminId = 0
    Const f_AdminAbbr = 1
    Const f_Name = 2 

    Dim lng As Long
    Dim lngRow As Long
    Dim lngTopRow As Long
    Dim vntAdministrators As Variant

    'fill administrators grid
    vntAdministrators = m_objClient.GetAllAdministrators()
    
    If Not IsEmpty(vntAdministrators) Then
        With grdAdministrators
            .Redraw = False
            .MaxRows = UBound(vntAdministrators, 2) + 1
            For lngRow = 0 To UBound(vntAdministrators, 2)
                .Row = lngRow + 1
                .Col = 0
                .Text = vntAdministrators(f_AdminId, lngRow)
                .Col = 1
                .Text = vntAdministrators(f_AdminAbbr, lngRow)
                .Col = 2
                .Text = vntAdministrators(f_Name, lngRow)
            Next lngRow
            .Redraw = True
        End With
        
        'set active row and select rows for any AdministratorISs passed in
        With grdAdministrators
            lngTopRow = .MaxRows
            For lng = 0 To UBound(m_vntSelectedIDs)
                For lngRow = 1 To .MaxRows
                    .Row = lngRow
                    .Col = 0
                    If m_vntSelectedIDs(lng) = .Text Then
                        If lngTopRow >= lngRow Then
                            lngTopRow = lngRow
                        End If
                    End If
                Next lngRow
            Next lng
            .Col = 2
            .Row = lngTopRow
            .Action = ActionActiveCell
            For lng = 0 To UBound(m_vntSelectedIDs)
                For lngRow = 0 To .MaxRows
                    .Row = lngRow
                    .Col = 0
                    If m_vntSelectedIDs(lng) = .Text Then
                        .SelModeSelected = True
                    End If
                Next lngRow
            Next lng
            .Row = .ActiveRow
        End With
    
    End If
End Sub

```
