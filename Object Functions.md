

### collectNonEnumProps

    // Keys in IE < 9 that won't be iterated by `for key in ...` and thus missed.
    // 判断'toString'是否是可枚举的
    var hasEnumBug = !{toString: null}.propertyIsEnumerable('toString');
    var nonEnumerableProps = ['valueOf', 'isPrototypeOf', 'toString',
                        'propertyIsEnumerable', 'hasOwnProperty', 'toLocaleString'];
    
    // 给keys添加本该枚举的属性
    function collectNonEnumProps(obj, keys) {
      var nonEnumIdx = nonEnumerableProps.length;
      // 获取对象的constructor
      var constructor = obj.constructor;
      // 获取对象的原型，如果constructor是个函数，直接通过constructor.prototype获得原型
      var proto = (_.isFunction(constructor) && constructor.prototype) || ObjProto;
    
      // 对Constructor进行特殊处理，obj[prop] !== proto[prop]会失败
      var prop = 'constructor';
      // 如果对象有constructor属性并且keys数组中也有，添加进来
      if (_.has(obj, prop) && !_.contains(keys, prop)) keys.push(prop);
    
      while (nonEnumIdx--) {
        prop = nonEnumerableProps[nonEnumIdx];
        // 如果toString在obj中，并且obj的toString并不是原型的toString，并且keys里面没有，添加进来
        if (prop in obj && obj[prop] !== proto[prop] && !_.contains(keys, prop)) {
          keys.push(prop);
        }
      }
    }

ECMAScript 5中定义了对象属性的特性：

1. [[Configurable]]
2. [[Enumerable]]
3. [[writable]]
4. [[Value]]


`for in`只能够能拿到可枚举的键。

对于ie9以下的版本，有个枚举bug，对象原型里`toString`是不可枚举的。但是`{toString : 1}`这里的`toString`应该可以枚举的，ie9以下的版本对此认为也是不可枚举的，所以需要对这一类特殊的属性进行处理。

*依赖*：

- _.isFunction - Function Functions
- _.has - Object Functions
- _.contains - Collection Functions 
  
  
  
### keys
  
    // Retrieve the names of an object's own properties.
    // Delegates to **ECMAScript 5**'s native `Object.keys`
    _.keys = function(obj) {
      if (!_.isObject(obj)) return [];
      if (nativeKeys) return nativeKeys(obj);
      var keys = [];
      for (var key in obj) if (_.has(obj, key)) keys.push(key);
      // Ahem, IE < 9.
      if (hasEnumBug) collectNonEnumProps(obj, keys);
      return keys;
    };
    
ECMAScript 5中提供了`Object.keys`方法，用来获得一个对象的所有属性键/值对的键。

如果运行环境不支持`Object.keys`方法，就重新实现：

1. 使用`for in`枚举obj对象的所有属性
2. 通过`_.has`得到对象本身的属性，过滤掉原型链上的属性
3. 对于`IE < 9`，通过`collectNonEnumProps`进行特殊处理

*依赖*：

- _.has - Object Functions
- collectNonEnumProps - Object Functions 



### allKeys

    // Retrieve all the property names of an object.
    _.allKeys = function(obj) {
      if (!_.isObject(obj)) return [];
      var keys = [];
      for (var key in obj) keys.push(key);
      // Ahem, IE < 9.
      if (hasEnumBug) collectNonEnumProps(obj, keys);
      return keys;
    };
    
这个函数跟`_.keys`的区别就在于`for in`中没有`if (_.has(obj, key))`判断。

`allKeys`会获取对象的所有属性，包括原型链上的。

*依赖*：

- collectNonEnumProps - Object Functions 



### values

    // Retrieve the values of an object's properties.
    _.values = function(obj) {
      var keys = _.keys(obj);
      var length = keys.length;
      var values = Array(length);
      for (var i = 0; i < length; i++) {
        values[i] = obj[keys[i]];
      }
      return values;
    };

用来获得obj对象的所有属性键/值对的值，并存放在一个数组中返回。  

*依赖*：

- _.keys - Object Functions


### mapObject
  
    // Returns the results of applying the iteratee to each element of the object
    // In contrast to _.map it returns an object
    _.mapObject = function(obj, iteratee, context) {
      iteratee = cb(iteratee, context);
      var keys =  _.keys(obj),
            length = keys.length,
            results = {},
            currentKey;
        for (var index = 0; index < length; index++) {
          currentKey = keys[index];
          results[currentKey] = iteratee(obj[currentKey], currentKey, obj);
        }
        return results;
    };

