# 学号抽号器设计与探究
## 目录

[TOC]

---

- 明确问题
- 界面设计
- 程序思想与基本实现
- 问题发现与反思
- 重新设计
- 细节处理——Randomize
***
## 明确问题
在设计一个程序时，我们首先需要思考我们要编写一个什么样的程序，或者程序需要具备什么样的功能。非常明显，学号抽号器的功能就是抽取学号。但在本程序中，这仅仅是其中的一部分。我们希望能够加入一个新的功能——最小间隔，以此来保证抽取的学号不会过于集中。

## 界面设计
根据学号抽号器的基本需求，我们需要在窗体上放置5个标签、4个文本框、1个按钮和1个列表框，如图所示：
![](https://github.com/YunyushuLiu/XueHaoChouHaoQi/blob/master/xhchq-image/interface.png)

## 程序思想与基本实现
完成运行界面的设计之后我们需要思考程序如何实现它的功能。首先，在用户按下按钮时，抽号才开始，因此，我们需要对Command1的Click事件作出响应：
```vb
  Private Sub Command1_Click()
  
  End Sub
```
在用户按下按钮时，我们首先需要创建变量并获取用户输入的数据。在这里，我们用qs表示起始号，zz表示终止号，n表示产生数，jg表示最小间隔：
```vb
  Dim qs,zz,n,jg As Integer
  qs = Val(Text1.Text)
  zz = Val(Text2.Text)
  n = Val(Text3.Text)
  jg = Val(Text4.Text)
```
获取用户输入的数据之后，我们还需要对输入的数据进行判断。1.qs不应该大于zz；2.jg不应该小于0（用户不填写jg为0，此时用代码将其置为1）；3.n不应该小于等于0；4.当zz-qs小于n-1个间隔时，无法生成满足条件的随机数。基于以上4点进行如下处理：
```vb
  If qs > zz Then
    Msgbox("请正确输入起始号、终止号！")
    Exit Sub
  End If
  If jg < 0 Then
    Msgbox("请正确输入最小间隔！")
    Exit Sub
  ElseIf jg = 0 Then
    jg = 1
  End If
  If n <= 0 Then
    Msgbox("请正确输入产生数！")
    Exit Sub
  End If
  If qs - zz < (n - 1) * jg Then
    Msgbox("条件矛盾，无法产生！")
    Exit Sub
  End If
```
此时错误处理完毕，我们可以开始生成随机数。在本程序中，我们遵循从前往后生成随机数的方法。由分析可知，对于第一个产生的数a，有a∈[qs,zz-(n-1)*jg]。将生成的随机数储存在变量rd中，那么下一个数的起始位置变为a+jg。对于第二个生成的数b，有b∈[a+jg,zz-(n-2)*jg]。以此类推。
所以在错误处理完毕之后，我们需要将List1清空，并且创建变量i表示正在生成的随机数的序号：
```vb
  List1.Clear
  Dim i,rd AS Integer
  For i = 1 To n
    Randomize
    rd = Int(Rnd() * (zz - (n - i) * jg - qs + 1) + qs)
    qs = rd + jg
    List1.AddItem CStr(rd)
  Next i
```
