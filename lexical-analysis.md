# 词法分析 - Lexical analysis

>
２.词法分析
一个 Python 程序是由一个解析器读取的。解析器的输入是一个由词法分析器生成的符号流。本章介绍了词法分析器如何将文件分解为符号。
Python 将程序文本以 Unicode 代码点的方式读入；源文件的编码格式由编码声明给出，其默认值为 UTF-8 ，详见 PEP 3120 。源文件无法被解析时，会引发 SyntaxError 异常。
2.1.行结构
一个 Python 程序会被分成多个逻辑行。
2.1.1. 逻辑行
逻辑行使用一个 NEWLINE 标记结尾。除非在语法中允许使用 NEWLINE (例如，复合语句中的声明语句之间)，否则声明语句不能跨多个逻辑行。逻辑行是由一个物理行，或多个通过显示或隐式行连接规则连接在一起的物理行组成。
2.1.2. 物理行
物理行是一个以行尾序列结尾的字符序列。在源文件中，任何平台下标准的行终止序列都可以使用-在 Unix 平台上使用 ASCII LF(换行)符，在 Windows 平台上使用 ASCII 字符序列CR LF(回车加换行)，在老版本的 Macintosh 平台上使用 ASCII CR(回车)符。无论使用什么平台，所有这些形式都可以使用。
使用嵌入式Python开发时，需要使用标准 C 语言约定的换行符将源代码传递给 Python API（字符 \n 表示 ASCII LF，即行终止符)。
2.1.3. 注释
注释以(#)字符开始(且此时该符号不是当前字符串的一部分)，以该物理行的结尾为结束。如果没有调用隐式行连接，那么注释即意味着逻辑行的结尾。注释会被语法分析所忽视，它们不会被作为程序代码处理。
2.1.4 编码声明
如果第一或第二行的 Python 脚本注释能够匹配正则表达式 coding[=:]\s*([-\w.]+),该注释会被当做编码声明处理；该表达式的第一部分定义了源代码文件的编码格式。此表达式的推荐形式是
# -*- coding: <encoding-name> -*-  
该表达式也可以被 GNU Emacs 所识别，Bram Moolenaar’s VIM 识别的表达式形式为
# vim:fileencoding=<encoding-name>
如果未发现编码声明，则默认编码格式为 UTF-8。另外，如果此文件的第一个字节是 UTF-8 字节序标记（b'\xef\xbb\xbf')，则声明的编码为 UTF-8（微软的 notepad 支持这种编码格式）。
如果声明了一种编码格式，则编码格式名称必须可以被 Python 识别。所有的词法分析，包括字符串、注释以及标识符都需要使用这种编码格式。编码声明必须出现在其自己的行中。
2.1.5 显式行连接
两个或者多个物理行可以使用反斜线(\)合并为一个逻辑行，例如：当一个物理行以反斜线结束，且该反斜线不是字符串值或注释的一部分时，它就与下面的物理行合并构成一个逻辑行，并将反斜线及行结束符删除。例如：
if 1900 < year < 2100 and 1 <= month <= 12 \  
   and 1 <= day <= 31 and 0 <= hour < 24 \  
   and 0 <= minute < 60 and 0 <= second < 60:   # Looks like a valid date  
return 1
以反斜线结尾的行不能在其后加注释，反斜线不能延续注释行。除了字符串文本，反斜线也不能延续语言符号(也就是说，其他不是字符串文本的语言符号不可以使用反斜线横跨多个物理行)。在字符串文本外的其他地方出现反斜线都是非法的。
2.1.6 隐式行连接
圆括号、 方括号或大括号中的表达式可以跨多个物理行，而无需使用反斜杠。例如：
month_names = ['Januari', 'Februari', 'Maart',  # These are the  
   'April',   'Mei',  'Juni',   # Dutch names  
   'Juli','Augustus', 'September',  # for the months  
   'Oktober', 'November', 'December']   # of the year  
