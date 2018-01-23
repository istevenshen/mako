======
语法
======

A Mako template is parsed from a text stream containing any kind
of content, XML, HTML, email text, etc. The template can further
contain Mako-specific directives which represent variable and/or
expression substitutions, control structures (i.e. conditionals
and loops), server-side comments, full blocks of Python code, as
well as various tags that offer additional functionality. All of
these constructs compile into real Python code. This means that
you can leverage the full power of Python in almost every aspect
of a Mako template.

Expression Substitution
=======================

The simplest expression is just a variable substitution. The
syntax for this is the ``${}`` construct, which is inspired by
Perl, Genshi, JSP EL, and others:


    this is x: ${x}

Above, the string representation of ``x`` is applied to the
template's output stream. If you're wondering where ``x`` comes
from, it's usually from the :class:`.Context` supplied to the
template's rendering function. If ``x`` was not supplied to the
template and was not otherwise assigned locally, it evaluates to
a special value ``UNDEFINED``. More on that later.

The contents within the ``${}`` tag are evaluated by Python
directly, so full expressions are OK:


    pythagorean theorem:  ${pow(x,2) + pow(y,2)}

The results of the expression are evaluated into a string result
in all cases before being rendered to the output stream, such as
the above example where the expression produces a numeric
result.

Expression Escaping
===================

Mako includes a number of built-in escaping mechanisms,
including HTML, URI and XML escaping, as well as a "trim"
function. These escapes can be added to an expression
substitution using the ``|`` operator:


    ${"this is some text" | u}

The above expression applies URL escaping to the expression, and
produces ``this+is+some+text``. The ``u`` name indicates URL
escaping, whereas ``h`` represents HTML escaping, ``x``
represents XML escaping, and ``trim`` applies a trim function.

Read more about built-in filtering functions, including how to
make your own filter functions, in [Syntax](Usage.md).

Control Structures
==================

