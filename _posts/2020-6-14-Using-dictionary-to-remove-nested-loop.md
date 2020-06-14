---
title: Replacing nested loops with dictionary
tags: [Replace nested loops, Software Performance Tuning optimization technique, Real world software performance]
style: border 
color: light 
description: Let us look at an interesting example of situation where I could gain some CPU at cost of RAM.
---

Let us look at a software performance problem/slowness which I fixed in the VB6 code.

The slowness was when opening a form that showed all the administrators and then rendered a selection of those administrators. You could select a few administrators and those will show as selected when you open the same form again. 
Somehow the administrator user’s information and the information about who all are selected were stored in different database tables. 
Original code worked fine for a few years but, once the amount of data grew if the user had selected a few thousand administrators; loading the screen took a few minutes. 

The original code was written something like below:

<div style="background: #ffffff; overflow:auto;width:auto;border:solid gray;border-width:.1em .1em .1em .8em;padding:.2em .6em;"><table><tr><td><pre style="margin: 0; line-height: 125%"> 1
 2
 3
 4
 5
 6
 7
 8
 9
10
11
12
13
14
15
16
17
18
19
20
21
22
23
24
25
26
27
28
29
30
31
32
33
34
35
36
37
38
39
40
41
42
43
44
45
46
47
48
49
50
51
52
53
54
55
56
57
58
59
60
61
62
63</pre></td><td><pre style="margin: 0; line-height: 125%"><span style="color: #0000aa">Private</span> <span style="color: #0000aa">Sub</span> <span style="color: #00aa00">DisplayAdministrators</span>()
    <span style="color: #0000aa">Const</span> myProcName = <span style="color: #aa5500">&quot;DisplayAdministrators&quot;</span>
    
    <span style="color: #aaaaaa; font-style: italic">&#39;&#39;define columns in administrator array</span>
    <span style="color: #0000aa">Const</span> f_AdminId = <span style="color: #009999">0</span>
    <span style="color: #0000aa">Const</span> f_AdminAbbr = <span style="color: #009999">1</span>
    <span style="color: #0000aa">Const</span> f_Name = <span style="color: #009999">2</span> 

    <span style="color: #0000aa">Dim</span> lng <span style="color: #0000aa">As</span> <span style="color: #00aaaa">Long</span>
    <span style="color: #0000aa">Dim</span> lngRow <span style="color: #0000aa">As</span> <span style="color: #00aaaa">Long</span>
    <span style="color: #0000aa">Dim</span> lngTopRow <span style="color: #0000aa">As</span> <span style="color: #00aaaa">Long</span>
    <span style="color: #0000aa">Dim</span> vntAdministrators <span style="color: #0000aa">As</span> <span style="color: #00aaaa">Variant</span>

    <span style="color: #aaaaaa; font-style: italic">&#39;fill administrators grid</span>
    vntAdministrators = m_objClient.GetAllAdministrators()
    
    <span style="color: #0000aa">If</span> <span style="color: #0000aa">Not</span> IsEmpty(vntAdministrators) <span style="color: #0000aa">Then</span>
        <span style="color: #0000aa">With</span> grdAdministrators
            .Redraw = <span style="color: #0000aa">False</span>
            .MaxRows = UBound(vntAdministrators, <span style="color: #009999">2</span>) + <span style="color: #009999">1</span>
            <span style="color: #0000aa">For</span> lngRow = <span style="color: #009999">0</span> <span style="color: #0000aa">To</span> UBound(vntAdministrators, <span style="color: #009999">2</span>)
                .Row = lngRow + <span style="color: #009999">1</span>
                .Col = <span style="color: #009999">0</span>
                .Text = vntAdministrators(f_AdminId, lngRow)
                .Col = <span style="color: #009999">1</span>
                .Text = vntAdministrators(f_AdminAbbr, lngRow)
                .Col = <span style="color: #009999">2</span>
                .Text = vntAdministrators(f_Name, lngRow)
            <span style="color: #0000aa">Next</span> lngRow
            .Redraw = <span style="color: #0000aa">True</span>
        <span style="color: #0000aa">End</span> <span style="color: #0000aa">With</span>
        
        <span style="color: #aaaaaa; font-style: italic">&#39;set active row and select rows for any AdministratorISs passed in</span>
        <span style="color: #0000aa">With</span> grdAdministrators
            lngTopRow = .MaxRows
            <span style="color: #0000aa">For</span> lng = <span style="color: #009999">0</span> <span style="color: #0000aa">To</span> UBound(m_vntSelectedIDs)
                <span style="color: #0000aa">For</span> lngRow = <span style="color: #009999">1</span> <span style="color: #0000aa">To</span> .MaxRows
                    .Row = lngRow
                    .Col = <span style="color: #009999">0</span>
                    <span style="color: #0000aa">If</span> m_vntSelectedIDs(lng) = .Text <span style="color: #0000aa">Then</span>
                        <span style="color: #0000aa">If</span> lngTopRow &gt;= lngRow <span style="color: #0000aa">Then</span>
                            lngTopRow = lngRow
                        <span style="color: #0000aa">End</span> <span style="color: #0000aa">If</span>
                    <span style="color: #0000aa">End</span> <span style="color: #0000aa">If</span>
                <span style="color: #0000aa">Next</span> lngRow
            <span style="color: #0000aa">Next</span> lng
            .Col = <span style="color: #009999">2</span>
            .Row = lngTopRow
            .Action = ActionActiveCell
            <span style="color: #0000aa">For</span> lng = <span style="color: #009999">0</span> <span style="color: #0000aa">To</span> UBound(m_vntSelectedIDs)
                <span style="color: #0000aa">For</span> lngRow = <span style="color: #009999">0</span> <span style="color: #0000aa">To</span> .MaxRows
                    .Row = lngRow
                    .Col = <span style="color: #009999">0</span>
                    <span style="color: #0000aa">If</span> m_vntSelectedIDs(lng) = .Text <span style="color: #0000aa">Then</span>
                        .SelModeSelected = <span style="color: #0000aa">True</span>
                    <span style="color: #0000aa">End</span> <span style="color: #0000aa">If</span>
                <span style="color: #0000aa">Next</span> lngRow
            <span style="color: #0000aa">Next</span> lng
            .Row = .ActiveRow
        <span style="color: #0000aa">End</span> <span style="color: #0000aa">With</span>
    
    <span style="color: #0000aa">End</span> <span style="color: #0000aa">If</span>
