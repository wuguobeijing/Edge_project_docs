iptables一般情况下是不支持对数据包内部信息进行匹配，主要是用来作数据的五元组匹配。为了解决更加个性化的报文内容匹配，iptables引入了“u32”的模块来帮助我们对于数据包内部的内容进行匹配识别。
u32模块一次可以读取报文内32bits的内容，可以针对想要匹配到的这32位数据（四个字节）进行深层次的匹配与控制。
在使用时，-match可以简写为-m，整体语句通常写为**iptables -m u32 --u32 "Start&Mask=Range" -j policy**的形式。
Start表示从哪里开始读取这32位数据，可以选择直接从头开始指定位数进行数据截取，也可以使用通过取出报头长度字段并用@来跳过ip和tcp的报头，从而到达应用层的位置。
Mask表示用以处理这32位数据的掩码，数据与掩码进行逐位运算并得到一个结果。
Range表示用来匹配上述Mask处理结果的方法，如果匹配的结果为真，则句尾的policy处理规则就会被执行，若结果为假，则不触发policy方法。

精简：
引入iptables的“match -u32”的模块来实现对于数据包内部的报文内容进行精准的匹配识别及控制；通过读取报文内指定的32位（4字节）的内容，并将匹配规则掩码与这32位内容按位进行运算得到结果，进而判定数据的合规性。