map函数用来对集合对象进行操作，返回一个数组。

mapObject函数操作对象的所有元素，返回一个对象。

*依赖*：

- cb - 重要的内部函数
- _.keys - Object Functions


  
### pairs
  
    // Convert an object into a list of `[key, value]` pairs.
    _.pairs = function(obj) {
      var keys = _.keys(obj);
      var length = keys.length;
      var pairs = Array(length);
      for (var i = 0; i < length; i++) {
        pairs[i] = [keys[i], obj[keys[i]]];
      }
      return pairs;
    };
  
把obj对象转变为一个[key, value]形式的数组，通过`_.keys`来获取对象的键数组。  

*依赖*：

- _.keys - Object Functions



### invert
    // Invert the keys and values of an object. The values must be serializable.
    _.invert = function(obj) {
      var result = {};
      var keys = _.keys(obj);
      for (var i = 0, length = keys.length; i < length; i++) {
        result[obj[keys[i]]] = keys[i];
      }
      return result;
    };
  
把对象obj的所有属性键/值进行反转。

*依赖*：

- _.keys - Object Functions



### functions & methods  

    // Return a sorted list of the function names available on the object.
    // Aliased as `methods`
    _.functions = _.methods = function(obj) {
      var names = [];
      for (var key in obj) {
        if (_.isFunction(obj[key])) names.push(key);
      }
      return names.sort();
    };

获取对象obj的所有方法，并排序返回。

*依赖*：

- _.isFunction - Function Functions
  
  

### extend  

    // Extend a given object with all the properties in passed-in object(s).
    _.extend = createAssigner(_.allKeys);

`extend`函数的常用形式为`_.extend(destination, *sources) `，将所有sources对象的属性扩展到destination对象上。

这里sources对象的属性包括对象本身的属性和对象原型链上的属性。
  
*依赖*：

- createAssigner - 重要的内部函数  
  
  
  
### extendOwn
   
    // Assigns a given object with all the own properties in the passed-in object(s)
    // (https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)
    _.extendOwn = _.assign = createAssigner(_.keys);

`extendOwn`函数的常用形式为`_.extendOwn(destination, *sources) `，将所有sources对象的属性（不包括对象原型链上的属性）扩展到destination对象上。
  
*依赖*：

- createAssigner - 重要的内部函数   
  
  

### findKey
  
    // Returns the first key on an object that passes a predicate test
    _.findKey = function(obj, predicate, context) {
      predicate = cb(predicate, context);
      // 通过_.keys获取所有属性键
      var keys = _.keys(obj), key;
      for (var i = 0, length = keys.length; i < length; i++) {
        key = keys[i];
        if (predicate(obj[key], key, obj)) return key;
      }
    };
  
该函数用了获取满足predicate条件的对象属性。类似于Array Functions中的findIndex。
  
*依赖*：

- cb - 重要的内部函数   
- _.keys - Object Functions

  
  
### pick
  
    // Return a copy of the object only containing the whitelisted properties.
    _.pick = function(object, oiteratee, context) {
      var result = {}, obj = object, iteratee, keys;
      if (obj == null) return result;
      if (_.isFunction(oiteratee)) {
        keys = _.allKeys(obj);
        iteratee = optimizeCb(oiteratee, context);
      } else {
        keys = flatten(arguments, false, false, 1);
        iteratee = function(value, key, obj) { return key in obj; };
        obj = Object(obj);
      }
      for (var i = 0, length = keys.length; i < length; i++) {
        var key = keys[i];
        var value = obj[key];
        if (iteratee(value, key, obj)) result[key] = value;
      }
      return result;
    };
    
该函数用来获得object对象的一个副本，这个副本是根据特定条件生成的：

1. 如果object为空，直接返回{}
2. 如果参数oiteratee是一个函数，通过optimizeCb函数处理生成一个对object所有属性键/值对处理的函数

        iteratee = function(value, index, collection) {
          return oiteratee.call(context, value, index, collection);
        };
        
3. 如果参数oiteratee不是一个函数，通过flatten函数`flatten(arguments, false, false, 1)`得到arguments中所有的属性键值
    
    - 这里flatten函数的startIndex设置为1，用来避免处理arguments中的object这个参数

    
*依赖*：

