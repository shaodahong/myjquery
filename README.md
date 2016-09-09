# jQuery源码分析

## 前言
有时候我在想jQuery为什么可以直接$操作，可以拥有比原生js更便利的DOM操作，而且只要你想就可以直接链式操作下去

## 核心框架
### 揭开一万多行代码的jQuery核心代码：
```
(function(window, undefined) {
	function jQuery(selector){
		return new jQuery.fn.init(selector)
	}
	jQuery.fn = jQuery.prototype = {
		init: function () {

		}
	}
	jQuery.fn.init.prototype = jQuery.fn;
	window.jQuery = window.$ = jQuery;
})(window)
```
+ 闭包结构传参window
	- 闭包结构传入实参window，然后里面用形参接收
		+ 减少内部每次引用window的查询时间
		+ 方便压缩代码
+ 形参undefined
	- 因为ie低版本的浏览器可以给undefined赋值成功，所以为了保证undefined的纯洁给它一个形参的位置而没有实参，保证了它一定是undefined
+ jQuery传参selector
	- selector可以是一对标签，可以是id、类、后代、子代等等，可以是jQuery对象，
+ jQuery原型对象赋值
	- 方便扩展jQuery的原型方法
+ return 实例化原型方法init
	- 其实就是为了我们每次使用$不用new $();
	- 为什么jQuery要new自己的原型方法呢，因为不new自己的就要new其他的函数返回，那干嘛不自己利用自己
+ jQuery原型对象赋值给jQuery原型方法init的原型
	- 因为内部给jQuery原型每扩展一个方法init也会有该方法，是不是很酷炫，init有了那么$()出来的jQuery对象是不是也有啦
+ 给window暴露可利用成员jQuery,$
	- 给window暴露后那么全局都可以直接使用了jQuery和$了
	- 至于为什么有$,因为短啊，当然你也可以每次jQuery()来使用


## 御用选择器-Sizzle
+ Sizzle也是jQuery的根本，当然了你也单独使用Sizzle
+ 上面说过$(selector)的参数selector可以是id、类、后代、子代等等，可以是jQuery对象,那么咱们每次$一下就可以心想事成的得到我们想要的jQuery对象是怎么办到的呢，没错，就是因为Sizzle，Sizzle封装了获取各种dom对象的方法，并且会把他们包装成jQuery对象
+ 浏览器能力测试
	- Sizzle内部有个support对象，support对象存储着正则测试浏览器能力的结果
	- 对于有能力问题的选择器使用通用兼容方案解决（繁琐的判断代码）
+ 正则
	- 正则表达式在jQuery中使用的还是比较多的，正则的使用可以很大的提交我们对数据的处理效率
+ 判断
	- 判断是在init内部判断selector的类型，
		+ 列如可能是个html标签，那么直接create一个selector标签的DOM对象包装成jQuery对象return出去
		+ 列如可能是个id名、类名、标签名等等，那么直接通过Sizzle获取到DOM对象包装成jQuery对象return出去
+ 包装
	- 我已经说了很多次的包装了，没错，jQuery对象其实也是个伪数组，这也是它的设计巧妙之处，因为用数组存储数据方便我们去进行更多的数据处理，比如`$("div").css("color": "red")`,那么jQuery会自动帮我们隐式迭代、再给页面上所有div包含的文字颜色设置为red,简单粗暴的一行代码搞定简直是程序猿的最爱

## 对外扩展-extend
+ jQuery核心的结构处理完毕之后基本上就可以对外使用了，但是我们知道我们是可以基于jQuery来实现插件的，包括jQuery自己可扩展性也必须要求他要对外提供一个接口方便进行二次开发，所以有了extend方法
+ 简单的extend就是混入，列子:
```
function extend(obj) {
        var k;
        for(k in obj) {
            this[k] = obj[k];
        }
    }

    Baiya.extend = extend;
    Baiya.fn.extend = extend;
```
对静态方法的和实例方法的扩展都要有，比如each方法，可以$.each来使用，也可以是$("div").each来使用
+ 之后jQuery一些方法都是基于extend来扩展的，当然我们自己也可以基于jQuery扩展方法


## DOM操作
+ DOM操作也是jQuery的一大特点，因为它太好用了，包含了我们所能想到的所有使用场景，完善了增删查改常用的方法
+ jQuery获取和设置类的方法如html()/css()/val()等等这些传参是设置值不传参是获取值



##链式编程
+ jQuery是支持链式编程的，只要你想你就可以一行代码写完所有的功能，这是怎么做到的呢
+ 每一个改变原型链的方法都会把当前的this对象保存成他自己的属性，然后可以调用end方法找到上一级链从而方便我们可以进行链式操作



## 事件操作
+ jQuery的事件操作一般可以通过click类(mouseover/mouseleave等等)和on来使用，但是click类的实现是调用on的
+ on的实现是对原生的onclick类的处理，因为相同的原生的事件在同一个DOM对象上只能被绑定一次，如果再次绑定会覆盖掉上一次的，所以jQuery帮我们封装了事件的存储，把相同的事件分成一个数组存储在一个对象里面，然后对数组进行遍历，依次调用数组里存储的每个方法
+ on实现之后会把所有的事件处理字符串处理一下用on来改造一下，如下:
```
Baiya.each(("onclick,onmousedown,onmouseenter,onmouseleave," +
    "onmousemove,onmouseout,onmouseover,onmouseup,onfocus," +
    "onmousewheel,onkeydown,onkeypress,onkeyup,onblur").split(","),     function (i, v) {
        var event = v.slice(2);
        Baiya.fn[event] = function (callback) {
            return this.on(event, callback);
        }
    });
```

