Mako 模板是从文本流中进行解析的，流中可以包含任意内容: XML, HTML, email 文本，等等。模板中可以包含 Mako 特定的指令(directives)，可用于表示变量或表达式替换，控制结构（如条件和循环），服务器端注释，整段的 Python 代码，以及各种用于提供附加功能的标签(tags)。所有这些将被编译为真实的 Python 代码。这意味着你可以在 Mako 模板中利用 Python 几乎所有的强大特性。

## 表达式替换

最简单的表达式是变量替换。其语法是 ${}，受 Perl, Genshi, JSP EL 和其他一些语言的启发：
```
this is x: ${x}
```

以上代码中，x 的字符串表示形式将被应用到模板的输出流中。x 的值通常来源于你提供给模板 rendering 函数作为参数的 Context 变量。如果 x 的值未指定给模板，并且也未指定为局部变量，则会估算为一个特殊的值 UNDEFINED. 

${} 中的内容会被 Python 直接估算，所以各种表达式都是支持的：
```
pythagorean theorem:  ${pow(x,2) + pow(y,2)}
```

在 render 到输出流之前，表达式的结果总是被估算为一个字符串。

## 表达式转义

Mako 包含了一系列内建的转义机制，包括 HTML, URI 和 XML 转义，以及 "trim" 函数。转义的动作可以通过 | 运算符附加到表达式后面：
```
${"this is some text" | u}
```

上例中对表达式进行了 URL 转义，其结果是：this+is+some+text. 其中 u 代表 URL 转义，与之类似的，h 代表 HTML 转义，x 代表 XML 转义，而 trim 则是通常的 trim 函数。