可以在隐式行连接的末尾添加注释。接续行的缩进可以不考虑。并且允许出现空的接续行。在隐式的接续行中是不存在 NEWLINE 符号的。隐式的行连接在三重引用串（后述）中也是合法的，在那种情况下不能添加注释。
2.1.7 空行
一个只包含空格符、制表符、换页符和可能是注释的逻辑行可以忽略（也就是说没有 NEWLINE 符号产生）。在交互式输入语句时，空行的处理可能不同，其依赖于输入-计算-输出循环的实现方式。在标准的交互实现中，一个纯粹的空逻辑行（即不包含任何东西，甚至是空白和注释）可以结束多行语句。
2.1.8 缩进
位于逻辑行开始前的空白（空格和制表符）用于计算行的缩进层次，该层次可用于语句的分组。
制表符被（从左到右）替换为一至八个空格，这样直到包括替换部分的总字符数是八的倍数（这与 Unix 中使用的规则是相同的）。第一个非空字符前的空格总数决定了行的缩进。不能使用反斜线在多个物理行之间对缩进进行拆分；第一个反斜线之前的空白字符用于检测缩进。
在以下情形下缩进会被认为是不符合逻辑的：一个源文件中既包含制表符又包含空格，此时如果该文件的意义依赖于包含在空格符中的制表符时，就会触发 TabError 异常。
跨平台兼容性注意：由于非 UNIX 平台上文本编辑器的特性，在单个源文件中混合使用空格符和制表符的缩进是不明智的。还需要注意的是，不同的平台可能会显式的限制最大缩进级别。
虽然在行首可能会出现换页符；但它在以上的缩进计算中会被忽略。出现在其他位置的换页符的作用是不确定的（比如，它可能将空格数重置为0）。
连续行的缩进层次用于生成 INDENT 及 DEDENT 语言符号，在此过程中使用了堆栈，如下所述。
文件的第一行未被读取之前，一个0被压入栈中；它以后也不会被弹出来。被压入栈中的数字都从栈底向栈顶增长，在每个逻辑行的开头处，将行的缩进层次与栈顶元素比较，如果相等则什么都不做。如果大于栈顶元素，则将其压入栈中并生成一个 INDENT 语言符号。如果小于栈顶元素，那么其应该是堆栈中已经存在的数字中的一个，堆栈中所有大于它的数都将被弹出，并且每个弹出的数字都会生成一个 DEDENT 语言符号。在文件的结尾，每个仍留在栈中且大于0的元素都会生成一个 DEDENT 语言符号。
这里有一个正确（尽管有点乱）缩进格式的 Python 代码的例子：
def perm(l):  
# Compute the list of all permutations of l  
if len(l) <= 1:  
  return [l]  
r = []  
for i in range(len(l)):  
 s = l[:i] + l[i+1:]  
 p = perm(s)  
 for x in p:  
  r.append(l[i:i+1] + x)  
