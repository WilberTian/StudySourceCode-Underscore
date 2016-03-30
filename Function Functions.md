

### executeBound

    // Determines whether to execute a function as a constructor
    // or a normal function with the provided arguments
    var executeBound = function(sourceFunc, boundFunc, context, callingContext, args) {
      if (!(callingContext instanceof boundFunc)) return sourceFunc.apply(context, args);
      var self = baseCreate(sourceFunc.prototype);
      var result = sourceFunc.apply(self, args);
      if (_.isObject(result)) return result;
      return self;
    };

executeBound函数是一个内部函数，被_.bind和_.partial函数使用，下面就看看这个函数的实现。

这里以bind为例，一般来说，当实现bind的功能的时候，会使用下面的实现：
    
    var bind = function(func, context) {
        var args = Array.prototype.slice.call(arguments, 2);
        
        return function(){
            return func.apply(context, args.concat(Array.prototype.slice.call(arguments)));
        };
    };    
    
上面的实现可以满足很多情况，但是由于bind返回的是一个函数，这个函数可以被当作**构造函数**使用。在这种情况下

在当作**构造函数**使用情况下，上面的bind实现就有问题了，因为this指向了绑定的对象，而不是new创建出来的对象：

    var Staff = function(name, age) {
        this.name = name;
        this.age = age;
    };  
    
    var will = {"name": "Will", "age": 29};
    
    var Func = bind(Staff, will);
    var wilber = new Func("Wilber", "30");
    
    console.log(will);
    // Object {name: "Wilber", age: "30"}
    console.log(wilber);
    // Object {}
    
为了解决上面的问题，bind的实现就相应的改变为：

    var bind = function(func,context) {
        var args = Array.prototype.slice.call(arguments, 2);
        
        var bound = function() {
            var arg = args.concat(Array.prototype.slice.call(arguments));
            
            return (function(that) {
                if(!(that instanceof bound)) {
                    return func.apply(context, arg);
                }
                
                // self是通过func创建的对象
                var self = Object.create(func.prototype);
                var result = func.apply(self, arg);
                // 如果result是一个对象，就返回；否则返回新建的实例self
                if (_.isObject(result)) return result;
                return self;
            })(this); // 把当前的执行上下文作为参数，如果使用new，那么this就是新建的对象
        };
        
        return bound;
    };    
    
通过对上面的逻辑可以看到，executeBound函数的实现就是基于上面的思路。

    
    
### bind

    // Create a function bound to a given object (assigning `this`, and arguments,
    // optionally). Delegates to **ECMAScript 5**'s native `Function.bind` if
    // available.
    _.bind = function(func, context) {
      // 如果原生的Function.prototype.bind可以使用就直接使用
      if (nativeBind && func.bind === nativeBind) return nativeBind.apply(func, slice.call(arguments, 1));
      // 如果func不是函数类型，直接报错
      if (!_.isFunction(func)) throw new TypeError('Bind must be called on a function');
      // 通过executeBound实现bind
      var args = slice.call(arguments, 2);
      var bound = function() {
        return executeBound(func, bound, context, this, args.concat(slice.call(arguments)));
      };
      return bound;
    };
    
bind函数的作用是用来绑定函数的执行上下文。

如果原生的Function.prototype.bind可以使用就直接使用，如果原生的方法不可用，就重新基于executeBound函数实现bind。

*依赖*：

- _.isFunction - Function Functions
- executeBound - Function Functions


    
### partial
   
    // Partially apply a function by creating a version that has had some of its
    // arguments pre-filled, without changing its dynamic `this` context. _ acts
    // as a placeholder, allowing any combination of arguments to be pre-filled.
    _.partial = function(func) {
      // 获取除了func之外的所有其他参数
      var boundArgs = slice.call(arguments, 1);
      var bound = function() {
        var position = 0, length = boundArgs.length;
        var args = Array(length);
        for (var i = 0; i < length; i++) {
          // 查找所有的占位符参数“_”
          args[i] = boundArgs[i] === _ ? arguments[position++] : boundArgs[i];
        }
        while (position < arguments.length) args.push(arguments[position++]);
        // 通过executeBound函数创建返回函数
        return executeBound(func, bound, this, this, args);
      };
      return bound;
    };
    
partial可以给一个函数填充人一个参数，然后返回一个新的函数。对于填充的参数，可以使用“_”作为一个参数的占位符。    
   
*依赖*：

- executeBound - Function Functions

   
   