## 属性操作
+ jQuery也提供给了我们方便的属性操作，底层就是对原生方法的包装，处理兼容性问题，如果jQuery不对IE浏览器的兼容处理的话，那么它的代码量可能会缩一半，当然锅不能全部甩给IE，比如innerText方法火狐是不支持的，但是支持textContent方法，所以jQuery会尽可能的处理这种浏览器带来的差异

## 样式操作
+ 基本思想如上

## Ajax操作
+ Ajax可以说是前端的跨越性进步，毫不夸张的说如果没有Ajax的发展，那么今天的前端可能不叫前端，可能是美工……

+ Ajax是什么？
	- 在我的理解来看Ajax就是一个方法，这个方法遵循着http协议的规范，我们可以使用这个方法来向服务器请求少量的数据，有了数据之后我们就可以操作DOM来达到局部更新网页的目的，这是一个非常酷的事情
+ jQuery的Ajax是基于XMLHttpRequest的封装，当然了他也有兼容性问题，具体的封装见我之前的文章[简单的ajax封装](http://baiya.me/2016/07/24/%E7%AE%80%E5%8D%95%E7%9A%84ajax%E5%B0%81%E8%A3%85/)
+ 具体就是区别get和post请求的区别，get请求的传参是直接拼接在url结尾，而post请求需要在send()里面传递，并且post请求还要设置请求头setRequestHeader("content-type", "application/x-www-form-urlencoded")
+ 请求后对json或者text或者xml的数据进行处理就可以渲染到页面了

## 提到Ajax就不得不提到跨域了
+ 跨域简单的来说限制了非同源（ip/域名/端口/协议）的数据交互，当然这肯定是极好的，因为如果不限制那么你的网页别人也可以操作是不是很恐怖
+ 但是有些情况下我们需要调用别人的服务器数据，而且别人也愿意怎么办呢，程序员是很聪明的，html标签中img，script，link等一些带有src属性的标签是可以请求外部资源的，img和link得到的数据是不可用的，只有script标签请求的数据我们可以通过函数来接收，函数的参数传递可以是任何类型，所以创建一个函数，来接收，参数就是请求到的数据，而对方的数据也要用该函数来调用就可以实现跨域了
+ 简单封装jsonp实现
```
// url是请求的接口
// params是传递的参数
// fn是回调函数
function jsonp(url, params, fn){
			// cbName实现给url加上哈希，防止同一个地址请求出现缓存
            var cbName = `jsonp_${(Math.random() * Math.random()).toString().substr(2)}`;
            window[cbName] = function (data) {
                fn(data);
                // 获取数据后移除script标签
                window.document.body.removeChild(scriptElement);
            };

            // 组合最终请求的url地址
            var querystring = '';
            for (var key in params) {
                querystring += `${key}=${params[key]}&`;
            }
            // 告诉服务端我的回调叫什么
            querystring += `callback=${cbName}`;

            url = `${url}?${querystring}`;

            // 创建一个script标签，并将src设置为url地址
            var scriptElement = window.document.createElement('script');
            scriptElement.src = url;
            // appendChild(执行)
            window.document.body.appendChild(scriptElement);
        }
```

## Animate
+ 很抱歉的是jQuery的动画源码我并没有阅读，但是我自己封装了一个动画函数，之后的源码阅读会补上的

+ 封装的代码
```
// element设置动画的DOM对象
// attrs设置动画的属性	object
// fn是回调函数
function animate(element, attrs, fn) {
        //清除定时器
        if(element.timerId) {
            clearInterval(element.timerId);
        }
        element.timerId = setInterval(function () {
            //设置开关
            var stop = true;
            //遍历attrs对象，获取所有属性
            for(var k in attrs) {
                //获取样式属性 对应的目标值
                var target = parseInt(attrs[k]);
                var current = 0;
                var step = 0;
                //判断是否是要修改透明度的属性
                if(k === "opacity") {
                    current = parseFloat( getStyle(element, k)) * 100 || 0;
                    target = target * 100;
                    step = (target - current) / 10;
                    step = step > 0 ? Math.ceil(step) : Math.floor(step);
                    current += step;
                    element.style[k] = current / 100;
                    //兼容ie
                    element.style["filter"] = "alpha(opacity="+  current +")";
                }else if(k === "zIndex") {
                    element.style[k] = target;
                } else {
                    //获取任意样式属性的值，如果转换数字失败，返回为0
                    current = parseInt(getStyle(element, k)) || 0;
                    step = (target - current) / 10;
                    console.log("current:" + current + "  step:" + step);
                    step = step > 0 ? Math.ceil(step) : Math.floor(step);
                    current += step;
                    //设置任意样式属性的值
                    element.style[k] = current + "px";
                }
                if(step !== 0) {
                    //如果有一个属性的值没有到达target  ，设置为false
                    stop = false;
                }

            }
            //如果所有属性值都到达target  停止定时器
            if(stop) {
                clearInterval(element.timerId);
                //动画执行完毕  回调函数
                if(fn) {
                    fn();
                }
            }
        },30);
    }

    //获取计算后的样式的值
    function getStyle(element, attr) {
        //能力检测
        if(window.getComputedStyle) {
            return window.getComputedStyle(element, null)[attr];
        }else{
            return element.currentStyle[attr];
        }
    }
```

