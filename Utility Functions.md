

### noConflict

    // Run Underscore.js in *noConflict* mode, returning the `_` variable to its
    // previous owner. Returns a reference to the Underscore object.
    _.noConflict = function() {
      root._ = previousUnderscore;
      return this;
    };
    
这是一种防止全局变量冲突的典型解决方式，`previousUnderscore`存放着旧的`_`值，`this`存放着当前使用的`_`值。



### identity

    // Keep the identity function around for default iteratees.
    _.identity = function(value) {
      return value;
    };
    
identity是一个可以获取自身值的函数。    
  
  
  
### constant

    // Predicate-generating functions. Often useful outside of Underscore.
    _.constant = function(value) {
      return function() {
        return value;
      };
    };
  
调用constant将得到一个函数，执行这个返回的函数将得到constant函数的参数


  
### noop
  
    _.noop = function(){};
  
noop仅仅代表一个空函数，在代码中需要使用空函数的时候可以直接使用noop。

另外，通过调用noop可以得到`undefined`。  

  
  
    // Run a function **n** times.
    _.times = function(n, iteratee, context) {
      var accum = Array(Math.max(0, n));
      iteratee = optimizeCb(iteratee, context, 1);
      for (var i = 0; i < n; i++) accum[i] = iteratee(i);
      return accum;
    };
  
  
  
### random
  
    // Return a random integer between min and max (inclusive).
    _.random = function(min, max) {
      if (max == null) {
        max = min;
        min = 0;
      }
      return min + Math.floor(Math.random() * (max - min + 1));
    };
  
这是一个产生随机数的函数，返回min和max之间的一个数。

如果只传一个参数，则返回0和该参数之间的随机数的。  
  
  
  
### iteratee
  
    _.iteratee = function(value, context) {
      return cb(value, context, Infinity);
    };
  
iteratee函数在Underscore中被广泛使用，该函数常常用来生成可应用到集合中每个元素的回调函数。

更详细的内容就需要看看cb函数的实现了，根据参数value值的不同，会得到不同功能的回调函数：

1. 如果value为空，返回一个能返回自身值的函数（参考`_.identity`的实现）
2. 如果value为函数，调用optimizeCb，由于argCount值为Infinity，最终会得到下面的函数：

        function() {
          return value.apply(context, arguments);
        };
      
3. 如果value为对象，返回一个是否匹配属性的函数
4. 其他情况，调用property，返回一个可以获取对象属性的函数

*依赖*：

- cb - 重要的内部函数  


  
### now
  
    // A (possibly faster) way to get the current timestamp as an integer.
    _.now = Date.now || function() {
      return new Date().getTime();
    };
  
now函数用来获得系统时间戳，如果Date.now不存在，就需要重新实现
  
  
  
### escape & unescape

     // List of HTML entities for escaping.
    var escapeMap = {
      '&': '&amp;',
      '<': '&lt;',
      '>': '&gt;',
      '"': '&quot;',
      "'": '&#x27;',
      '`': '&#x60;'
    };
    var unescapeMap = _.invert(escapeMap);
  
    // Functions for escaping and unescaping strings to/from HTML interpolation.
    var createEscaper = function(map) {
      var escaper = function(match) {
        return map[match];
      };
      // Regexes for identifying a key that needs to be escaped
      var source = '(?:' + _.keys(map).join('|') + ')';
      var testRegexp = RegExp(source);
      var replaceRegexp = RegExp(source, 'g');
      return function(string) {
        string = string == null ? '' : '' + string;
        return testRegexp.test(string) ? string.replace(replaceRegexp, escaper) : string;
      };
    };
    _.escape = createEscaper(escapeMap);
    _.unescape = createEscaper(unescapeMap);
  
  
  
  
    // If the value of the named `property` is a function then invoke it with the
    // `object` as context; otherwise, return it.
    _.result = function(object, property, fallback) {
      var value = object == null ? void 0 : object[property];
      if (value === void 0) {
        value = fallback;
      }
      return _.isFunction(value) ? value.call(object) : value;
    };
  
  
  
### uniqueId 
  
    // Generate a unique integer id (unique within the entire client session).
    // Useful for temporary DOM ids.
    var idCounter = 0;
    _.uniqueId = function(prefix) {
      var id = ++idCounter + '';
      return prefix ? prefix + id : id;
    };
    
在同一个client session中生存一个唯一的id，常被用来作为历史DOM节点的id。
    
  
    // By default, Underscore uses ERB-style template delimiters, change the
    // following template settings to use alternative delimiters.
    _.templateSettings = {
      evaluate    : /<%([\s\S]+?)%>/g,
      interpolate : /<%=([\s\S]+?)%>/g,
      escape      : /<%-([\s\S]+?)%>/g
    };
  
    // When customizing `templateSettings`, if you don't want to define an
    // interpolation, evaluation or escaping regex, we need one that is
    // guaranteed not to match.
    var noMatch = /(.)^/;
  
    // Certain characters need to be escaped so that they can be put into a
    // string literal.
    var escapes = {
      "'":      "'",
      '\\':     '\\',
      '\r':     'r',
      '\n':     'n',
      '\u2028': 'u2028',
      '\u2029': 'u2029'
    };
  
    var escaper = /\\|'|\r|\n|\u2028|\u2029/g;
  
    var escapeChar = function(match) {
      return '\\' + escapes[match];
    };
  
    // JavaScript micro-templating, similar to John Resig's implementation.
    // Underscore templating handles arbitrary delimiters, preserves whitespace,
    // and correctly escapes quotes within interpolated code.
    // NB: `oldSettings` only exists for backwards compatibility.
    _.template = function(text, settings, oldSettings) {
      if (!settings && oldSettings) settings = oldSettings;
      settings = _.defaults({}, settings, _.templateSettings);
  
      // Combine delimiters into one regular expression via alternation.
      var matcher = RegExp([
        (settings.escape || noMatch).source,
        (settings.interpolate || noMatch).source,
        (settings.evaluate || noMatch).source
      ].join('|') + '|$', 'g');
  
      // Compile the template source, escaping string literals appropriately.
      var index = 0;
      var source = "__p+='";
      text.replace(matcher, function(match, escape, interpolate, evaluate, offset) {
        source += text.slice(index, offset).replace(escaper, escapeChar);
        index = offset + match.length;
  
        if (escape) {
          source += "'+\n((__t=(" + escape + "))==null?'':_.escape(__t))+\n'";
        } else if (interpolate) {
          source += "'+\n((__t=(" + interpolate + "))==null?'':__t)+\n'";
        } else if (evaluate) {
          source += "';\n" + evaluate + "\n__p+='";
        }
  
        // Adobe VMs need the match returned to produce the correct offest.
        return match;
      });
      source += "';\n";
  
      // If a variable is not specified, place data values in local scope.
      if (!settings.variable) source = 'with(obj||{}){\n' + source + '}\n';
  
      source = "var __t,__p='',__j=Array.prototype.join," +
        "print=function(){__p+=__j.call(arguments,'');};\n" +
        source + 'return __p;\n';
  
      try {
        var render = new Function(settings.variable || 'obj', '_', source);
      } catch (e) {
        e.source = source;
        throw e;
      }
  
      var template = function(data) {
        return render.call(this, data, _);
      };
  
      // Provide the compiled source as a convenience for precompilation.
      var argument = settings.variable || 'obj';
      template.source = 'function(' + argument + '){\n' + source + '}';
  
      return template;
    };
    
    
    