# Mako Templates for Python{docsify-ignore}

Mako is a template library written in Python. It provides a familiar, non-XML syntax which compiles into Python modules for maximum performance. Mako's syntax and API borrows from the best ideas of many others, including Django and Jinja2 templates, Cheetah, Myghty, and Genshi. Conceptually, Mako is an embedded Python (i.e. Python Server Page) language, which refines the familiar ideas of componentized layout and inheritance to produce one of the most straightforward and flexible models available, while also maintaining close ties to Python calling and scoping semantics.

Mako is used by [reddit.com](https://www.reddit.com/) where it delivers over [one billion page views per month](http://mashable.com/2011/02/02/reddit-surpasses-1-billion-monthly-pageviews/#E1OPKBTwR5qP). It is the default template language included with the [Pylons and Pyramid](https://www.pylonsproject.org/) web frameworks.

**Nutshell:**

```
<%inherit file="base.html"/>
<%
    rows = [[v for v in range(0,10)] for row in range(0,10)]
%>
<table>
    % for row in rows:
        ${makerow(row)}
    % endfor
</table>

<%def name="makerow(row)">
    <tr>
    % for name in row:
        <td>${name}</td>\
    % endfor
    </tr>
</%def>
``` 

**Philosophy:**

Python is a great scripting language. Don't reinvent the wheel...your templates can handle it !

**Features:**

- Super-simple API. For basic usage, just one class, `Template` is needed: 

```
from mako.template import Template
print(Template("hello ${data}!").render(data="world"))
```

For filesystem management and template caching, add the `TemplateLookup` class. 

- Super fast. As templates are ultimately compiled into Python bytecode, Mako's approach is extremely efficient, and was originally written to be just as fast as Cheetah. Today, Mako is very close in speed to Jinja2, which uses a similar approach and for which Mako was an inspiration. 

- Standard template features 
  - control structures constructed from real Python code (i.e. loops, conditionals)
  - straight Python blocks, inline or at the module-level
  - plain old includes

- Callable blocks 
  - two types - the `<%def>` tag provides Python def semantics, whereas the `<%block>` tag behaves more like a Jinja2 content block.
  - can access variables from their enclosing scope as well as the template's request context
  - can be nested arbitrarily
  - can specify regular Python argument signatures
  - outer-level callable blocks can be called by other templates or controller code (i.e. "method call")
  - Calls to functions can define any number of sub-blocks of content which are accessible to the called function This is the basis for nestable custom tags.

- Inheritance
  - supports "multi-zoned" inheritance - define any number of areas in the base template to be overridden using `<%block>` or `<%def>`.
  - supports "chaining" style inheritance - call next.body() to call the "inner" content.
  - the full inheritance hierarchy is navigable in both directions (i.e. parent and child) from anywhere in the chain.
  - inheritance is dynamic! Specify a function instead of a filename to calculate inheritance on the fly for every request.

- Full-Featured
  - filters, such as URL escaping, HTML escaping. Markupsafe is used for performant and secure HTML escaping, and new filters can be constructed as a plain Python callable.
  - complete caching system, allowing caching to be applied at the page level or individual block/def level. The caching system includes an open API that communicates with [Beaker](http://beaker.groovie.org/) and soon [dogpile.cache](https://bitbucket.org/zzzeek/dogpile.cache/) by default, and new cache backends can be added with ease via setuptools entrypoints.
  - Supports Python 2.6 through modern 3 versions.
  - Supports Google App Engine.


To get started, visit the [documentation](http://www.makotemplates.org/docs/) and the [download page](http://www.makotemplates.org/download.html).

Mako is covered by the [MIT License](http://www.opensource.org/licenses/mit-license.php).