- _.isFunction - Function Functions
- _.allKeys - Object Functions 
- optimizeCb - 重要的内部函数
- flatten - Array Functions


      
### omit

     // Return a copy of the object without the blacklisted properties.
    _.omit = function(obj, iteratee, context) {
      if (_.isFunction(iteratee)) {
        iteratee = _.negate(iteratee);
      } else {
        var keys = _.map(flatten(arguments, false, false, 1), String);
        iteratee = function(value, key) {
          return !_.contains(keys, key);
        };
      }
      return _.pick(obj, iteratee, context);
    };

该函数的功能与_.pick函数正好相反，同样是返回一个object副本：只过滤出除去keys

1. 如果iteratee是一个函数，那么就根据这个函数对Object的属性进行过滤
2. 如果iteratee不是一个函数，那么通过map和flatten函数处理arguments为一个键数组，然后创建一个过滤条件iteratee

*依赖*：

- _.isFunction - Function Functions
- _.negate -  Function Functions
- _.map -  Collection Functions
- flatten - Array Functions  
- _.contains - Collection Functions
- _.pick - Object Functions
  

  
### defaults
  
    // Fill in a given object with default properties.
    _.defaults = createAssigner(_.allKeys, true);
  
defaults函数的功能跟`_.allKeys`类似，常用的形式为`_.defaults(object, *defaults)`。唯一一点不同就是，如果

唯一一点不同就是，如果object[key]是undefined，并且defaults对象中包含该key，则使用defaults[key]进行覆盖。

    var person = {name : undefined,age : 1};
    var defaultPerson = {name:'unknown',age :0,sex:'male'};
    var p = _.defaults(person,defaultPerson,{work:'nowork'});
    console.dir(p);//=>{name:'unknow',age:1,sex:'male',work:'nowork'};
  
*依赖*：

- createAssigner - 重要的内部函数     
  
  

### create

    // Creates an object that inherits from the given prototype object.
    // If additional properties are provided then they will be added to the
    // created object.
    _.create = function(prototype, props) {
      var result = baseCreate(prototype);
      if (props) _.extendOwn(result, props);
      return result;
    };
  
create函数用来创建对象，支持两个参数：

1. 一个是被创建对象要继承的原型
2. 第二个参数是一个对象，通过这个对象来扩展新建的对象。

最终实现的效果类似于ECMAScript 5中提供了`Object.create`函数。
  
*依赖*：

- baseCreate - 重要的内部函数    
- _.extendOwn - Object Functions



### clone

    // Create a (shallow-cloned) duplicate of an object.
    _.clone = function(obj) {
      if (!_.isObject(obj)) return obj;
      return _.isArray(obj) ? obj.slice() : _.extend({}, obj);
    };

clone函数用来实现对象的浅拷贝：

1. 如果对象是数组，就直接通过slice方法生成一个新的数组对象
2. 对于非数组类型对象，直接通过_.extend进行克隆
  
*依赖*：

- _.isObject - Object Functions
- _.isArray - Array Functions
- _.extend - Object Functions


  
### tap  

    // Invokes interceptor with the obj, and then returns obj.
    // The primary purpose of this method is to "tap into" a method chain, in
    // order to perform operations on intermediate results within the chain.
    _.tap = function(obj, interceptor) {
      interceptor(obj);
      return obj;
    };
  
tap函数有两个参数：一个对象和一个操作对象的函数。

执行interceptor函数对obj进行操作，然后返回obj。


### property
  
    _.property = function(key) {
      return function(obj) {
        return obj == null ? void 0 : obj[key];
      };
    };
    
访问一个对象的属性，例如：
var wilber = {"name": "wilber", "age":29};
_.property("name")(wilber)  // "wilber"
    


### propertyOf

    // Generates a function for a given object that returns a given property.
    _.propertyOf = function(obj) {
      return obj == null ? function(){} : function(key) {
        return obj[key];
      };
    };
    
同样是访问一个对象的属性，但是跟property函数的形式不同，例如：
var wilber = {"name": "wilber", "age":29};
_.propertyOf(wilber)("name")  // "wilber"    
    

    
### matcher & matches
    
    // Returns a predicate for checking whether an object has a given set of 
    // `key:value` pairs.
    _.matcher = _.matches = function(attrs) {
      attrs = _.extendOwn({}, attrs);
      return function(obj) {
        return _.isMatch(obj, attrs);
      };
    };
    
