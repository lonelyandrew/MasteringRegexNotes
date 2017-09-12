# 第二章 入门示例拓展

## Perl简单入门
Perl中变量名由`$`开头, 下面是一个简单的Perl程序:

```perl
> cat test.pl

$celsius = 20;

while($celsius <= 45)
{
    $fahrenheit = ($celsius * 9 / 5) + 32;
    print "$celsius C is $fahrenheit F.\n";
    $celsius = $celsius + 5;
}

> perl test.pl

20 C is 68 F.
25 C is 77 F.
30 C is 86 F.
35 C is 95 F.
40 C is 104 F.
45 C is 113 F.
```

## 使用正则表达式匹配文本

下面的代码用来检查`$reply`中所含的字符串是否全部由数字组成:

```perl
> cat test.pl

$reply = "1234";
if ($reply =~ m/^[0-9]+$/)
{
    print "only digits\n";
}
else
{
    print "not only digits\n";
}

> perl test.pl

only digits
```

`m/・・・/`表示进行正则表达式匹配, 其中`m`可以省略, `=~`用来连接正则表达式和待检查的字符串.

Perl中`==`用来测试两个数值是否相等, `eq`运算符用来检查两个字符串是否相等, `=`用来赋值.

### 更实用的程序

```perl
> cat test.pl

print "Enter a temperature in Celsius:\n";
$celsius = <STDIN>;
chomp($celsius);

if ($celsius =~ m/^[+-]?[0-9]+(\.[0-9]*)?$/)
{
    $fahrenheit = ($celsius * 9 / 5) + 32;
    printf "%.2f is %.2f F\n", $celsius, $fahrenheit;
}
else
{
    print "Excepting a number, so I don't understand \"$celsius\".\n";
}

> perl test.pl

Enter a temperature in Celsius:
22
22.00 is 71.60 F

> perl test.pl

Enter a temperature in Celsius:
foo
Excepting a number, so I don't understand "foo".
```

### 成功匹配的副作用
成功匹配之后, Perl支持在正则表达式之外引用匹配的文本, (其实就是Python中`group`的功能), 我们需要把需要引用文本所属的子表达式放进一对括号之中, 这种括号叫做**捕获型括号**. 需要引用时, 使用`$1`, `$2`, `$3`来引用, 变量名就是数字, 每匹配成功一次, 这些变量就重置一次, (有点类似于vim中的特殊寄存器).

### 非捕获型括号
Perl以及近期出现的其他正则表达式流派提供了**非捕获型括号**. 用｢(・・・)｣用来分组和捕获, 而｢(?:・・・)｣只分组不捕获(这个功能在Python也支持).

### Perl常见元字符
+ `\t`: Tab制表符
+ `\n`: 换行符
+ `\f`: 进纸符
+ `\b`: 退格符
+ '\r': 回车符
+ `\w`: ｢a-zA-Z0-9｣
+ `\W`: ｢\^a-zA-Z0-9｣
+ `\d`: ｢0-9｣
+ `\D`: ｢^0-9｣
+ `\s`: 任何空白字符
+ `\S`: 任何非空白字符
元字符在正则表达式中并不是统一的, 取决于具体的情况.

### 忽略大小写
Perl中可以在匹配表达式的最后添加`i`来忽略匹配的大小写.

```Perl
$input =~ m/^([-+]?[0-9]+(\.[0-9]*)?\s*[CF])$/i
```

此处`i`叫做*修饰符*. 类似于grep的`-i`选项. 其他的常见修饰符有`/g`(全局匹配), `/x`(宽松排列的表达式).

### 温度转换的最终版本

```Perl
print "Enter a temperature (e.g., 32F, 100C): \n";
$input = <STDIN>;
chomp($input);

if ($input =~ m/^([+-]?[0-9]+(\.[0-9]+)?)\s*([CF])?$/i)  # 添加了小数和忽略大小写
{
    $InputNum = $1;
    $type = $3;  # 注意这里的分组顺序也发生了变化

    if ($type =~ m/c/i)  # 这里的eq被match代替了, 更加简洁
    {
        $celsius = $InputNum;
        $fahrenheit = ($celsius * 9 / 5) + 32;

    }
    else
    {
        $fahrenheit = $InputNum;
        $celsius = ($fahrenheit - 32) * 5 / 9;
    }
    printf "%.2f is %.2f F\n", $celsius, $fahrenheit;
}
else
{
    print "Excepting a number followed by \"C\" or \"F\",";
    print "so I don't understand \"$input\".\n";
}
```