### bindAll
    
    // Bind a number of an object's methods to that object. Remaining arguments
    // are the method names to be bound. Useful for ensuring that all callbacks
    // defined on an object belong to it.
    _.bindAll = function(obj) {
      var i, length = arguments.length, key;
      if (length <= 1) throw new Error('bindAll must be passed function names');
      for (i = 1; i < length; i++) {
        // 获取需要绑定的方法名
        key = arguments[i];
        // 通过obj[key]得到方法，然后进行绑定
        obj[key] = _.bind(obj[key], obj);
      }
      return obj;
    };
   
该函数的常用形式为`_.bindAll(object, *methodNames) `，作用是把methodNames参数指定的方法绑定到obj对象上，这样就改变了这些方法的执行上下文。
   
*依赖*：

- _.bind- Function Functions 
   
   
   
### memoize　
   
    // Memoize an expensive function by storing its results.
    _.memoize = function(func, hasher) {
      var memoize = function(key) {
        // 通过cache这个 对象缓存中间计算结果 
        var cache = memoize.cache;
        var address = '' + (hasher ? hasher.apply(this, arguments) : key);
        // 如果cache中已经有计算过的结果，就直接返回结果
        if (!_.has(cache, address)) cache[address] = func.apply(this, arguments);
        return cache[address];
      };
      memoize.cache = {};
      return memoize;
    };
    
该函数可以用来缓存某函数的计算结果，对于耗时较长的计算是很有帮助的。

*依赖*：

- _.has- Object Functions 

    
   
### delay 
   
    // Delays a function for the given number of milliseconds, and then calls
    // it with the arguments supplied.
    _.delay = function(func, wait) {
      var args = slice.call(arguments, 2);
      return setTimeout(function(){
        return func.apply(null, args);
      }, wait);
    };
    
该函数是对setTimeout的简单封装，等待wait毫秒后调用function。

    
   
### defer
   
    // Defers a function, scheduling it to run after the current call stack has
    // cleared.
    _.defer = _.partial(_.delay, _, 1);
   
该函数的作用是延迟调用function直到当前调用栈清空为止，对于执行开销大的计算和无阻塞UI线程的HTML渲染时候非常有用   
   
*依赖*：

- _.partial - Function Functions 
- _.delay - Function Functions 


   
### throttle & debounce
   
    // Returns a function, that, when invoked, will only be triggered at most once
    // during a given window of time. Normally, the throttled function will run
    // as much as it can, without ever going more than once per `wait` duration;
    // but if you'd like to disable the execution on the leading edge, pass
    // `{leading: false}`. To disable execution on the trailing edge, ditto.
    _.throttle = function(func, wait, options) {
      var context, args, result;
      var timeout = null;
      var previous = 0;
      if (!options) options = {};
      var later = function() {
        previous = options.leading === false ? 0 : _.now();
        timeout = null;
        result = func.apply(context, args);
        if (!timeout) context = args = null;
      };
      return function() {
        var now = _.now();
        if (!previous && options.leading === false) previous = now;
        var remaining = wait - (now - previous);
        context = this;
        args = arguments;
        if (remaining <= 0 || remaining > wait) {
          if (timeout) {
            clearTimeout(timeout);
            timeout = null;
          }
          previous = now;
          result = func.apply(context, args);
          if (!timeout) context = args = null;
        } else if (!timeout && options.trailing !== false) {
          timeout = setTimeout(later, remaining);
        }
        return result;
      };
    };
   
    // Returns a function, that, as long as it continues to be invoked, will not
    // be triggered. The function will be called after it stops being called for
    // N milliseconds. If `immediate` is passed, trigger the function on the
    // leading edge, instead of the trailing.
    _.debounce = function(func, wait, immediate) {
      var timeout, args, context, timestamp, result;
   
      var later = function() {
        var last = _.now() - timestamp;
   
        if (last < wait && last >= 0) {
          timeout = setTimeout(later, wait - last);
        } else {
          timeout = null;
          if (!immediate) {
            result = func.apply(context, args);
            if (!timeout) context = args = null;
          }
        }
      };
   
      return function() {
        context = this;
        args = arguments;
        timestamp = _.now();
        var callNow = immediate && !timeout;
        if (!timeout) timeout = setTimeout(later, wait);
        if (callNow) {
          result = func.apply(context, args);
          context = args = null;
        }
   
        return result;
      };
    };
    
    
#### 遇到的问题

在开发过程中会遇到频率很高的事件或者连续的事件，如果不进行性能的优化，就可能会出现页面卡顿的现象，比如：

1. 鼠标事件：mousemove(拖曳)/mouseover(划过)/mouseWheel(滚屏)
2. 键盘事件：keypress(基于ajax的用户名唯一性校验)/keyup(文本输入检验、自动完成)/keydown(游戏中的射击)
3. window的resize/scroll事件(DOM元素动态定位)

为了解决这类问题，常常使用的方法就是**throttle(节流)**和**debounce(去抖)**

