## 基本用法

这一节介绍 Mako template 的 Python API. 如果你在 Pylons 之类的 web 框架中使用 Mako，那么集成 Mako API 的工作也许已经给你做好了。你可以直接跳到下一节，[语法](zh-cn/doc_syntax)。

最简单的办法是通过 Template 类，创建模板并渲染之：
```
from mako.template import Template

mytemplate = Template("hello world!")
print mytemplate.render()
```

上述例子中，传递给 Template 的文本参数被编译成了一个 python 的模块。该模块有一个 render_body() 函数，用于输出模板内容。当 mytemplate.render() 被调用时，Mako 会为此模板创建起一个运行时环境，并调用 render_body() 函数，然后捕获其输出到缓冲区，然后返回其字符串内容。

render_body() 函数中的代码可以访问包含了一些变量的一个名称空间。你可以传递额外的关键字参数给 render() 方法，这些参数将转化为可被访问的变量：
```
from mako.template import Template

mytemplate = Template("hello, ${name}!")
print mytemplate.render(name="jack")
```

template.render() 方法会让 Mako 创建一个 Context 对象，其中包含了所有该模板可访问的变量的名称，以及一个用于捕获输出的缓冲区。你也可以自己创建 Context 对象，并命令模板利用此 Context 来 render，用 render_context 方法即可：
```


from mako.template import Template
from mako.runtime import Context
from StringIO import StringIO

mytemplate = Template("hello, ${name}!")
buf = StringIO()
ctx = Context(buf, name="jack")
mytemplate.render_context(ctx)
print buf.getvalue()
```

## 使用基于文件的模板

你也可以从文件中加载 Template 的内容，使用 filename 关键字参数：你也可以从文件中加载 Template 的内容，使用 filename 关键字参数：
```
from mako.template import Template

mytemplate = Template(filename='/docs/mytmpl.txt')
print mytemplate.render()
```

为提高性能，从文件中加载的 Template, 可以将它产生的模块的源代码以普通 python 模块文件的形式(.py)，缓存到文件系统中。只要加一个参数 module_directory 即可做到这一点：
```
from mako.template import Template

mytemplate = Template(filename='/docs/mytmpl.txt', module_directory='/tmp/mako_modules')
print mytemplate.render()
```

