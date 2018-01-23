# 用法

## 基本用法
   
Mako 是一个模板引擎，快速而强大。其语法类似于 Python，可以在模板内随时建立可复用的函数，灵活性比较高。让我想起来最初学习写 ASP 的感觉。

Mako 的主页地址：[http://www.makotemplates.org/docs/usage.html](http://www.makotemplates.org/docs/usage.html)

相比而言，Django 内建的模板引擎，为了维持所谓模板语法的纯粹性和简单性，更纯粹的满足 MVC 模式的规定，牺牲了很多灵活性，一些高级的功能不得不利用 tag 和 filter 来实现，其写法并不太方便。而我前一阵为此事苦恼过，曾经想如何在 Django 中借鉴一点 Karrigell 那样自由的 pih 方式，但自己时间太少最终又不了了之。

## 测试段落

因此，将 Mako 集成到 Django 中，以取代 Django 自带的模板引擎，也许是一个不错的办法。可以提高模板的灵活性和可操作性。

这一节介绍 Mako template 的 Python API. 如果你在 Pylons 之类的 web 框架中使用 Mako，那么集成 Mako API 的工作也许已经给你做好了。你可以直接跳到下一节，