<span style="color: #0000aa">End</span> <span style="color: #0000aa">Sub</span>
</pre></td></tr></table></div>

In the later of the procedure, the nested loops will cause a lot of CPU cycles to be spent to get the output. The inner loop is to find out the intended row from the grid (index of it). 

We can store an administrator ID's index in a dictionary (where key will be administrator ID and the vlaue will be its index in the grid). In the nested loop, we will have to just look up for the administrator ID in the dictionary. 

Lets us look at the code change:

<!-- HTML generated using hilite.me --><div style="background: #ffffff; overflow:auto;width:auto;border:solid gray;border-width:.1em .1em .1em .8em;padding:.2em .6em;"><table><tr><td><pre style="margin: 0; line-height: 125%">  1
  2
  3
  4
  5
  6
  7
  8
  9
 10
 11
 12
 13
 14
 15
 16
 17
 18
 19
 20
 21
 22
 23
 24
 25
 26
 27
 28
 29
 30
 31
 32
 33
 34
 35
 36
 37
 38
 39
 40
 41
 42
 43
 44
 45
 46
 47
 48
 49
 50
 51
 52
 53
 54
 55
 56
 57
 58
 59
 60
 61
 62
 63
 64
 65
 66
 67
 68
 69
 70
 71
 72
 73
 74
 75
 76
 77
 78
 79
 80
 81
 82
 83
 84
 85
 86
 87
 88
 89
 90
 91
 92
 93
 94
 95
 96
 97
 98
 99
