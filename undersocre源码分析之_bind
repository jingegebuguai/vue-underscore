## 用法

    bind_.bind(function, object, *arguments) 
    
绑定函数 function 到对象 object 上, 也就是无论何时调用函数, 函数里的 this 都指向这个 object. 任意可选参数 arguments 可以传递给函数 function , 可以填充函数所需要的参数, 这也被称为 partial application。

    var func = function(greeting){ return greeting + ': ' + this.name };
    func = _.bind(func, {name: 'moe'}, 'hi');
    func();
    => 'hi: moe'
    
下面来看看源码是如何实现的：

    var FuncProto = Function.prototype, ArrayProto = Array.prototype
    var nativeBind = FuncProto.bind, slice = ArrayProto.slice ;

    var Ctor = function(){};
    _.bind = function(func, context) {
    var args, bound;
    if (nativeBind && func.bind === nativeBind) 
        return nativeBind.apply(func,     slice.call(arguments, 1));
    if (!_.isFunction(func)) 
        throw new TypeError('Bind must be called on a function');
    args = slice.call(arguments, 2);
    bound = function() {
        if (!(this instanceof bound)) 
            return func.apply(context, args.concat(slice.call(arguments)));
        Ctor.prototype = func.prototype;
        var self = new Ctor;
        Ctor.prototype = null;
        var result = func.apply(self, args.concat(slice.call(arguments)));
        if (_.isObject(result)) return result;  
            return self;
    };
        return bound;
    };
    

如果js基础薄弱的同学可能看这段源码会感到迷茫，我刚开始看的时候也十分迷茫，感觉这段代码很多都是多余的成分。下面就来仔细分析：

    
    var FuncProto = Function.prototype, ArrayProto = Array.prototype
    var nativeBind = FuncProto.bind, slice = ArrayProto.slice ;

上面这两行代码目的为了获取bind和slice函数原型变量，相信大家能都明白。

    //优先调用宿主环境提供的bind方法(换句话说就是如果支持原生bind)
    if (nativeBind && func.bind === nativeBind) 
        return nativeBind.apply(func,     slice.call(arguments, 1));
    
这两行得仔细分析一下

    nativeBind === Function.prototype.bind
    function func(){}
    console.log(func.bind === Function.prototype.bind) //true
    
我们可以发现函数func.bind === nativeBind, 既然有了原生bind支持，干嘛还要下面那么多代码呢，我想可能是以前版本js没有bind函数吧...

bind.apply(func, [].slice.call(arguments, 1))?这段代码如何理解？我们先来做个实验：

    function func(){
    	console.log(this.name)
    }
    var obj = {name: 'xxx'}
    var f1 = func.bind(obj)
    var f2 = Function.prototype.bind.apply(func, [obj])
    f1() // 'xxx'
    f2() // 'xxx'
    console.log(f2 === f1) //false
    console.log(f2.toString() === f1.toString()) //true
    
看到这里，相信大家应该都明白了吧，bind.apply(func, [].slice.call(arguments, 1))和func.bind(obj, arguments[1], arguments[2], ...)返回一个相同功能的函数。

    //如果func不是函数，则抛出异常    
    if (!_.isFunction(func)) 
        throw new TypeError('Bind must be called on a function');
    //取出_bind()函数后面的参数数组，如_bind(func, {name: 'xxx'}, 2, 3),则args = {0: 1, 1: 2},是一个类数组对象    
    args = slice.call(arguments, 2);
    //构建要生成的函数
    bound = function() {
        ...
    };
    //返回函数
    return bound;
    
上面这几行代码可以说是_bind源码里最简单的几行了，不再深入分析。

    if (!(this instanceof bound)) 
        return func.apply(context, args.concat(slice.call(arguments)));
    
上面代码中的this到底指向什么？主要问题是这个。this的使用有显式指向和隐式指向，细分为如下几种：

- 直接调用f(),this指向Global Object,在浏览器中为window对象
- 作为对象属性调用obj.f()，this为所调用的对象obj
- 作用构造函数调用new f，this为一个新构造的对象，其原型为f.prototype
- 最后是以call，apply方式调用，this为传入的参数

到这里我们再来做一个实验：

    function func(){
    	console.log(this instanceof func)
    }
    //this指向为Global
    func() //false
    var f = new func()
    //this为f对象，f.__proto__ === func.prototype
    f   //true
    
是不是这样更加清晰了。所以当我们直接以bound()调用时，就执行func.apply(context, args.concat(slice.call(arguments)))，这里上面已经讨论过了，就不再细说。主要就是将func的this绑定到context对象上。
    
    //原型式继承，可以用Object.create(func)
    Ctor.prototype = func.prototype;
    var self = new Ctor;
    Ctor.prototype = null;
    var result = func.apply(self, args.concat(slice.call(arguments)));
    if ((typeof result) === 'object') return result;
    return self;
    
上面这段代码是在以new bound()实例化后才会运行的代码，相当简单，要知道用了原型继承来继承func的原型属性。