return r
下面的例子展示了多种缩进错误：
def perm(l):                # error: first line indented  
for i in range(len(l)):     # error: not indented  
s = l[:i] + l[i+1:]  
p = perm(l[:i] + l[i+1:])   # error: unexpected indent  
for x in p:  
r.append(l[i:i+1] + x)  
return r                    # error: inconsistent dedent
（实际上，前三种错误都是由解析器检测出来的，只有最后一个错误是由词法分析器发现的——return r的缩进层次与堆栈中弹出的数字不匹配。）
2.1.9 符号间的空白
除非位于逻辑行的行首或者字符串当中，空格符、制表符以及换页符都可以用于分割语言符号。只有当两个符号串接在一起可能会被解释为不同的符号时，才会在两个符号之间增加空格（例如： ab 是一个符号，但 a b 却是两个符号）。
2.2 其他语言符号
除了 NEWLINE， INDENT 和 DEDENT，还有以下几类语言符号：标识符、关键字、文本、运算符及分隔符。空格符不是语言符号（除了断行符，之前讨论过），但是可以用来分隔语言符号。当解释某个语言符号存在歧义时，该语言符号被看作是由一个尽可能长的字符串组成的合法符号（从左至右）。
2.3 标识符及关键字
标识符（也被称为名字）是由以下词法定义描述的。
Python 语言中的标识符语法基于 Unicode 标准附件 UAX-31，阐述变化定义如下；也可以点击 PEP 3131 获得更详细的信息。
在 ASCII 范围内（ U+0001..U+007F ），合法的标识符字符与 Python 2.X版本中是一致的：大写及小写字母的A到Z，下划线_以及数字从0到9（第一个字符除外）。
Python 3.0版本引入了 ASCII 范围以外其他的字符（详见 PEP 3131 ）。这些字符使用包含在 unicodedata 模块中的 Unicode 字符数据库版本进行分类。
标识符的长度是没有限制的。
identifier ::= xid_start xid_continue *
idstart ::= <all characters in general categories Lu, Ll, Lt, Lm, Lo, Nl, the underscore, and characters with the OtherIDStart property>
idcontinue ::= <all characters in id_start , plus characters in the categories Mn, Mc, Nd, Pc and others with the OtherIDContinue property>
xidstart ::= <all characters in id_start whose NFKC normalization is in "idstart xidcontinue">
xidcontinue ::= <all characters in id_continue whose NFKC normalization is in "id_continue">
上面提到的 Unicode 分类代码分别代表：
Lu - 大写字母
LI - 小写字母
Lt - 标题字母
Lm - 修饰字符
Lo - 其他字符
NI - 字母数字
Mn - 无空格标记
Mc - 空格组合标记
Nd - 十进制数字
Pc - 连接符
OtherIDStart - PropList.txt中的显示字符列表，支持后向兼容
OtherIDContinue - 同上
解析器会将所有标识符转换为标准形式的 NFKC，标识符的比较是基于 NFKC 的。
非标准的 HTML 文件列出了Unicode 4.1 中用到的所有合法标识符，可以在http://www.dcl.hpi.uni-potsdam.de/home/loewis/table-3131.html中找到。
2.3.1 关键字
下面列出的标识符被用作保留字，或者叫做该语言的关键字，这些保留字不能作为普通标识符使用。这些关键字必须严格像下面一样拼写：
False   class      finally    is          return
None    continue   for        lambda      try
True    def        from       nonlocal    while
and     del        global     not         with
as      elif       if         or          yield
assert  else       import     pass
break   except     in         raise
2.3.2.保留的标识符类型
不同类型的标识符（关键字除外）都有其特殊含义。这些类型由前导和尾随的下划线字符模式确定：
_*
不能由from module import *导入。特殊字符_，在交互式解释器中用于存储上次估值的结果；它存储在 builtins 模块中。在非交互式模式下，字符 _ 没有特殊含义，且是未定义的。详见 The import statement 。
注意:_ 常用于结合国际化；您也可以通过 gettext 获取更多关于此问题的信息。
_ _*_ _
系统定义的名称。这些名称由解释器及其实现定义(包括标准库)。当前系统名称在 Special method names 部分讨论。更多的系统名称可能会在后续的 Python 版本中定义。在任意上下文中不遵循明确记录的使用_ _ *_ _名字，会更容易产生错误且不会提示任何告警。
_ _* 类私有名称。在一个类定义的上下文中使用此类别中的名字时，为避免基类及派生类中“私有”属性之间的命名冲突，会被重写为一种乱序形式。
2.4 文本
文本是某些内建类型常量的表示方法。
2.4.1. 字符串文本及字节文本
字符串文本由以下词法定义描述：
stringliteral ::= [stringprefix] (shortstring | longstring)
stringprefix::= "r" | "u" | "R" | "U"
shortstring ::= "'" shortstringitem* "'" | '"' shortstringitem* '"'
longstring ::= "'''" longstringitem* "'''" | '"""' longstringitem* '"""'
shortstringitem ::= shortstringchar | stringescapeseq
longstringitem ::= longstringchar | stringescapeseq
shortstringchar ::= <any source character except "\" or newline or the quote>
longstringchar ::= <any source character except "\">
stringescapeseq ::= "\" <any source character>
bytesliteral ::= bytesprefix(shortbytes | longbytes)
bytesprefix::= "b" | "B" | "br" | "Br" | "bR" | "BR" | "rb" | "rB" | "Rb" | "RB"
shortbytes ::= "'" shortbytesitem* "'" | '"' shortbytesitem* '"'
longbytes ::= "'''" longbytesitem* "'''" | '"""' longbytesitem* '"""'
shortbytesitem ::= shortbyteschar | bytesescapeseq
longbytesitem ::= longbyteschar | bytesescapeseq
shortbyteschar ::= <any ASCII character except "\" or newline or the quote>
longbyteschar ::= <any ASCII character except "\">
bytesescapeseq ::= "\" <any ASCII character>
上面没有表示出来的一个语法限制是在 stringprefix 或 bytesprefix 和字符串文本之间不允许出现空格。编码声明定义了源字符集；如果在源文件中未明确声明编码方式，则默认使用 UTF-8 编码；详见 Encoding declarations 部分。
简单来说：两种类型的字符串文本都可以用成对的单引号(')或双引号（"）括起来。它们也可以用成对的三个单引号或双引号括起来（这些通常统称为三重引号的字符串）。反斜线字符（\）称为转义字符，如果不转义则其可能会产生歧义，例如换行符、反斜线本身或者引号。
字节文本总是以 ‘b’ 或 ‘B’ 开头；它们会生成 bytes 类型实例而不是 str 类型。它们可能只包含 ASCII 字符；大于或等于 128 的数字必须使用转义字符表示。
Python 3.3 中可能会再次在字符串文本前增加 u 字符前缀，用以简化 2.X 及 3.X 版本代码的维护工作。
字符串文本及字节文本都可以（可选）加上字符前缀 r 或 R；这样的字符串被称为原始字符串，将反斜线视为原义字符。 这样会导致在字符串文本中，原始串中的转义字符 ‘\U’ 及 ‘\u’ 不会被特殊处理。考虑到 Python 2.X 版本中的原始 unicode 类型的特性与 3.X 版本差距较大，因此在 Python 3.X 版本中不再支持 ‘ur’ 语法。
3.3 版本新特性：原始字节文本的前缀 ‘rb’ 与 ‘br’ 意义相同。
3.3 版本新特性：旧版本中的（u ‘value’）被再次引入，用以简化 Python 2.X 及 3.X 版本的维护工作。更多的信息，请参阅 PEP 414。
在三重引用串中，允许出现未转义的新行和引用字符（并被保留），除非三个连续的引用字符串中断了该串。（引用字符是用于引用字符串的字符，如，’ 或 ”。）
如果给出一个 ‘r’ 或 ‘R’，那么其含义就像标准C中的规则类似的解释，转义序列如下：
转义序列	 意义	 注意事项
\newline
反斜线且忽略换行	
\\
反斜线（\）	
\’
单引号（’）	
\"	  双引号（”）	
\a	 ASCII Bell（BEL）	
\b	 ASCII 退格（BS）	
\f	 ASCII 换页符（FF）	
\n	 ASCII 换行符（LF）	
\r	 ASCII 回车符（CR）	
\t	 ASCII 水平制表符（TAB）	
\v	 ASCII 垂直制表符（VT）	
\ooo	 八进制值为 ooo 的字符	 （1,3）
\xhh	 十六进制值为 hh 的字符	 （2,3）
字符串文本中的转义序列规则如下：
转义序列	 意    义	 注意事项
\N{name}	 Unicode 数据库中以 name 命名的字符	 （4）
\uxxxx	 16位16进制字符值：xxxx	 （5）
\Uxxxxxxxx	 32位16进制字符值：xxxxxxxx	 （6）

注意：
与标准 C 相同的是，最多只可以接受三位八进制数字。
与标准 C 不同的是，只能接收两个十六进制数字。
在字节文本中，十六进制和八进制转义字符表示给定值的字节数。在字符串文本中，这些转义字符表示给定值的 Unicode 字符。
与 3.3 版本不同之处：增加了对别名[ 1 ]的支持。
可以使用该转义序列为那些构成代理对的单个代码单元编码。只能使用四个十六进制数表示。
任何 Unicode 字符都可以采用这样的编码方式。需要注意的是只能使用八个十六进制数表示。
不像标准 C，所有不能被识别的转义序列都保留在串中且不做改变，例如，反斜线会保留在结果中。（这个行为在调试过程中非常有用：如果输入了一个错误的转义序列，在输出结果中更容易识别出错误。）此外，至关重要的是要注意转义字符只能在字符串文本中起作用，在字节文本 类别中无法被识别。
即使在原始文本中，可以使用反斜线将引号转义，但是反斜线本身会在结果中保留；比如 r"\"" 是一个由两个字符组成的合法字符串：一个反斜线与一个双引号；但 r"\" 却是一个非法字符串（即原始的字符串也不能以奇数个反斜杠结尾）。 具体而言，一个原始的文本不能以单个反斜杠结尾（由于反斜线会将跟在其后的引号转义）。另外需要注意的是，如果一个反斜线跟在换行符后，反斜线与换行符会被当做文本的两个字符，而不是一个连续行。
2.4.2 字符串的连接
多个相邻的字符串文本或字节文本（由空白分隔），允许使用不同的引用习惯，并且其含义与连接在一起时是一样的。因此， "hello" ‘world’ 与 "helloworld"是等价的。这个特性可以用来减少反斜线的使用数量，可以很方便的将一个长字符串分隔在多行上，甚至可以在字符串的某一部分添加注释，例如：
re.compile("[A-Za-z_]"           # letter or underscore
           "[A-Za-z0-9_]*"       # letter, digit or underscore
          )
需要注意的是，这个特性是定义在句法层次上的，但是是在编译时实现的。在运行时连接串必须使用 ‘+’ 运算符。并且不同的引用字符可以混用，甚至可以将原始串与三重引用串混合使用。
2.4.3. 数字文本
总共有三种类型数字文本：整型、浮点型以及虚数型。不存在复数文本（复数可以由一个实数加一个虚数的形式给出）。
注意，数字文本不包含符号（正负号）；像 -1 实际上是一个组合了一元运算符 '-' 和数字 1 的表达式。
2.4.4 整型文本
整型文本描述的词法定义如下：
integer::= decimalinteger | octinteger | hexinteger | bininteger
decimalinteger ::= nonzerodigit digit* | "0"+
nonzerodigit ::= "1"..."9"
digit ::= "0"..."9"
octinteger ::= "0" ("o" | "O") octdigit+
hexinteger ::= "0" ("x" | "X") hexdigit+
bininteger ::= "0" ("b" | "B") bindigit+
octdigit ::= "0"..."7" 
hexdigit ::= digit | "a"..."f" | "A"..."F"
bindigit ::= "0" | "1"
除了可以存储在可用内存中外，对整型文本的长度是没有限制的。
注意，不允许在非零的小数前加零。这是为了消除与 Python 3.0 之前版本中使用的 C 风格八进制文本的歧义。
整型文本的例子如下：
7   2147483647                        0o177             0b100110111  
3   79228162514264337593543950336     0o377             0x100000000
    79228162514264337593543950336                       0xdeadbeef
2.4.5. 浮点型文本
浮点型文本描述的词法定义如下：
floatnumber ::= pointfloat | exponentfloat
pointfloat ::= [intpart] fraction | intpart "."
exponentfloat ::= (intpart | pointfloat) exponent
intpart ::= digit+
fraction ::= "." digit+ exponent ::= ("e" | "E") ["+" | "-"] digit+
注意，浮点型文本的整数部分及指数部分都使用十进制表示。比如, 077e010 是合法字符，并且与 77e10 等价。浮点型文本允许的范围是依赖于实现的。一些浮点型文本的例子如下：
3.14    10.    .001    1e100    3.14e-10    0e0
注意，数字文本不包含符号（正负号）；像 -1 实际上是由一元运算符 - 和文本 1 组成。
2.4.6. 虚数文本
虚数文本描述的词法定义如下：
imagnumber ::= (floatnumber | intpart) ("j" | "J")
虚数是一个实部为零的复数，复数代表一对有着相同取值范围限制的浮点数对。为了创建一个实部非零的复数，可以对它增加一个浮点数，例如（3+4j）。下面是一些关于虚数文本的例子：
3.14j   10.j    10j     .001j   1e100j  3.14e-10j
2.5 运算符
下面列出的符号为操作符：
+       -       *       **      /       //      %
<<      >>      &       |       ^       ~
<       >       <=      >=      ==      !=
2.5 分隔符
以下符号用作语法上的分隔符：
(       )       [       ]       {       }
,       :       .       ;       @       =       ->
+=      -=      *=      /=      //=     %=
&=      |=      ^=      >>=     <<=     **=
句号也可以在浮点型及虚数文本中出现，一个连续三个句号的序列具有特殊含义，代表了一个片段中的省略部分。该列表的后半部分，即参数化赋值运算符，在词法上是作为分隔符处理，但也执行运算。
下面列出的 ASCII 字符作为其他符号的一部分具有特殊含义，或者对于词法分析器具有重要意义。
'       "       #       \
下面列出的 ASCII 字符没有在 Python 中使用。当它们出现在字符串文本及注释之外时就认为是非法的。
$       ?       `
脚注
[1] http://www.unicode.org/Public/6.3.0/ucd/NameAliases.txt