100
101
102
103
104
105
106
107
108
109
110
111
112
113
114
115
116
117
118
119
120
121
122
123
124
125
126
127
128
129
130
131
132
133
134
135
136</pre></td><td><pre style="margin: 0; line-height: 125%"><span style="color: #0000aa">Private</span> <span style="color: #0000aa">Sub</span> <span style="color: #00aa00">DisplayAdministrators</span>()
    <span style="color: #0000aa">Const</span> myProcName = <span style="color: #aa5500">&quot;DisplayAdministrators&quot;</span>
    
    <span style="color: #aaaaaa; font-style: italic">&#39;&#39;define columns in administrator array</span>
    <span style="color: #0000aa">Const</span> f_AdminId = <span style="color: #009999">0</span>
    <span style="color: #0000aa">Const</span> f_AdminAbbr = <span style="color: #009999">1</span>
    <span style="color: #0000aa">Const</span> f_Name = <span style="color: #009999">2</span> <span style="color: #0000aa">Private</span> <span style="color: #0000aa">Sub</span> <span style="color: #00aa00">DisplayAdministrators</span>()
    <span style="color: #0000aa">Const</span> myProcName = <span style="color: #aa5500">&quot;DisplayAdministrators&quot;</span>
    
    <span style="color: #aaaaaa; font-style: italic">&#39;&#39;define columns in administrator array</span>
    <span style="color: #0000aa">Const</span> f_AdminId = <span style="color: #009999">0</span>
    <span style="color: #0000aa">Const</span> f_AdminAbbr = <span style="color: #009999">1</span>
    <span style="color: #0000aa">Const</span> f_Name = <span style="color: #009999">2</span> 

	<span style="color: #0000aa">Dim</span> dictAdministratorIdIndex <span style="color: #0000aa">As</span> <span style="color: #0000aa">New</span> Dictionary
    <span style="color: #0000aa">Dim</span> lng <span style="color: #0000aa">As</span> <span style="color: #00aaaa">Long</span>
    <span style="color: #0000aa">Dim</span> lngRow <span style="color: #0000aa">As</span> <span style="color: #00aaaa">Long</span>
    <span style="color: #0000aa">Dim</span> lngTopRow <span style="color: #0000aa">As</span> <span style="color: #00aaaa">Long</span>
    <span style="color: #0000aa">Dim</span> vntAdministrators <span style="color: #0000aa">As</span> <span style="color: #00aaaa">Variant</span>
	<span style="color: #0000aa">Dim</span> vntKey <span style="color: #0000aa">As</span> <span style="color: #00aaaa">Variant</span>

    <span style="color: #aaaaaa; font-style: italic">&#39;fill administrators grid</span>
    vntAdministrators = m_objClient.GetAllAdministrators()
    
    <span style="color: #0000aa">If</span> <span style="color: #0000aa">Not</span> IsEmpty(vntAdministrators) <span style="color: #0000aa">Then</span>
        <span style="color: #0000aa">With</span> grdAdministrators
            .Redraw = <span style="color: #0000aa">False</span>
            .MaxRows = UBound(vntAdministrators, <span style="color: #009999">2</span>) + <span style="color: #009999">1</span>
            <span style="color: #0000aa">For</span> lngRow = <span style="color: #009999">0</span> <span style="color: #0000aa">To</span> UBound(vntAdministrators, <span style="color: #009999">2</span>)
                .Row = lngRow + <span style="color: #009999">1</span>
                .Col = <span style="color: #009999">0</span>
                .Text = vntAdministrators(f_AdminId, lngRow)
				dictAdministratorIdIndex.Add <span style="color: #0000aa">CStr</span>(vntAdministrators(f_AdminId, lngRow)), lngRow + <span style="color: #009999">1</span> <span style="color: #aaaaaa; font-style: italic">&#39;new line</span>
                .Col = <span style="color: #009999">1</span>
                .Text = vntAdministrators(f_AdminAbbr, lngRow)
                .Col = <span style="color: #009999">2</span>
                .Text = vntAdministrators(f_Name, lngRow)
            <span style="color: #0000aa">Next</span> lngRow
            .Redraw = <span style="color: #0000aa">True</span>
        <span style="color: #0000aa">End</span> <span style="color: #0000aa">With</span>
        
        <span style="color: #aaaaaa; font-style: italic">&#39;set active row and select rows for any AdministratorIDs passed in</span>
        <span style="color: #0000aa">With</span> grdAdministrators
            lngTopRow = .MaxRows
			<span style="color: #0000aa">If</span> vntAdministrators.Count &gt; <span style="color: #009999">0</span> <span style="color: #0000aa">Then</span>
				<span style="color: #0000aa">For</span> lng = <span style="color: #009999">0</span> <span style="color: #0000aa">To</span> UBound(m_vntSelectedIDs)
                    lngRow = ItemOrMinusOne(vntAdministrators, m_vntSelectedIDs(lng)) <span style="color: #aaaaaa; font-style: italic">&#39; New line to replace the inner loop with a dictionary search</span>
                    <span style="color: #0000aa">If</span> lngRow &gt; <span style="color: #009999">0</span> <span style="color: #0000aa">And</span> lngTopRow &gt;= lngRow <span style="color: #0000aa">Then</span>
                        lngTopRow = lngRow
                    <span style="color: #0000aa">End</span> <span style="color: #0000aa">If</span>
                <span style="color: #0000aa">Next</span>
			<span style="color: #0000aa">End</span> <span style="color: #0000aa">If</span>
            .Col = <span style="color: #009999">2</span>
            .Row = lngTopRow
            .Action = ActionActiveCell
			<span style="color: #0000aa">If</span> vntAdministrators.Count &gt; <span style="color: #009999">0</span> <span style="color: #0000aa">Then</span>
				<span style="color: #0000aa">For</span> lng = <span style="color: #009999">0</span> <span style="color: #0000aa">To</span> UBound(m_vntSelectedIDs)
                    lngRow = ItemOrMinusOne(vntAdministrators, m_vntSelectedIDs(lng))
                    <span style="color: #0000aa">If</span> lngRow &gt;= <span style="color: #009999">0</span> <span style="color: #0000aa">Then</span>
                        .Row = lngRow
                        .Col = <span style="color: #009999">0</span>
                        .SelModeSelected = <span style="color: #0000aa">True</span>
                    <span style="color: #0000aa">End</span> <span style="color: #0000aa">If</span>
                <span style="color: #0000aa">Next</span> lng
			<span style="color: #0000aa">End</span> <span style="color: #0000aa">If</span>
            .Row = .ActiveRow
        <span style="color: #0000aa">End</span> <span style="color: #0000aa">With</span>
    
    <span style="color: #0000aa">End</span> <span style="color: #0000aa">If</span>