#### 认识throttle和debounce

throttle和debounce的作用就是确认事件执行的方式和时机，举个简单的例子看看两者的区别：

- debounce：假设你正在乘电梯上楼，当电梯门关闭之前发现有人也要乘电梯，礼貌起见，你会按下开门开关，然后等他进电梯；如果在电梯门关闭之前，又有人来了，你会继续开门；这样一直进行下去，你可能需要等待几分钟，最终没人进电梯了，才会关闭电梯门，然后上楼。

  所以debounce的作用是，当调用动作触发一段时间后后，才会执行该动作，若在这段时间间隔内又调用此动作则将重新计算时间间隔

- throttle：假设你正在乘电梯上楼，当电梯门关闭之前发现有人也要乘电梯，礼貌起见，你会按下开门开关，然后等他进电梯；但是，你是个没耐心的人，你最多只会等待电梯停留一分钟，在这一分钟内，你会开门让别人进来，但是过了一分钟之后，你就会关门，让电梯上楼。

  所以throttle的作用是，预先设定一个执行周期，当调用动作的时刻大于等于执行周期则执行该动作，然后进入下一个新周期。

#### 使用场景    

下面给出了throttle和debounce的常用场景

**debounce**

Use it to discard a number of fast-pace events until the flow slows down

- When typing fast in a textarea that will be processed: you don't want to start to process the text until user stops typing.
- When saving data to the server via AJAX: You don't want to spam your server with dozens of calls per second.

**throttle**

Same use cases than debounce, but you want to warranty that there is at least some execution of the callbacks at certain interval

- If that user types really fast for 30 secs, maybe you want to process the input every 5 secs.
- It makes a huge performance difference to throttle handling scroll events. A simple mouse-wheel movement can trigger dozens of events in a second. 
    
    
   
   
### wrap
   
    // Returns the first function passed as an argument to the second,
    // allowing you to adjust arguments, run code before and after, and
    // conditionally execute the original function.
    _.wrap = function(func, wrapper) {
      return _.partial(wrapper, func);
    };
    
该函数将一个函数func封装到另一个函数wrapper里面, 并把函数func作为第一个参数传给wrapper。

这样可以让wrapper在func运行之前、后 执行代码。
   
*依赖*：

- _.partial - Function Functions

   
   
### negate
   
    // Returns a negated version of the passed-in predicate.
    _.negate = function(predicate) {
      return function() {
        return !predicate.apply(this, arguments);
      };
    };
   
该函数接受一个谓词判断条件，然后对该条件取反。   
   
   
   
### compose
   
    // Returns a function that is the composition of a list of functions, each
    // consuming the return value of the function that follows.
    _.compose = function() {
      // 获得所有的函数参数
      var args = arguments;
      var start = args.length - 1;
      return function() {
        var i = start;
        // 第一次调用，保存结果到result
        var result = args[start].apply(this, arguments);
        // 循环调用剩余的函数
        while (i--) result = args[i].call(this, result);
        return result;
      };
    };
    
该函数可以接受一组函数作为参数，然后返回函数集组合后的复合函数。

也就是一个函数执行完之后把返回的结果再作为参数赋给下一个函数来执行，以此类推。类似的，把函数 f()、g()、和 h() 组合起来可以得到复合函数 f(g(h()))。    
   
   

### after
   
    // Returns a function that will only be executed on and after the Nth call.
    _.after = function(times, func) {
      return function() {
        if (--times < 1) {
          return func.apply(this, arguments);
        }
      };
    };
    
该函数接受两个参数times和func，并返回一个函数。作用是当返回的函数运行times之后才真正执行func。

该函数在处理同组异步请求返回结果时，如果要确保同组里所有异步请求完成之后才执行这个函数，这将非常有用。

    var renderNotes = _.after(notes.length, render);
    _.each(notes, function(note) {
      note.asyncSave({success: renderNotes});
    });
    // renderNotes is run once, after all notes have saved.
    
    
    
### before
   
    // Returns a function that will only be executed up to (but not including) the Nth call.
    _.before = function(times, func) {
      var memo;
      return function() {
        if (--times > 0) {
          memo = func.apply(this, arguments);
        }
        // 在times小于等于1的时候，将func设置为null
        if (times <= 1) func = null;
        return memo;
      };
    };
    
创建一个函数，调用不超过times次。 

当count已经达到时，最后一个函数调用的结果将被记住并返回。    



### once
   
    // Returns a function that will be executed at most one time, no matter how
    // often you call it. Useful for lazy initialization.
    _.once = _.partial(_.before, 2);
    
该函数的作用是创建一个只能调用一次的函数。    
    
*依赖*：

- _.partial - Function Functions    
- _.before- Function Functions


