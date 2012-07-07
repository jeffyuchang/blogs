--- 
layout: post
title: JavaScript中的函数对象
published: true
tags: 
- Chinese
- Javascript
- Programming Languages
type: post
status: publish
--- 
很多时候,我总是把Javascript当做一个脚本,没把它当做一个语言来看待.我记得是从04年开始因为需要使用XMLHttpRequest来进行局部刷新(那时候Ajax名词还没产生),我开始比较多的接触Javascript,但是最多也只是局限于用它来操作(D)HTML中的对象,也就是DOM.

再后来,在我印象中,我们试图把Javascript封装成类似面向对象的语言,比如我们去尝试着增加继承等的概念,再到后来,随着前端无数框架的出现,
轻量级的有prototype,jquery,重量级别的有dojo等,再到现在后端新秀,比如node.js.随着HTML5的推行,浏览器的强大,其实浏览器本身就是一个跨平台的
客户端,而javascript就是这个平台上的语言.

大力推荐Douglas Crockford著作的 JavaScript: The Good Parts, 这本书很简洁,简洁的有时候让你读完之后,觉得原来还可以这样,但是过一会儿,你不用,就又给忘了.而且这本书是从编程语言的角度来解释JavaScript这个语言,不是去罗列一堆已有的对象和方法等.在这篇博客中,我想重点介绍下JavaScript中的函数对象.

##### Function Object (函数对象)定义 #####
Function在Javascript中也是对象,简单的理解,就是Function里面它也可以有一系列的 name -> value 组合.我之前很容易把Function跟Java中的Method联系起来,但是其实,这两个在这边是完全不同的. 先看下面的创建一个函数:

{% highlight javascript %}
function add(a, b) {
	return a+b;
}
{% endhighlight %}

我们可以把一个函数赋给一个变量,如下:

{% highlight javascript %}
var add = function add(a, b) {
	return a+b;
};
{% endhighlight %}

又或者如下创建一个匿名函数然后赋给一个变量:

{% highlight javascript %}
function (a, b) {
	return a+b;
}
{% endhighlight %}

因为Function是一个对象,所以我们可以在Function里面创建一个Function,比如:

{% highlight javascript %}
var anotherMath = function anotherMath(a, b) {
	var sum = function() {
		return a+b;
	}
	return sum();
};
anotherMath(5, 3); //结果是8.
{% endhighlight %}