该函数有两个特点：

1. 返回值是一个断言函数
2. 函数的参数是一个包含一组键/值属性的对象

该函数可以用来辨别给定的对象是否匹配attrs指定键/值属性。  

*依赖*：

- _.isMatch - Object Functions  
    
    

### isMatch

    // Returns whether an object has a given set of `key:value` pairs.
    _.isMatch = function(object, attrs) {
      var keys = _.keys(attrs), length = keys.length;
      // _.isMatch(null,{});
      if (object == null) return !length;
      // 这里通过Object(object)将object强制转换成对象类型
      var obj = Object(object);
      for (var i = 0; i < length; i++) {
        var key = keys[i];
        if (attrs[key] !== obj[key] || !(key in obj)) return false;
      }
      return true;
    };

isMatch函数用来判断是否attrs对象的所有属性都在object中。

*依赖*：

- _.keys - Object Functions 



### isEqual

    // Internal recursive comparison function for `isEqual`.
    var eq = function(a, b, aStack, bStack) {
      // 由于`0 === -0`，所以Underscore在这里进行了特殊处理
      // 这里还结合了`1/0 ===Infinity`和`1/-0===-Infinity`进行判断
      if (a === b) return a !== 0 || 1 / a === 1 / b;
      // 对于`null == undefined`的情况进行特殊处理
      if (a == null || b == null) return a === b;
      // 如果a或b是_的实例，就比较其包裹_wrapped对象
      if (a instanceof _) a = a._wrapped;
      if (b instanceof _) b = b._wrapped;
      // 获取a和b的类型，然后进行类型比较，如果类型不同，直接返回false
      var className = toString.call(a);
      if (className !== toString.call(b)) return false;
      
      // 如果a和b的类型相同，进一步比较
      switch (className) {
        // Strings, numbers, regular expressions, dates, and booleans are compared by value.
        case '[object RegExp]':
        // RegExps are coerced to strings for comparison (Note: '' + /a/i === '/a/i')
        case '[object String]':
          // Primitives and their corresponding object wrappers are equivalent; thus, `"5"` is
          // equivalent to `new String("5")`.
          // 对于RegExp和String类型，通过''+操作转换为字符串，然后直接进行值比较
          return '' + a === '' + b;
        case '[object Number]':
          // 这里主要针对NaN的情况进行处理，NaN虽然不等于自身，但是Underscore认为它是值相等的
	  	// 如果下面+a !== +a为true，说明a是NaN
	  	// 如果+b !== +b为true，说明b也是NaN，所以a和b值相等
	  	// 如果+b !== +b为false，说明b不是NaN，自然a和b值不等
          if (+a !== +a) return +b !== +b;
          // 如果不是NaN，就要考虑0的情况
          return +a === 0 ? 1 / +a === 1 / b : +a === +b;
        case '[object Date]':
        case '[object Boolean]':
          // 日期和布尔值会被转化成数字来比较
          return +a === +b;
      }
    
      var areArrays = className === '[object Array]';
      if (!areArrays) {
        // 如果a不是数组类型，进入该逻辑
        // 只要a和b有一个不是Object类型，直接返回false 
        if (typeof a != 'object' || typeof b != 'object') return false;
    
        // 这里进行了三个判断：构造器是否一致；构造器是否是函数并满足`?Ctor instanceof ?Ctor`；对象是否包含构造器属性
        // 关于第二个判断主要是针对下面的情况
        // var a = Object.create([1,2,3])
        // Object.prototype.toString.call(a) // "[object Object]"
        // a.constructor // function Array
        // 由于a的特殊性，所以需要进心`?Ctor instanceof ?Ctor`
        // Function instanceof Function == true
        // Object instanceof Object == true
        // Array instanceof Array == false
        var aCtor = a.constructor, bCtor = b.constructor;
        if (aCtor !== bCtor && !(_.isFunction(aCtor) && aCtor instanceof aCtor &&
                                 _.isFunction(bCtor) && bCtor instanceof bCtor)
                            && ('constructor' in a && 'constructor' in b)) {
          return false;
        }
      }
      // Assume equality for cyclic structures. The algorithm for detecting cyclic
      // structures is adapted from ES 5.1 section 15.12.3, abstract operation `JO`.
      
      // Initializing stack of traversed objects.
      // It's done here since we only need them for objects and arrays comparison.
      // 初始化stack
      // 用于对象,数组的比较
      aStack = aStack || [];
      bStack = bStack || [];
      var length = aStack.length;
      while (length--) {
        // Linear search. Performance is inversely proportional to the number of
        // unique nested structures.
        // 线性搜索，判断是否是嵌套结构
        if (aStack[length] === a) return bStack[length] === b;
      }
    
      // 把a和b放到栈里
      aStack.push(a);
      bStack.push(b);
    
      // 递归比较对象和数组
      if (areArrays) {
        // 深度比较数组
        // 通过比较数组长度来确认是否需要进一步比较，如果长度不等，直接返回false
        length = a.length;
        if (length !== b.length) return false;
        // 通过递归比较每个数组元素，任何一个元素不相等都返回false
        while (length--) {
          if (!eq(a[length], b[length], aStack, bStack)) return false;
        }
      } else {
        // 深度比较对象
        var keys = _.keys(a), key;
        length = keys.length;
        // 确保两个对象含有相同数量的属性，否则直接返回false
        if (_.keys(b).length !== length) return false;
        while (length--) {
          // 深度比较每个属性
          // 属性名存在，然看值是否相等，不等就返回false
          key = keys[length];
          if (!(_.has(b, key) && eq(a[key], b[key], aStack, bStack))) return false;
        }
      }
      
      // 移除栈里最新的元素，说明该层是相等的
      aStack.pop();
      bStack.pop();
      return true;
    };
    
    // Perform a deep comparison to check if two objects are equal.
    _.isEqual = function(a, b) {
      return eq(a, b);
    };
  