## 使用正则表达式修改文本
Perl在匹配的基础上也可以用来做替换:

```Perl
$var =~ s/reg/replacement/;
```

如果`$var`能够匹配`reg`中的表达式, 那么其中匹配到的片段将会被替换为`replacement`的内容.

这里还需要提到两个极其重要内容:
+ grep支持的单词分界符为`\<`和`\>`, 而Perl中单词分界符为`\b`.
+ Perl中提供了`/g`为**全局替换**修饰符. (和Vim中的替换相同)

### 自动编辑的操作
Perl可以从命令行直接执行语句:

```bash
% perl -p -i -e 's/sysread/read/g' file
```

其中`-p`表示对`file`中的每一行进行查找替换, `-i`表示将替换完成的结果回写到原文件中, `-e`表示把这一行代码当做程序来运行.

### 用环视功能为数值添加逗号
环视结构不匹配任何字符, 也不占用字符, 只匹配文本中的特定位置, 和单词分界符或者句子分界符类似.

环视有顺序缓释和逆序环视(其符号为｢(?<=・・・)｣), 而其中又分为肯定型环视(其符号为｢(?=・・・)｣)和否定型环视.

|  类型 |  正则表达式|  匹配成功的条件 |
| --- | --- | --- |
| 肯定逆序环视 </br> 否定逆序环视|`(?<=......)`</br> `(?<!......)`| 子表达式能够匹配左侧文本 </br> 子表达式不能匹配左侧文本 |
| 肯定顺序环视 </br> 否定顺序环视 |`(?=......)`</br> `(?!......)`| 子表达式能够匹配右侧文本 </br> 子表达式不能匹配右侧文本 |

### Text-to-HTML转换

```Perl
undef $/;  # 进入"file-slurp"(文件读取)模式
$text = <>;  # 读入命令行中指定的第一个文件

$text =~ s/&/&amp;/g;  # 替换的顺序很重要, 因为后面的符号里都有&
$text =~ s/</&lt;/g;
$text =~ s/>/&gt;/g;

# 此处的`/m`修饰符为增强的行锚点(enhanced line anchor)匹配模式, 
# ｢^｣和｢$｣会从字符串模式切换到逻辑行模式, 
# 即从匹配字符串的开头和结尾切换到切换到匹配一行文本的开头和结尾,
# 因为我们读取文件时把所有行连接成了一个字符串, 故需要切换到逻辑行模式
# 在Python中有名为re.M或re.MULTILINE的flag可以实现同样的切换
$text =~ s/^\s*$/<p>/mg;

# 此处的qr和前面的m或者s类似, 但是不把正则表达式应用到文本中, 而是储存成regex对象.
$HostnameRegex = qr/[-a-z0-9]+(\.[-a-z0-9]+)*\.(com|edu|info)/i;

# 前面的语句中, 我们的替换和匹配都使用'/'作为分隔符
# 然而如果我们使用斜线的话就必须转义
# 例如在最后的'</a>'中就要替换为'<\/a>', 非常不美观,
# Perl允许我们自定义分隔符, 例如's!regex!string!modifiers',
# 我们使用s{regex}{string}modifiers
# 在最后的修饰符里使用了新的｢/x｣修饰符
# 它是宽松排版(free-format)修饰符, 首先它可以增强可读性, 其次它容许出现以#开头的注释
# 加上｢/x｣以后大部分的空格符变为"忽略自身元字符"(ignore me metacharacter)
# 而'#'的意思是忽略该字符及其第一个换行符之前的所有字符
# Python中的flag为re.X或re.VERBOSE
$text =~ s{
    \b
    (
        \w[-.\w]*  # username
        \@  # @被Perl用来标记数组名, 所以需要转义
        $HostnameRegex  # hostname
    )
    \b
}{<a href="mailto:$1">$1</a>}/gix;

$text =~ s{
    \b
    (
        http:// $HostnameRegex \b
            (
                / p[-a-z0-9_:\@&?=+,.!/~*'%\$]*
                   (?<![.,?!])
            )?
    )
}{<a href="$1">$1</a>}/gix;

print $text;
```

(第二章结束)