作为一个对象,Function也可以被作为参数被传递,又或者是作为一个返回值被返回,我想这可能刚看上去觉得比较不可思议，特别是当你把Function和Method等同看待的话. 但是如果你知道他是一个对象的话,就很正常,因为我们经常可以是传入一个对象,或者是返回一个对象.在Functional Programming中,我们也把这种特性叫做: [High Order Function](http://en.wikipedia.org/wiki/Higher-order_function).

##### Function Object (函数对象)作为参数 #####

先看一个把函数对象作为参数的例子.

{% highlight javascript %}
var getBiggerValue = function getBiggerValue(a, b, compareValue) {
	if (compareValue(a,b) > 0) {
		return a;
	} else {
		return b;
	}
};

getBiggerValue(12, 3, function (x, y){
	return x - y;	
});
{% endhighlight %}

这里我们可以看到,我们在调用getBiggerValue的时候,传入一个匿名函数作为参数. 下面再来看一个从[What is a Closure](http://nathansjslessons.appspot.com/lesson?id=1040)网站的一个例子.

{% highlight javascript %}
var addOne = function addOne(x) {
	return x+1;
};

var callTwice = function callTwice(f, x) {
	return f(f(x));
};

callTwice(addOne, 5) //结果是7.
{% endhighlight %}

在JavaScript中,我们大量应用这种方式来构建回调(callback)方法.

##### Function Object (函数对象)作为返回值 #####

接下来,我们来看个把函数对象作为返回值的例子.

{% highlight javascript %}
var anotherMath = function anotherMath(a, b) {
	var sum = function() {
		return a+b;
	}
	return sum;
};
anotherMath(3, 5)(); //结果是8,注意,因为最后的括号不可缺少,因为返回的是函数,你需要去执行它才能获得最后的值.
{% endhighlight %}
再来看一个简单的例子:
{% highlight javascript %}
var makeFunction = function () {
    var addOne = function (x) {
        return x + 1;
    };
    return addOne; // 返回一个函数
};
var f = makeFunction();//返回的是一个函数.
var y = f(3); //最后结果是4.
{% endhighlight %}
注意到我们第二个例子中,第一个函数不需要参数,返回的函数才需要参数.

##### Function Object (函数对象)的调用模式 #####
Function对象和普通的对象有一个不同,那就是Function对象可以被执行,或者叫调用.
在调用函数的时候,除了传进去函数本身所定义的参数外,还传递了 *this* 和 *arguments* 这两个参数, *this*所赋予的值是根据调用模式而定;
*arguments*:是调用这个函数时所传递的参数列表.值得注意的是,它不是严格意义的数组，但是它有length的属性,以及你可以用下标去访问元素.

函数的调用模式有以下四种:

###### 1. 方法调用模式(The Method Invocation Pattern) ######
当一个函数被作为一个对象的属性时,这个函数的调用方式就叫做方法调用模式.
{% highlight javascript %}
var theObject = {
	value: 0,
	increment: function(inc) {
		this.value += inc;
	}	
};
theObject.increment();
{% endhighlight %}
这里,我们定义一个对象叫做theObject,有个value的属性和increment方法.在运行这个increment函数时,我们是当做theObject的属性运行,这种情况属于方法调用模式,*this*的值就是当前这个对象,在这里就是theObject.
###### 2. 函数调用模式 (The Function Invocation Pattern) ######
当一个函数并不是作为一个对象的属性时,被调用时我们称这种为函数调用模式.这个时候*this*的值是Global Object,可以看做成一个最外面的全局对象.
{% highlight javascript %}
var sum = add(3, 4);
{% endhighlight %}

###### 3. 构造器调用模式(The Constructor Invocation Pattern) ######
{% highlight javascript %}
var Quo = function (string) {
	this.status = string;
};
var myQuo = new Quo("confused");
{% endhighlight %}
这种方式,用new来初始化一个函数的称为构造器调用模式.

###### 4. 应用调用模式 (The Apply Invocation Pattern) ######
{% highlight javascript %}
var theArray = [3, 4];
var sum = add.apply(null, theArray);
{% endhighlight %}
这里add函数是我们上面所定义的,apply方法有两个参数,第一个是赋值给*this*,第二个是参数数组.

##### 闭包 (Closure) #####
在创建函数的时候,我们说过可以在函数里面创建内嵌函数.在内嵌函数中,我们可以访问到外围函数的变量，但是不能访问到外围函数的*this*和*arguments*.
{% highlight javascript %}
var makeCounter = function () {
    var count, f;
    count = 0;
    f = function () {
        count = count + 1;
        return count;
    };
    return f;
};

var counter = makeCounter();

var a = counter(); // gets 1
var b = counter(); // gets 2
var c = counter(); // gets 3
{% endhighlight %}
当我们创建一个counter的时候,它返回一个内嵌函数f,这个内嵌函数跟之前的内嵌函数不同的是,这个内嵌函数它可以操作外面函数的count属性.
通常而言,当我们调用makeCounter()函数后返回,它的count属性随着function()函数生命周期就应该结束,但是因为内嵌函数f继续使用count,所以这个变量现在继续保存在f函数中,我们叫这样的函数叫闭包.
我们可以通过这种闭包的方式,对变量count进行封装,因为你只能通过返回函数里面的方法对count进行操作.

##### Curry Function #####
Curry Function说简单点,就是通过对一个函数进行调用curry方法,然后传入部分参数,返回的函数是对之前传入参数赋值完后的原函数.
比方说,add函数有2个参数,我们可以先传1个参数,然后获得的返回函数只能再接受另外一个函数.千言万语不如一段代码清晰.
{% highlight javascript %}
var add1 = add.curry(5);
var sum = add1(3); //结果是8
{% endhighlight %}
Curry函数的内部实现其实就是用我们上面说讲的闭包.把之前通过curry方法赋的值存在闭包的变量中.

##### Continuation #####
我们也可以通过把函数作为参数传入,来达成Continuation,也就是,一个函数执行完后,直接去执行另外一个函数,这样一直下去,这样的函数没有返回值.
比如以下的代码:
{% highlight javascript %}
// regular
var addOne = function (x) {
    return x + 1;
};

// CPS
var addOneCPS = function (x, ret) {
    ret(x + 1);
}; //这里我们传入一个ret函数,所以当我们调用addOneCPS时候,它就会接着调用ret函数,构成一种持续执行的效果.
{% endhighlight %}

##### 模块化 (Module) #####
在JavaScript中, 有个众所周知的问题,那就是缺少名字空间.比如说我们外面定义的变量,或者说我们定义的函数,它都是默认给Global Object中的变量和函数,这样如果每个javascript都这样的话,很容易重名.
那么解决这个问题的有效问题就是模块化利用我们上面所讲的采用函数,闭包方式来进行模块封装.
{% highlight javascript %}
var serial_maker = function(){
	var prefix = '';
	var seq = 0;
	return {
		set_prefix: function(p){
			prefix = p;
		},
		set_seq: function(s) {
			seq = s;
		},
		gensym: function(){
			var result = prefix + seq;
			seq += 1;
			return result;
		}
	};	
};

var seqer = serial_maker();
{% endhighlight %}
上面代码中,我们可以看出我们对prefix和seq变量进行了封装,它只能通过对外公开的两个方法set_prefix和set_seq对其操作.
注:我们最外面匿名函数返回的是一个对象,对象里面包括了三个属性,set_prefix,set_seq和gensym,这三个属性分别对应三个函数.

如果说在上面代码中,我们或许会给你一种serial_maker函数类似类(class)的感觉,我们可以简化一下,直接通过调用函数返回一个对象.
{% highlight javascript %}
var seqer = (function(){
	var prefix = '';
	var seq = 0;
	return {
		set_prefix: function(p){
			prefix = p;
		},
		set_seq: function(s) {
			seq = s;
		},
		gensym: function(){
			var result = prefix + seq;
			seq += 1;
			return result;
		}
	};	
}());
{% endhighlight %}
注意到最后一行的括弧,这就表示马上调用我们定义的匿名函数来返回一个对象,把这个对象赋值给seqer.

所以在你看似乎不起眼的JavaScript,他其实对函数编程的支持还是不错的.

#### 参考资料: ####
1. [Javascript: The Good Parts](http://shop.oreilly.com/product/9780596517748.do) by Douglas Crockford.
2. [Javascript in the small](http://olabini.com/blog/2011/10/javascript-in-the-small/) by Ola bini.
3. [What is a Closure](http://nathansjslessons.appspot.com/lesson?id=1040) by Nathan Whitehead.