isEqual函数用来判断两个对象是否相等，所有的实现细节都放在了eq这个函数中，具体请看eq函数的注释。

eq函数单独实现是为了方便递归，进行深度比较。

*依赖*：

- _.keys - Object Functions
- _.has - Object Functions
- _.isFunction - Function Functions

  
  
### isEmpty

    // Is a given array, string, or object empty?
    // An "empty" object has no enumerable own-properties.
    _.isEmpty = function(obj) {
      if (obj == null) return true;
      if (isArrayLike(obj) && (_.isArray(obj) || _.isString(obj) || _.isArguments(obj))) return obj.length === 0;
      return _.keys(obj).length === 0;
    };

该函数用来判断obj是否为空：

1. 如果obj为`null`或者`undefined`，直接返回`true`
2. 对于`Array`，`ArrayLike`，`String`和`arguments`等对象，通过判断`length`来确定是否为空
3. 对于`{}`或者`new Object()`对象，通过`key`个数来确定是否为空对象

*依赖*：

- _.keys - Object Functions 



### isElement

    // Is a given value a DOM element?
    _.isElement = function(obj) {
      return !!(obj && obj.nodeType === 1);
    };

判断obj是否为dom元素，可以直接通过`nodeType`属性值是否为1来判断。    

注意这里`!!`的使用，`!!`的作用就是将后面的表达式强转为boolean值，也就是结果只能是`true`或者`false`。  

例如：

    var a；    // a默认是undefined
    var b=!!a;    // !a是true，!!a则是false，所以b的值是false，而不再是undefined
    
    

### isArray

    // Is a given value an array?
    // Delegates to ECMA5's native Array.isArray
    _.isArray = nativeIsArray || function(obj) {
      return toString.call(obj) === '[object Array]';
    };

`Array.isArray`函数是ECMAScript 5新增函数，用来判断对象是否是数组。   

如果运行环境不支持`Array.isArray`函数，就通过`Object.prototype.toString`来判断。



### isArrayLike

    // Helper for collection methods to determine whether a collection
    // should be iterated as an array or as an object
    // Related: http://people.mozilla.org/~jorendorff/es6-draft.html#sec-tolength
    var MAX_ARRAY_INDEX = Math.pow(2, 53) - 1;
    var isArrayLike = function(collection) {
      var length = collection != null && collection.length;
      return typeof length == 'number' && length >= 0 && length <= MAX_ARRAY_INDEX;
    };

在JavaScript中有一些“类数组对象”，例如arguments，document.getElementsByTagName()的结果。

对于这类对象的判断，主要是根据length属性，要求该属性首先是数字类型，然后是在一定范围内的正数。

    
    
### isObject

    // Is a given variable an object?
    _.isObject = function(obj) {
      var type = typeof obj;
      return type === 'function' || type === 'object' && !!obj;
    };

判断obj是否为对象：

