## flex词法分析实验指导

### 实验目标

本实验目的是加深对词法分析过程的理解，掌握词法分析的方法，使用任何编程语言或者词法分析工具进行词法分析。 分析的语言可以自定义，也可以选择实现现有语言的文法分析。 具体要求如下：

1. 识别出关键字、标识符、常数、运算符、界符。并输出所识别词法单元的编码和单词符号。
2. 单行注释和多行注释。
3.  具有一定的错误分析能力，譬如显示错误，并跳过错误，继续分析。

要求代码中能体现关键字、各个不同词法单元的构造规则，或者能结合代码进行解释。

### 实验原理

词法分析是编译器工作的第一步。词法分析负责将用户的代码从左至右依次读入，识别出来其中的词素（lexeme），并将词素映射为token（role）。

flex通过读取一个*.l或者*.lex文件里的单词匹配规则生成C语言的实现代码。一个*.l的文件里的结构大概如下，用%%分隔开来。

```text
%{ /*declaration*/ %} 
/* Definition */
%%
/* Rules */ 
%%
/* C Code */
```

- **定义区**包含一些简单的**名字定义**(name definitions)来简化词法**扫描器**(scanner)的规则，并且还有**起始条件**(start condition)的定义。
- **规则区**包含了一系列具有**pattern-action**形式的规则，并且模式 pattern 位于行首不能缩进，action 也应该起始于同一行。
- **用户代码区** 的代码将被原封不动地拷贝到输出文件中，并且这些代码通常会被扫描器调用，当然，该区是可选的，如果 Flex 源文件中不存在该区，那么可以省略第二个 "%%" 。



### 实验例子

首先安装flex`sudo apt-get install flex`

统计输入中的字符数，单词数目，行数，并且打印每一个单词。

```c
%{
#define T_WORD 1
int numChars = 0, numWords = 0, numLines = 0;
%}

WORD            ([^ \t\n\r\a]+)

%%
\n              { numLines++; numChars++; }
{WORD}          { numWords++; numChars += yyleng; return T_WORD; }
<<EOF>>         { return 0; }
.               { numChars++; }
%%

int main() {
        int token_type;
        while (token_type = yylex()) {
                printf("WORD:\t%s\n", yytext);
        }
        printf("\nChars\tWords\tLines\n");
        printf("%d\t%d\t%d\n", numChars, numWords, numLines);
        return 0;
}

int yywrap() {
        return 1;
}
```

将此文件另存为 `word-spliter.l` 。注意此文件中的 **%{** 和 **%}** 必须在本行的最前面（前面不能有空格），同时，注意 **%}** 不要写成 **}%** 了。在终端输入：

```
$ flex word-spliter.l
$ gcc -o word-spliter lex.yy.c
$ ./word-spliter < word-spliter.l
```

将输出：

```
WORD:       %{
WORD:       #define
...
WORD:       }

Chars       Words   Lines
577          70       27
```

可见此程序其实就是一个原始的分词器，它将输入文件分割成一个个的 **WORD** 再输出到终端，同时统计输入文件中的字符数、单词数和行数。此处的 **WORD** 指一串连续的非空格字符。

下面，详细介绍 flex 输入文件的完整格式，同时解释一下本文件的代码。一个完整的 flex 输入文件的格式为：

```
%{
Declarations
%}
Definitions
%%
Rules
%%
User subroutines
```

输入文件的第 1 段 **%{** 和 **%}** 之间的为 **声明（Declarations）** ，都是 C 代码，这些代码会被原样的复制到 **lex.yy.c** 文件中，一般在这里声明一些全局变量和函数，这样在后面可以使用这些变量和函数。

第 2 段 **%} 和 %%** 之间的为 **定义（Definitions）**，在这里可以定义正则表达式中的一些名字，可以在 **规则（Rules）** 段被使用，如本文件中定义了 **WORD** 为 **([^** **\t\n\r\a]+)** ， 这样在后面可以用 **{WORD}** 代替这个正则表达式。

第 3 段为 **规则（Rules）** 段，上一个例子中已经详细说明过了。

第 4 段为 **用户定义过程（User subroutines）** 段，也都是 C 代码，本段内容会被原样复制到 **yylex.c** 文件的最末尾，一般在此定义第 1 段中声明的函数。

以上 4 段中，除了 **Rules** 段是必须要有的外，其他三个段都是可选的。

输入文件中最后一行的 **yywrap** 函数的作用是将多个输入文件打包成一个输入，当 **yylex** 函数读入到一个文件结束（EOF）时，它会向 yywrap 函数询问， yywrap 函数返回 1 的意思是告诉 **yylex** 函数后面没有其他输入文件了，此时 yylex 函数结束，yywrap 函数也可以打开下一个输入文件，再向 yylex 函数返回 0 ，告诉它后面还有别的输入文件，此时 yylex 函数会继续解析下一个输入文件。总之，由于我们不考虑连续解析多个文件，因此此处返回 1 。

和上一个例子不同的是，本例中的 action 中有 return 语句，而 **main** 函数内是一个 while 循环，只要 yylex 函数的返回值不为 0 ，则 yylex 函数将被继续调用，此时将从下一个字符开始新一轮的扫描。

另外，本例中使用到了 flex 提供的两个全局变量 **yytext** 和 **yyleng**，分别用来表示刚刚匹配到的字符串以及它的长度。



**仿照上面的流程更改下面的例子跑通程序基本就可以满足实验要求（PS:鼓励自己进一步扩展）**：

```c
%%

[\n]                { cur_line_num++;                       }
[ \t\r\a]+          { /* ignore all spaces */               }
{SINGLE_COMMENT1}   { /* skip for single line comment */    }
{SINGLE_COMMENT2}   { /* skip for single line commnet */    }

{OPERATOR}          { return yytext[0];         }

"<="                { return T_Le;              }
">="                { return T_Ge;              }
"=="                { return T_Eq;              }
"!="                { return T_Ne;              }
"&&"                { return T_And;             }
"||"                { return T_Or;              }
"void"              { return T_Void;            }
"int"               { return T_Int;             }
"while"             { return T_While;           }
"if"                { return T_If;              }
"else"              { return T_Else;            }
"return"            { return T_Return;          }
"break"             { return T_Break;           }
"continue"          { return T_Continue;        }
"print"             { return T_Print;           }
"readint"           { return T_ReadInt;         }

{INTEGER}           { return T_IntConstant;     }
{STRING}            { return T_StringConstant;  }
{IDENTIFIER}        { return T_Identifier;      }

<<EOF>>             { return 0; }
{UNTERM_STRING}     { lex_error("Unterminated string constant", cur_line_num);  }
.                   { lex_error("Unrecognized character", cur_line_num);        }

%%
```

### 相关链接

[词法分析](https://zhuanlan.zhihu.com/p/65490271) PS：知乎专栏对flex的原理解释的很清楚，我只是搬运工 :cowboy_hat_face: