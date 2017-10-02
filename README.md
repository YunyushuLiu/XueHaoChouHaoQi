# 学号抽号器设计与探究——刘云玉树
## 目录
1. 明确问题
2. 界面设计
3. 程序思想与基本实现
4. 问题发现与反思
5. 重新设计
6. 细节处理——Randomize
***
## 1.明确问题
在设计一个程序时，我们首先需要思考我们要编写一个什么样的程序，或者程序需要具备什么样的功能。非常明显，学号抽号器的功能就是抽取学号。但在本程序中，这仅仅是其中的一部分。我们希望能够加入一个新的功能——最小间隔，以此来保证抽取的学号不会过于集中。

## 2.界面设计
根据学号抽号器的基本需求，我们需要在窗体上放置5个标签、4个文本框、1个按钮和1个列表框，如图所示：
![](https://github.com/YunyushuLiu/XueHaoChouHaoQi/blob/master/xhchq-image/interface.png)

## 3.程序思想与基本实现
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

## 4.问题发现与反思
经历重重思考终于完成了这个程序，我们迫不及待运行它了。以起始号1，终止号50，产生数6，最小间隔2为例，我们将程序运行三次：</br>
![](https://github.com/YunyushuLiu/XueHaoChouHaoQi/blob/master/xhchq-image/不等概率1.png)</br>
![](https://github.com/YunyushuLiu/XueHaoChouHaoQi/blob/master/xhchq-image/不等概率2.png)</br>
![](https://github.com/YunyushuLiu/XueHaoChouHaoQi/blob/master/xhchq-image/不等概率3.png)</br>
程序运行看上去非常的好，满足了起始号、终止号、产生数和最小间隔的要求。但是仔细一看，我们就会发现40号及以后的学号出现的频率是异常的，即使运行多次也会发现他们频繁出现。此时我们思考刚才的算法，它在本质上就是不等概率的。仍然以起始号1，终止号50，产生数6，最小间隔2为例，第一个数的范围是[1,40]，如果随机数为20，那么下一个数的范围变成了[22,42]，左端向后移动19，而右端受最小间隔制约只能移动2。基于上述算法的程序，后半部分的几率远高于前半部分。所以，我们的程序并没有满足学号抽号器的要求，接下来需要重新设计算法。

## 5.重新设计
从前到后生成随机数的方法看上去并不可行，那么我们不妨试试数组。</br>
在本算法中，我们使用数组的下标表示数字，元素表示该数字的状态。我们不妨设定：0表示该数字没有产生过，1表示该数字被产生，2表示该数字因为最小间隔不满足要求被禁止生成。同时，我们可以创建一个count变量来统计数组中0的个数，防止程序无法结束循环：
```vb
    Dim a() As Integer '0表示未产生，1表示已经产生过，2表示该数因最小间隔不符要求被禁止生成
    ReDim a(qs To zz) '根据起始号和终止号设置数组
    Dim i, j, rd, count As Integer 'i表示正在产生随机数的序号，rd储存生成的随机数，count表示可以使用的数（即数组a中0的个数）
```
随后，我们开始生成随机数。如果生成的随机数没产生过，那么将其置为1，并且将count减去1；同时要看其周围没生成过的数字是否满足最小间隔，如果不满足，将其置为2，并将count减去1。当count为0而生成序号加1之后小于等于n，则表明未完成任务时已经没有可以生成的数字了，那么此时生成过程被重置。基于以上考虑，我们可以作如下处理：
```vb
    i = 1: j = 0: rd = 0: count = zz - qs + 1
    Do While i <= n '生成n个随机数
        Randomize
        rd = Int(Rnd() * (zz - qs + 1) + qs)
        If a(rd) = 0 Then 'rd未产生过
            a(rd) = 1: i = i + 1: count = count - 1: If i > n Then Exit Do
            For j = 1 To jg - 1
                If rd + j <= zz Then '按照最小间隔锁定右边的数据区域
                    If a(rd + j) = 0 Then '防止重复锁定导致计数错误
                        a(rd + j) = 2: count = count - 1
                    End If
                End If
                If rd - j >= qs Then
                    If a(rd - j) = 0 Then
                        a(rd - j) = 2: count = count - 1
                    End If
                End If
                If count <= 0 Then '未完成任务时无可用的数字可以生成，重置生成过程
                    For i = qs To zz
                        a(i) = 0
                    Next i
                    i = 1: count = zz - qs + 1: j = 0
                    Exit For
                End If
            Next j
        End If
    Loop
```
循环结束则表明随机数已经产生完毕，那么我们就可以根据数组a对产生的随机数进行输出：
```vb
    For i = qs To zz
        If a(i) = 1 Then
            List1.AddItem CStr(i)
        End If
    Next i
```
算法终于重新设计完毕，我们又迫不及待运行它了，这次它真的没让我们失望：</br>
![](https://github.com/YunyushuLiu/XueHaoChouHaoQi/blob/master/xhchq-image/等概率1.png)</br>
![](https://github.com/YunyushuLiu/XueHaoChouHaoQi/blob/master/xhchq-image/等概率2.png)</br>
![](https://github.com/YunyushuLiu/XueHaoChouHaoQi/blob/master/xhchq-image/等概率3.png)</br>

## 6.细节处理——Randomize
在学习VB的过程中，我们在考试中已经可以非常熟练地使用Rnd()函数了。但在作业和考试中，我们可能很难看到Randomize的身影。而对Randomize的思考，源于第二种算法在运行时总是得到相同的结果。我和我的同伴为学号抽号器编写了不同的程序，但我们惊奇地发现虽然是两个独立运行的程序，运行结果却是完全相同。经历了无数次内心的挣扎，我们才想到Randomize。最终它给了我们满意的答案，学号抽号器才真正编写完成。以下是未使用Randomize的随机数生成程序：</br>
![](https://github.com/YunyushuLiu/XueHaoChouHaoQi/blob/master/xhchq-image/无randomize1.png)</br>
![](https://github.com/YunyushuLiu/XueHaoChouHaoQi/blob/master/xhchq-image/无randomize2.png)</br>
我们可以发现独立运行的程序会得到相同的答案，这明显不是我们想看到的。而加上Randomize之后，情况有了明显改善：</br>
![](https://github.com/YunyushuLiu/XueHaoChouHaoQi/blob/master/xhchq-image/randomize1.png)</br>
![](https://github.com/YunyushuLiu/XueHaoChouHaoQi/blob/master/xhchq-image/randomize2.png)</br>
由此可见，Randomize对于一个随机数产生程序来说是十分重要的。

## 7.写在最后的话
纸上的代码是我们的梦想，运行的代码大概就是现实吧。只有真正着手去实践，我们才能得到最真实、最满意的答案。