1. 首先通过`typeof`获取对象的类型
2. 函数也属于对象
3. 注意由于`typeof null`也是`object`，所以用!!obj来特殊处理这种情况。



### isArguments, isFunction, isString, isNumber, isDate, isRegExp, isError

    // Add some isType methods: isArguments, isFunction, isString, isNumber, isDate, isRegExp, isError.
    _.each(['Arguments', 'Function', 'String', 'Number', 'Date', 'RegExp', 'Error'], function(name) {
      _['is' + name] = function(obj) {
        return toString.call(obj) === '[object ' + name + ']';
      };
    });

通过`_`的`each`方法给`Arguments`, `Function`, `String`, `Number`, `Date`, `RegExp`, `Error`添加判断。

判断方法直接使用的是`Object.prototype.toString`。

*依赖*：

- _.each - Collection Functions 



### isArguments hack

    // Define a fallback version of the method in browsers (ahem, IE < 9), where
    // there isn't any inspectable "Arguments" type.
    if (!_.isArguments(arguments)) {
      _.isArguments = function(obj) {
        return _.has(obj, 'callee');
      };
    }

这里对于`IE < 9`的情况进行了特殊处理。

有的浏览器没有`arguments`这种类型，通过判断是否有`callee`属性来判断。

*依赖*：

- _.isArguments - Object Functions 
- _.has - Object Functions 



### isFunction hack

    // Optimize `isFunction` if appropriate. Work around some typeof bugs in old v8,
    // IE 11 (#1621), and in Safari 8 (#1929).
    if (typeof /./ != 'function' && typeof Int8Array != 'object') {
      _.isFunction = function(obj) {
        return typeof obj == 'function' || false;
      };
    }
    
在进行是否是`Function`对象的判断的时候，对ie11和safari进行了特殊处理

*依赖*：

- _.isFunction - Function Functions 



### isFinite

    // Is a given object a finite number?
    _.isFinite = function(obj) {
      return isFinite(obj) && !isNaN(parseFloat(obj));
    };
    
对于全局的`isFinite(number)`，如果 number 是有限数字（或可转换为有限数字），那么返回 true。否则，如果 number 是 NaN（非数字），或者是正、负无穷大的数，则返回 false。

**在`_`中，对Empty String, Booleans, Dates以及可以转换成Number担不是Number的对象进行了处理.**

例如：

    isFinite('') IS true;  
    isNaN('') IS false; 
    
    isFinite(new Date()) IS true;  
    isNaN(new Date()) IS false;      

    
    
### isNaN
    
    // Is the given value `NaN`? (NaN is the only number which does not equal itself).
    _.isNaN = function(obj) {
      return _.isNumber(obj) && obj !== +obj;
    };

判断obj是否为`NaN`，NaN这个值有两个特点：它是一个数；它不等于它自己。  

1. 先通过`_.isNumber`看看obj是不是数字类型，`typeof NaN === "number"`
2. 如果是，进一步判断obj是否不等于自身
3. 如果是，那么obj就是就`NaN`

全局函数也有个isNaN，二者区别是，这里的isNaN是排除掉undefined也是NaN的情况。

*依赖*：

- _.isNumber - Object Functions



### isBoolean

    // Is a given value a boolean?
    _.isBoolean = function(obj) {
      return obj === true || obj === false || toString.call(obj) === '[object Boolean]';
    };

该函数用来判断obj是不是`boolean`，除了判断`true`和`false`，表达式的第三部分判断了`var b = new Boolean()`这种情况。



### isNull

    // Is a given value equal to null?
    _.isNull = function(obj) {
      return obj === null;
    };

该函数用来判断obj是不是`null`，注意使用的是三等号。



### isUndefined

    // Is a given variable undefined?
    _.isUndefined = function(obj) {
      return obj === void 0;
    };

该函数用来判断obj是不是`undefined`，这里使用了JavaScript中的`void`操作符，该操作符有两种使用形式`void expression`和`void (expression)`。  

无论`void`后的表达式是什么，`void`操作符都会返回`undefined`，使用`void 0`是一种更保险的做法，因为`undefined`在javascript中不是保留字。



### has

    // Shortcut function for checking if an object has a given property directly
    // on itself (in other words, not on a prototype).
    _.has = function(obj, key) {
      return obj != null && hasOwnProperty.call(obj, key);
    };

通过原生的`hasOwnProperty`方法来查看obj对象是否含有key属性  