可以在 [Filtering and Buffering](http://docs.makotemplates.org/en/latest/filtering.html) 中获取更多关于过滤器函数的内容，包括如何创建你自己的过滤器。

## 控制结构
控制结构指的是控制程序流程的那些语法 — 条件（如 if/else），循环（如 while 和 for），以及 try/except 之类。

在 Mako 中，控制结构用 % 后附加常规 Python 控制结构的写法，并用另一个 % 语法 "end<name>" 结束语句块。其中 "<name>" 是该表达式的关键字：
```
% if x==5:
    this is some output
% endif
```

% 可以出现在一行里的任何位置，只要其前面没有其他文本；缩进是无所谓的。Python 中的所有“冒号”表达式在这里都完全支持，包括 if/elif/else, while, for 甚至 def，尽管 Mako 有其内建的功能更强大的 def 标签。
```
% for a in ['one', 'two', 'three', 'four', 'five']:
    % if a[0] == 't':
     its two or three
    % elif a[0] == 'f':
    four/five
    % else:
    one
    %endif
% endfor
```

The % sign can also be “escaped”, if you actually want to emit a percent sign as the first non whitespace character on a line, by escaping it as in %%:
```
%% some text

    %% some more text
```

## 循环上下文

The loop context provides additional information about a loop while inside of a % for structure:
```
<ul>
% for a in ("one", "two", "three"):
    <li>Item ${loop.index}: ${a}</li>
% endfor
</ul>
```

See [The Loop Context](http://docs.makotemplates.org/en/latest/runtime.html#loop-context) for more information on this feature.

## 注释

注释有两种形式。单行注释在一行中以 ## 开头（前面可以有空白）：
```
## 这是一个注释.
...text ...
```

而多行注释则使用<%doc> ... 文本.. </%doc> 的语法：
```
<%doc>
    these are comments
    more comments
</%doc>
```

## 换行过滤器

反斜线 ("\") 字符放在任意一行的后面，会吃掉一个换行符。
```
here is a line that goes onto \
another line.
```

上述语法会产生下列文本：
```
here is a line that goes onto another line.
```

## Python 语句块

任意 python 语句块都可以用 <% %> 标签来定义：
```
this is a template
<%
    x = db.get_resource('foo')
    y = [z.element for z in x if x.frobnizzle==5]
%>
% for elem in y:
    element: ${y}
% endfor
```

在 <% %> 内定义的 python 代码，其整体的缩进量是不重要的，但是他们内部的语句之间必须有正确的缩进层次（因为这里没法使用 end），Mako 的编译器会调整 python 语句块的缩进层次与其周围的其他生成的 Python 代码相配合。

## 模块级的语句块

<% %> 的一个变体是 <%! %>，代表模块级别的代码块。其中的代码会在模板的模块级别执行，而不是在模板的 rendering 函数中。所以，这段代码不能访问模板的 context，仅在模板被加载到内存时被执行（有可能是相对于每应用程序仅执行一次，或者多次，这取决于运行时环境）。可以使用 <%! %> 来定义模板的导入语句：
```
<%!
    import mylib
    import re

    def filter(text):
        return re.sub(r'^@', '', text)
%>
```

在模板中，可以在任何位置，定义任意数目的 <%! %> 语句块；他们在编译产生的模块文件中出现的次序和定义时相同。

## 标签

Mako 还提供了标签。所有标签的语法都一样，类似于 XML 标签，不同之处在于其标签名称必须以 % 开头。标签的关闭可以用反斜杠的内联形式，或者独立的关闭标签：
```
<%include file="foo.txt"/>

<%def name="foo" buffered="True">
    this is a def
</%def>
```

每种标签都定义了一系列特定的属性。有些属性是必须的。并且，很多属性支持估算操作(evaluation)，也就是说你可以在属性文本中嵌入一个表达式（用 ${} 语法）。
```
<%include file="/foo/bar/${myfile}.txt"/>
```

属性是否支持运行时的估算，取决于标签的类型，以及该标签是如何编译成模板的。想知道某个标签的属性是否支持估算，最好的办法就是去试验它！如果不合法，词法器会告诉你。

下面是所有标签的一个简单介绍：
**<%page>**

定义了当前模板的总体特性，包括缓存参数，以及模板被调用时期待的参数列表（非必须）。
```
<%page args="x, y, z='default'"/>
```

定义缓存特性：
```
<%page cached="True" cache_type="memory"/>
```

关于 <%page> 使用的细节在 The "body()" method 和 Caching 中有深入描述。目前只有 <%page> 标签是模板唯一的，其他的会被胡略，这在将来的发布版本中也许会被改变，但现在请确认这一点。

**<%include>**

类似于其他模板语言的一个标签，%include 接受一个文件名称作为参数，调用被引用文件的输出结果。
```
<%include file="header.html"/>

    hello world

<%include file="footer.html"/>
```

**<%def>**

%def 标签用于定义包含一系列内容的一个 Python 函数，此函数在当前模板的其他某个地方被调用到：
```
<%def name="myfunc(x)">
    this is myfunc, x is ${x}
</%def>

${myfunc(7)}
```

%def 标签比 Python 的 def 要强大一些，因为 Mako 的编译器为 %def 提供了很多额外服务，比如可以导出 defs 为模板“方法”，自动传播当前的 context (原文：automatic propigation of the current Context)，缓冲/过滤/缓存 标志位，以及带有内容的 def 调用，这使得 defs 的包可以被以参数的形式，提供给其他的 def 调用（不像听起来那么困难）。可以在 Defs 中了解 %def 的详细内容。

**<%block>**

%block is a tag that is close to a %def, except executes itself immediately in its base-most scope, and can also be anonymous (i.e. with no name):
```
<%block filter="h">
    some <html> stuff.
</%block>
```

Inspired by Jinja2 blocks, named blocks offer a syntactically pleasing way to do inheritance:
```
<html>
    <body>
    <%block name="header">
        <h2><%block name="title"/></h2>
    </%block>
    ${self.body()}
    </body>
</html>
```

Blocks are introduced in [Using Blocks](http://docs.makotemplates.org/en/latest/defs.html#blocks) and further described in [Inheritance](http://docs.makotemplates.org/en/latest/inheritance.html).

**<%namespace>**

%namespace 是 Mako 中相对 Python 里 import 语句的等价物。它允许访问其他模板文件的所有 rendering 函数和元数据，纯文本的 python 模块，以及局部定义的函数“包”。
```
<%namespace file="functions.html" import="*"/>
```
%namespace 编译后产生的对象是 mako.runtime.Namespace 的一个实例，这是在模块中被用来引用模板特定的信息（如当前的 URI，继承结构以及其他一些东西）的一个中心结构，名称空间的详细描述在 [Namespaces](http://docs.makotemplates.org/en/latest/namespaces.html) 中。 

**<%inherit>**

允许模板可以在继承链中安排其自身的位置。这是其他很多模板语言都有的一个熟悉的概念。
```
<%inherit file="base.html"/>
```

当使用 %inherit 标签时，首先，控制权被转交给继承树最顶层的父模板，由它来决定如何配合继承自它的子模板来处理其中的调用区域(calling areas)的内容。Mako 在这方面提供了很多灵活性，包括动态继承(dynamic inheritance), 内容包装(content wrapping), 以及多态的方法调用(polymorphic method calls). 详见 [Inheritance](http://docs.makotemplates.org/en/latest/inheritance.html)。

**<%nsname:defname>**

Any user-defined “tag” can be created against a namespace by using a tag with a name of the form <%<namespacename>:<defname>>. The closed and open formats of such a tag are equivalent to an inline expression and the <%call> tag, respectively.
```
<%mynamespace:somedef param="some value">
    this is the body
</%mynamespace:somedef>
```

To create custom tags which accept a body, see [Calling a Def with Embedded Content and/or Other Defs](http://docs.makotemplates.org/en/latest/defs.html#defs-with-content).

New in version 0.2.3.


**<%call>**

call 标签用于调用 <%defs %> 标签，可传递额外的内嵌内容。详细描述在 [Calling a def with embedded content and/or other defs](http://docs.makotemplates.org/en/latest/defs.html#defs-with-content).

**<%doc>**

处理多行注释：
```
<%doc>
    these are comments
    more comments
</%doc>
```

Also the ## symbol as the first non-space characters on a line can be used for single line comments.

**<%text>**

该标签使得 Mako 的词法器对模板指令的常规解析动作停止，并以纯文本的形式返回其整个内容部分。该语法可以用来书写 Mako 的文档：
```
<%text filter="h">
    heres some fake mako ${syntax}
    <%def name="x()">${x}</%def>
</%text>
```

## 从模板中提前退出

Sometimes you want to stop processing a template or <%def> method in the middle and just use the text you’ve accumulated so far. This is accomplished by using return statement inside a Python block. It’s a good idea for the return statement to return an empty string, which prevents the Python default return value of None from being rendered by the template. This return value is for semantic purposes provided in templates via the STOP_RENDERING symbol:
```
% if not len(records):
    No records found.
    <% return STOP_RENDERING %>
% endif
```

Or perhaps:
```
<%
    if not len(records):
        return STOP_RENDERING
%>
```

In older versions of Mako, an empty string can be substituted for the STOP_RENDERING symbol:
```
<% return '' %>
```

New in version 1.0.2: - added the `STOP_RENDERING` symbol which serves as a semantic identifier for the empty string "" used by a Python return statement.