当上述代码被 render 的时候，会创建文件 `/tmp/mako_modules/docs/mytmpl.txt.py`. 下一次 [Template](http://docs.makotemplates.org/en/latest/usage.html#mako.template.Template) 对象被用同样参数调用的时候，就会直接重用该模块文件。

## 使用 TemplateLookup

在模板中，我们有时候需要调用或引用其他模板的内容，这就牵涉一个模板查找定位的问题，通常用简单的 URI 字符串来定位。我们用 TemplateLookup 类来负责这个任务。该类的构造函数需要传递一系列可供查找模板的路径的列表。然后我们再将此 TemplateLookup 对象用关键字参数的形式传递给 Template 对象。
```
from mako.template import Template
from mako.lookup import TemplateLookup

mylookup = TemplateLookup(directories=['/docs'])
mytemplate = Template("""<%include file="header.txt"/> hello world!""", lookup=mylookup)
```

上述例子中创建了一个文本模板，其中有一个对 header.txt 文件的包含引用。而从何处去查找 header.txt, 则由 TemplateLookup 指明，是 "/docs" 目录。

通常，应用程序会把模板用文本文件的形式保存在文件系统中。而为了方便起见，我们可以直接通过 TemplateLookup 来获取模板对象，利用 TemplateLookup 的 get_template 方法，并传递模板的 URI 作为参数：
```
from mako.template import Template
from mako.lookup import TemplateLookup

mylookup = TemplateLookup(directories=['/docs'], module_directory='/tmp/mako_modules')

def serve_template(templatename, **kwargs):
    mytemplate = mylookup.get_template(templatename)
    print(mytemplate.render(**kwargs))
```

In the example above, we create a TemplateLookup which will look for templates in the `/docs` directory, and will store generated module files in the `/tmp/mako_modules` directory. The lookup locates templates by appending the given URI to each of its search directories; so if you gave it a URI of `/etc/beans/info.txt`, it would search for the file `/docs/etc/beans/info.txt`, else raise a `TopLevelNotFound` exception, which is a custom Mako exception.

当 lookup 找到模板时，它还会给 Template 指定一个 uri 属性，这个 uri 就是传递给 get_template() 方法的参数。Template 可以用此 uri 来计算出其对应的模块文件的名称。比如在上述例子中，`/etc/beans/info.txt` 这个 URI 名称参数，会导致创建模块文件 `/tmp/mako_modules/etc/beans/info.txt.py`.

## 设定集合的大小

TemplateLookup 同时也会在内存中缓存一组模板，所以并不是每一次请求都会导致模板的重新编译和模块重新加载。默认 TemplateLookup 的大小没有限制，但你可以通过 collection_size 参数来限制它：
```
mylookup = TemplateLookup(directories=['/docs'],
                module_directory='/tmp/mako_modules', collection_size=500)
```

The above lookup will continue to load templates into memory until it reaches a count of around 500. At that point, it will clean out a certain percentage of templates using a least recently used scheme.

以上的 lookup 会持续加载模板到内存中，直到达到 500 的时候，它就会清除掉一定比例的模板缓存项，根据“最近最少访问”原则。

## 设置文件系统检查

另一个 TemplateLookup 相关的标志是  filesystem_checks. 默认为 True, 每一次 get_template() 方法返回模板后，原始的模板文件的 revision time 会和上次加载模板的时间做对比，如果文件更新，则会加载其内容，并重新编译该模板。在生产环境下，设置 filesystem_checks 为 False 可以带来一定的性能提升（和具体的文件系统有关）。

## 使用 Unicode 和 Encoding

Template 和 TemplateLookup 都可以接受 output_encoding 和 encoding_errors 参数，用来对输出以 Python 支持的任何方式进行编码： 
```
from mako.template import Template
from mako.lookup import TemplateLookup

mylookup = TemplateLookup(directories=['/docs'], output_encoding='utf-8', encoding_errors='replace')

mytemplate = mylookup.get_template("foo.txt")
print(mytemplate.render())
```

When using Python 3, the render() method will return a bytes object, if output_encoding is set. Otherwise it returns a string.

Additionally, the render_unicode() method exists which will return the template output as a Python unicode object, or in Python 3 a string:
```
print(mytemplate.render_unicode())
```

The above method disregards the output encoding keyword argument; you can encode yourself by saying:
```
print(mytemplate.render_unicode().encode('utf-8', 'replace'))
```

Note that Mako’s ability to return data in any encoding and/or unicode implies that the underlying output stream of the template is a Python unicode object. This behavior is described fully in The [Unicode Chapter](http://docs.makotemplates.org/en/latest/unicode.html).

## 处理异常

模板异常可能在两种截然不同的地方出现。第一种是当你查找，分析和编译模板时，另一种是当你运行模板的时候。在模板的运行过程中，异常通常从产生问题的 python 代码出抛出。Mako 有其独立的一套异常类，它们大多数针对模板构造过程的查找和词法分析/编译阶段。Mako 还提供了一些库函数，这些函数用来帮助提供 Mako 相关的异常栈跟踪信息，并可用纯文本或 HTML 方式格式化异常信息。不管是哪种情况，这些处理函数的作用在于将 Python 文件名，行号，以及代码例子转换为 Mako 的模板文件名，行号，以及代码范例。在跟踪栈里对应于某个 Moko 模板的每一行，都回被转换为和源模板文件相关的。

为了格式化异常跟踪信息，系统提供了 text_error_template 和 html_error_template 函数。它们都利用了 sys.exc_info() 函数来获取最近抛出的异常信息。下面是常见的用法：
```
from mako import exceptions

try:
    template = lookup.get_template(uri)
    print(template.render())
except:
    print(exceptions.text_error_template().render())
```

如果使用 HTML 的输出函数： 
```
from mako import exceptions

try:
    template = lookup.get_template(uri)
    print(template.render())
except:
    print(exceptions.html_error_template().render())
```

The html_error_template() template accepts two options: specifying full=False causes only a section of an HTML document to be rendered. Specifying css=False will disable the default stylesheet from being rendered.

E.g.:
```
print(exceptions.html_error_template().render(full=False))
```

HTML 输出函数也内建到了 Template 中。通过 format_exceptions 这个标志位参数。这样，任何在 template 的 render 阶段引发的异常，都会使得 template 的输出内容被 html_error_template 方法的输出所替代:
```
template = Template(filename="/foo/bar", format_exceptions=True)
print(template.render())
```

注意，上述模板的编译阶段发生在你构造 Template 对象本身的时候，并且没有定义输出流。所以，在查找/解析/编译阶段引发的异常不会被处理，而是像平常那样被继续抛出到更高层次的调用堆栈上（繁殖, propagate）。虽然 pre-render traceback 不会包含任何 Mako 特定的行，这意味着发生在 rendering 之前的异常，以及 rendering 过程中发生的异常，需要用不同的办法分别处理。因此，上面的 try/except 模式可能是比较通用的一种写法。

被错误模板函数使用的内部对象是 RichTraceback. 该对象也可以被直接用于提供自定义错误视图。下面是一个范例应用，可以描述其一般使用的 API:
```
from mako.exceptions import RichTraceback

try:
    template = lookup.get_template(uri)
    print(template.render())
except:
    traceback = RichTraceback()
    for (filename, lineno, function, line) in traceback.traceback:
        print("File %s, line %s, in %s" % (filename, lineno, function))
        print(line, "\n")
    print("%s: %s" % (str(traceback.error.__class__.__name__), traceback.error))
```

## 常见框架的集成

Mako 的发布包包含了一些帮助代码，用于在其他流行的 web 框架中使用 Mako 的场景。下面是其概述。

### WSGI
在 examples/wsgi/run_wsgi.py 中，包含了一个 WSGI 程序的例子。该程序的目的是从 templates 以及 htdocs 目录中提取文件，并包含了一个初步的双文件布局。 WSGI 运行程序担任了一个功能齐全的 web 服务器的角色，使用 wsgiutils 来运行它自身，并把 GET 和 POST 参数信息从 request 中传递到 Context. 它可以服务图片，css 文件以及其它类型的文件。并可以使用 Mako 内建的异常处理函数来显示错误。

### Pygments
A Pygments-compatible syntax highlighting module is included under mako.ext.pygmentplugin. This module is used in the generation of Mako documentation and also contains various setuptools entry points under the heading pygments.lexers, including mako, html+mako, xml+mako (see the setup.py file for all the entry points).

### Babel
Mako provides support for extracting gettext messages from templates via a Babel extractor entry point under mako.ext.babelplugin.

Gettext messages are extracted from all Python code sections, including those of control lines and expressions embedded in tags.

Translator comments may also be extracted from Mako templates when a comment tag is specified to Babel (such as with the -c option).

For example, a project "myproj" contains the following Mako template at myproj/myproj/templates/name.html:
```
<div id="name">
  Name:
  ## TRANSLATORS: This is a proper name. See the gettext
  ## manual, section Names.
  ${_('Francois Pinard')}
</div>
```

To extract gettext messages from this template the project needs a Mako section in its Babel Extraction Method Mapping file (typically located at myproj/babel.cfg):
```
# Extraction from Python source files

[python: myproj/**.py]

# Extraction from Mako templates

[mako: myproj/templates/**.html]
input_encoding = utf-8
```

The Mako extractor supports an optional input_encoding parameter specifying the encoding of the templates (identical to Template/TemplateLookup’s input_encoding parameter).

Invoking Babel’s extractor at the command line in the project’s root directory:
```
myproj$ pybabel extract -F babel.cfg -c "TRANSLATORS:" .
```

will output a gettext catalog to stdout including the following:
```
#. TRANSLATORS: This is a proper name. See the gettext
#. manual, section Names.
#: myproj/templates/name.html:5
msgid "Francois Pinard"
msgstr ""
```

This is only a basic example: Babel can be invoked from setup.py and its command line options specified in the accompanying setup.cfg via Babel Distutils/Setuptools Integration.

Comments must immediately precede a gettext message to be extracted. In the following case the TRANSLATORS: comment would not have been extracted:
```
<div id="name">
  ## TRANSLATORS: This is a proper name. See the gettext
  ## manual, section Names.
  Name: ${_('Francois Pinard')}
</div>
```

See the [Babel User Guide](http://babel.edgewall.org/wiki/Documentation/index.html) for more information.

## API 参考