<span style="color: #0000aa">End</span> <span style="color: #0000aa">Sub</span>

<span style="color: #0000aa">Public</span> <span style="color: #0000aa">Function</span> <span style="color: #00aa00">ItemOrMinusOne</span>(<span style="color: #0000aa">ByVal</span> dictSource <span style="color: #0000aa">As</span> Dictionary, <span style="color: #0000aa">ByVal</span> strIndexKey <span style="color: #0000aa">As</span> <span style="color: #00aaaa">String</span>) <span style="color: #0000aa">As</span> <span style="color: #00aaaa">Long</span>
    <span style="color: #0000aa">On</span> <span style="color: #0000aa">Error</span> <span style="color: #0000aa">GoTo</span> Proc_Error
    ItemOrMinusOne = dictSource(strIndexKey)
Proc_Exit:
    <span style="color: #0000aa">Exit</span> <span style="color: #0000aa">Function</span>
<span style="color: #00aa00">Proc_Error</span>:
    ItemOrMinusOne = -<span style="color: #009999">1</span>
    <span style="color: #0000aa">Resume</span> Proc_Exit
<span style="color: #0000aa">End</span> <span style="color: #0000aa">Function</span>

    <span style="color: #0000aa">Dim</span> lng <span style="color: #0000aa">As</span> <span style="color: #00aaaa">Long</span>
    <span style="color: #0000aa">Dim</span> lngRow <span style="color: #0000aa">As</span> <span style="color: #00aaaa">Long</span>
    <span style="color: #0000aa">Dim</span> lngTopRow <span style="color: #0000aa">As</span> <span style="color: #00aaaa">Long</span>
    <span style="color: #0000aa">Dim</span> vntAdministrators <span style="color: #0000aa">As</span> <span style="color: #00aaaa">Variant</span>

    <span style="color: #aaaaaa; font-style: italic">&#39;fill administrators grid</span>
    vntAdministrators = m_objClient.GetAllAdministrators()
    
    <span style="color: #0000aa">If</span> <span style="color: #0000aa">Not</span> IsEmpty(vntAdministrators) <span style="color: #0000aa">Then</span>
        <span style="color: #0000aa">With</span> grdAdministrators
            .Redraw = <span style="color: #0000aa">False</span>
            .MaxRows = UBound(vntAdministrators, <span style="color: #009999">2</span>) + <span style="color: #009999">1</span>
            <span style="color: #0000aa">For</span> lngRow = <span style="color: #009999">0</span> <span style="color: #0000aa">To</span> UBound(vntAdministrators, <span style="color: #009999">2</span>)
                .Row = lngRow + <span style="color: #009999">1</span>
                .Col = <span style="color: #009999">0</span>
                .Text = vntAdministrators(f_AdminId, lngRow)
                .Col = <span style="color: #009999">1</span>
                .Text = vntAdministrators(f_AdminAbbr, lngRow)
                .Col = <span style="color: #009999">2</span>
                .Text = vntAdministrators(f_Name, lngRow)
            <span style="color: #0000aa">Next</span> lngRow
            .Redraw = <span style="color: #0000aa">True</span>
        <span style="color: #0000aa">End</span> <span style="color: #0000aa">With</span>
        
        <span style="color: #aaaaaa; font-style: italic">&#39;set active row and select rows for any AdministratorISs passed in</span>
        <span style="color: #0000aa">With</span> grdAdministrators
            lngTopRow = .MaxRows
            <span style="color: #0000aa">For</span> lng = <span style="color: #009999">0</span> <span style="color: #0000aa">To</span> UBound(m_vntSelectedIDs)
                <span style="color: #0000aa">For</span> lngRow = <span style="color: #009999">1</span> <span style="color: #0000aa">To</span> .MaxRows
                    .Row = lngRow
                    .Col = <span style="color: #009999">0</span>
                    <span style="color: #0000aa">If</span> m_vntSelectedIDs(lng) = .Text <span style="color: #0000aa">Then</span>
                        <span style="color: #0000aa">If</span> lngTopRow &gt;= lngRow <span style="color: #0000aa">Then</span>
                            lngTopRow = lngRow
                        <span style="color: #0000aa">End</span> <span style="color: #0000aa">If</span>
                    <span style="color: #0000aa">End</span> <span style="color: #0000aa">If</span>
                <span style="color: #0000aa">Next</span> lngRow
            <span style="color: #0000aa">Next</span> lng
            .Col = <span style="color: #009999">2</span>
            .Row = lngTopRow
            .Action = ActionActiveCell
            <span style="color: #0000aa">For</span> lng = <span style="color: #009999">0</span> <span style="color: #0000aa">To</span> UBound(m_vntSelectedIDs)
                <span style="color: #0000aa">For</span> lngRow = <span style="color: #009999">0</span> <span style="color: #0000aa">To</span> .MaxRows
                    .Row = lngRow
                    .Col = <span style="color: #009999">0</span>
                    <span style="color: #0000aa">If</span> m_vntSelectedIDs(lng) = .Text <span style="color: #0000aa">Then</span>
                        .SelModeSelected = <span style="color: #0000aa">True</span>
                    <span style="color: #0000aa">End</span> <span style="color: #0000aa">If</span>
                <span style="color: #0000aa">Next</span> lngRow
            <span style="color: #0000aa">Next</span> lng
            .Row = .ActiveRow
        <span style="color: #0000aa">End</span> <span style="color: #0000aa">With</span>
    
    <span style="color: #0000aa">End</span> <span style="color: #0000aa">If</span>
<span style="color: #0000aa">End</span> <span style="color: #0000aa">Sub</span>
</pre></td></tr></table></div>


Well, the above code is not perfect (it is not a single solution to replace every nested loop). It will use a little more RAM than before. But, in my case, the increase was just a few MB and we could afford that.

I hope this helps in case you come across the same problem.