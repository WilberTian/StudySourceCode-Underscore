


### OOP

    // OOP
    // ---------------
    // If Underscore is called as a function, it returns a wrapped object that
    // can be used OO-style. This wrapper holds altered versions of all the
    // underscore functions. Wrapped objects may be chained.
    
    // Helper function to continue chaining intermediate results.
    var result = function(instance, obj) {
      return instance._chain ? _(obj).chain() : obj;
    };
    
    // Add your own custom functions to the Underscore object.
    _.mixin = function(obj) {
      // 通过_.functions方法获取obj的所有属性方法
      _.each(_.functions(obj), function(name) {
        // 添加静态方法
        var func = _[name] = obj[name];
        // 通过原型添加实例方法
        _.prototype[name] = function() {
          var args = [this._wrapped];
          push.apply(args, arguments);
          return result(this, func.apply(_, args));
        };
      });
    };
    
    // Add all of the Underscore functions to the wrapper object.
    _.mixin(_);
  
在Underscore中实现了很多公开的静态方法，但是Underscore库同样提供了以面向对象的方式使用这些方法的能力，例如：

    _.map([1, 2, 3], function(n){ return n * 2; });
    _([1, 2, 3]).map(function(n){ return n * 2; });  
  
在Underscore中，当将`_`对象作为构造函数使用的时候，新建的实例中会通过`_wrapped`来保存obj。

    var _ = function(obj) {
      if (obj instanceof _) return obj;
      if (!(this instanceof _)) return new _(obj);
      this._wrapped = obj;
    };
    
同时，通过_.mixin方法，可以将一个对象的所有属性方法添加到`_`对象和`_`对象产生的实例上。

在Underscore中的`_.mixin(_);`语句就是将`_`对象的所有静态方法添加到了所有的`_`对象的实例上。
  
  
  
### chain
  
    // Add a "chain" function. Start chaining a wrapped Underscore object.
    _.chain = function(obj) {
      var instance = _(obj);
      instance._chain = true;
      return instance;
    };
    
通过_.chain方法可以支持链式调用，对于支持链式调用的`_`实例，都有`{_wrapped: ***, _chain: true}`这种形式。


    