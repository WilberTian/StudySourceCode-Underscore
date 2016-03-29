


### each & forEach

    // The cornerstone, an `each` implementation, aka `forEach`.
    // Handles raw objects in addition to array-likes. Treats all
    // sparse array-likes as if they were dense.
    _.each = _.forEach = function(obj, iteratee, context) {
      iteratee = optimizeCb(iteratee, context);
      var i, length;
      if (isArrayLike(obj)) {
        for (i = 0, length = obj.length; i < length; i++) {
          iteratee(obj[i], i, obj);
        }
      } else {
        var keys = _.keys(obj);
        for (i = 0, length = keys.length; i < length; i++) {
          iteratee(obj[keys[i]], keys[i], obj);
        }
      }
      return obj;
    };

该函数的功能类似于Array.prototype.forEach，同时可以支持对象和类数组对象。

在函数的开始会判断obj是不是类数组类型：

1. 如果obj不是数组，那么就通过_.keys得到所有的属性键，通过iteratee函数处理obj的所有属性键/值对
2. 如果obj是类数组类型，就通过iteratee函数处理数组中的每一个元素

*依赖*：

- optimizeCb - 重要的内部函数  
- isArrayLike - Object Functions
- _.keys - Object Functions
  
  
  
### map & collect
  
    // Return the results of applying the iteratee to each element.
    _.map = _.collect = function(obj, iteratee, context) {
      iteratee = cb(iteratee, context);
      var keys = !isArrayLike(obj) && _.keys(obj),
          length = (keys || obj).length,
          results = Array(length);
      for (var index = 0; index < length; index++) {
        var currentKey = keys ? keys[index] : index;
        results[index] = iteratee(obj[currentKey], currentKey, obj);
      }
      return results;
    };
    
该函数功能类似与Array.prototype.map，同时可以支持对象和类数组对象。

在函数的开始会判断obj是不是类数组类型：

1. 如果obj不是数组，那么就通过_.keys得到所有的属性键，通过iteratee函数处理obj的所有属性键/值对
2. 如果obj是类数组类型，就通过iteratee函数处理数组中的每一个元素

对于iteratee这个函数，是根据iteratee参数结合cb函数生成的一个函数。

*依赖*：

- cb - 重要的内部函数  
- isArrayLike - Object Functions
- _.keys - Object Functions


  
### reduce & foldl & inject & reduceRight & foldr

    // dir参数表示迭代的方向
    function createReduce(dir) {

      function iterator(obj, iteratee, memo, keys, index, length) {
        // 对集合中的元素进行遍历
        for (; index >= 0 && index < length; index += dir) {
          var currentKey = keys ? keys[index] : index;
          // 进行迭代计算，并保存计算结果
          memo = iteratee(memo, obj[currentKey], currentKey, obj);
        }
        return memo;
      }
    
      return function(obj, iteratee, memo, context) {
        // 设置执行上下文
        iteratee = optimizeCb(iteratee, context, 4);
        var keys = !isArrayLike(obj) && _.keys(obj),
            length = (keys || obj).length,
            index = dir > 0 ? 0 : length - 1;
        // 如果没有设置初始值，就将集合中的第一个元素作为初始值
        if (arguments.length < 3) {
          memo = obj[keys ? keys[index] : index];
          index += dir;
        }
        return iterator(obj, iteratee, memo, keys, index, length);
      };
    }
    
    // **Reduce** builds up a single result from a list of values, aka `inject`,
    // or `foldl`.
    _.reduce = _.foldl = _.inject = createReduce(1);
    
    // The right-associative version of reduce, also known as `foldr`.
    _.reduceRight = _.foldr = createReduce(-1);

reduce函数的作用就是对集合中的元素进行迭代计算，参数memo为迭代的初始状态。

reduceRight函数作用跟reduce类似，只不过迭代的方向正好相反。

reduce和reduceRight函数都依赖了createReduce这个内部函数。
  
*依赖*：

- optimizeCb - 重要的内部函数
- isArrayLike - Object Functions
- _.keys - Object Functions 
  
  
  
### find & detect
  
    // Return the first value which passes a truth test. Aliased as `detect`.
    _.find = _.detect = function(obj, predicate, context) {
      var key;
      if (isArrayLike(obj)) {
        key = _.findIndex(obj, predicate, context);
      } else {
        key = _.findKey(obj, predicate, context);
      }
      if (key !== void 0 && key !== -1) return obj[key];
    };

在obj中逐项查找，返回第一个通过predicate检测的元素值，如果没有值传递给测试迭代器将返回undefined。 

1. 如果是类数组对象，就通过findIndex判断是否有有满足条件的值，通过索引进行取值
2. 如果是普通对象，通过findKey拿到属性键，然后获取对应的属性值  
  
*依赖*：

- isArrayLike - Object Functions
- _.findIndex - Array Functions 
- _.findKey - Object Functions  
  
  
  
### filter & select
  
    // Return all the elements that pass a truth test.
    // Aliased as `select`.
    _.filter = _.select = function(obj, predicate, context) {
      var results = [];
      predicate = cb(predicate, context);
      _.each(obj, function(value, index, list) {
        if (predicate(value, index, list)) results.push(value);
      });
      return results;
    };
    
该函数的功能类似于Array.prototype.filter，但是同样可以支持对象和类数组对象。

函数中通过cb创建一个判断函数，然后通过判断函数对对象的所有属性或者数组的所有元素进行判断。

*依赖*：

- cb - 重要的内部函数
- _.each - Collection Functions    



### reject

    // Return all the elements for which a truth test fails.
    _.reject = function(obj, predicate, context) {
      return _.filter(obj, _.negate(cb(predicate)), context);
    };
    
该函数的功能与_.filter正好相反，用来过滤出不满足条件的元素。

函数中通过_.negate获得相反的谓词条件，然后使用_.filter函数进行过滤。

*依赖*：

- _.filter - Collection Functions    
- _.negate - Function Functions
  
  

### every & all
  
    // Determine whether all of the elements match a truth test.
    // Aliased as `all`.
    _.every = _.all = function(obj, predicate, context) {
      predicate = cb(predicate, context);
      // 判断对象类型，对于普通对象就获取所有的属性键数组
      var keys = !isArrayLike(obj) && _.keys(obj),
          length = (keys || obj).length;
      for (var index = 0; index < length; index++) {
        var currentKey = keys ? keys[index] : index;
        // 遍历obj中的每一个元素，进行判断，任何一个失败都直接返回false
        if (!predicate(obj[currentKey], currentKey, obj)) return false;
      }
      return true;
    };

该函数用来判读是否所有的obj中的元素都满足predicate判断。

*依赖*：

- cb - 重要的内部函数
- isArrayLike - Object Functions
- _.keys - Object Functions


  
### some & any
  
    // Determine if at least one element in the object matches a truth test.
    // Aliased as `any`.
    _.some = _.any = function(obj, predicate, context) {
      predicate = cb(predicate, context);
      var keys = !isArrayLike(obj) && _.keys(obj),
          length = (keys || obj).length;
      for (var index = 0; index < length; index++) {
        var currentKey = keys ? keys[index] : index;
        if (predicate(obj[currentKey], currentKey, obj)) return true;
      }
      return false;
    };

该函数逻辑跟every/all类似，但是用来判断是否obj中至少有一个元素满足predicate判断

*依赖*：

- cb - 重要的内部函数
- isArrayLike - Object Functions
- _.keys - Object Functions


  
### contains & includes & include
 
    // Determine if the array or object contains a given value (using `===`).
    // Aliased as `includes` and `include`.
    _.contains = _.includes = _.include = function(obj, target, fromIndex) {
      // 如果不是类数组类型，就获取所有的属性值组成的数组
      if (!isArrayLike(obj)) obj = _.values(obj);
      // 通过_.indexOf来获取目标元素的下标
      return _.indexOf(obj, target, typeof fromIndex == 'number' && fromIndex) >= 0;
    };
  
该函数用来判断集合中是否包含指定的元素，通过fromIndex参数可以指定起始的查找下标。

*依赖*：

- isArrayLike - Object Functions
- _.values - Object Functions
- _.indexOf - Array Functions



### invoke

    // Invoke a method (with arguments) on every item in a collection.
    _.invoke = function(obj, method) {
      var args = slice.call(arguments, 2);
      var isFunc = _.isFunction(method);
      return _.map(obj, function(value) {
        var func = isFunc ? method : value[method];
        return func == null ? func : func.apply(value, args);
      });
    };
  
在obj这个集合对象的每一个元素上执行method操作，根据method是否是函数类型会产生不同的效果：

1. 如果method是函数，invoke的效果就相当于map函数
2. 如果method不是函数，那就就会认为method是对象的属性，返回所有的属性值，效果相当与values函数  

*依赖*：

- isFunction - Object Functions
- _.map - Collection Functions

  
  
###pluck
  
    // Convenience version of a common use case of `map`: fetching a property.
    _.pluck = function(obj, key) {
      return _.map(obj, _.property(key));
    };
  
执行_.property函数返回的是一个属性访问器/下标访问器函数，结合_.map一起使用时，pluck函数的作用就是提取数组对象中的某个属性，返回一个数组。

    var staff = [{name: wilber', age: 29}, {name: 'will', age: 30}, {name: 'july', age: 28}];
    _.pluck(staff, 'name');
    
    // ["wilber", "will", "july"]
  
*依赖*：

- _.map - Collection Functions
- _.property- Object Functions



### where

    // Convenience version of a common use case of `filter`: selecting only objects
    // containing specific `key:value` pairs.
    _.where = function(obj, attrs) {
      return _.filter(obj, _.matcher(attrs));
    };

该函数遍历集合对象obj，得到包含attrs所有属性键/值对的元素组成的数组。 
  
*依赖*：

- _.filter - Collection Functions
- _.matcher- Object Functions  
  
  
  
### findWhere
  
    // Convenience version of a common use case of `find`: getting the first object
    // containing specific `key:value` pairs.
    _.findWhere = function(obj, attrs) {
      return _.find(obj, _.matcher(attrs));
    };

该函数功能跟where类似，但是值返回第一个匹配的元素。

*依赖*：

- _.find - Collection Functions
- _.matcher- Object Functions    


  
### max
  
    // Return the maximum element (or element-based computation).
    _.max = function(obj, iteratee, context) {
      var result = -Infinity, lastComputed = -Infinity,
          value, computed;
      // 如果iteratee为null，直接进行值比较
      if (iteratee == null && obj != null) {
        // 判断obj是类数组对象，还是一个普通对象
        obj = isArrayLike(obj) ? obj : _.values(obj);
        for (var i = 0, length = obj.length; i < length; i++) {
          value = obj[i];
          if (value > result) {
            result = value;
          }
        }
      } else {
        // 通过cb函数创建确定每个元素判断依据的回调函数
        iteratee = cb(iteratee, context);
        _.each(obj, function(value, index, list) {
          // computed为计算后的判断依据
          computed = iteratee(value, index, list);
          if (computed > lastComputed 
              // 通过下面的条件避免结果直接返回-Infinity
              || computed === -Infinity && result === -Infinity) {
            result = value;
            lastComputed = computed;
          }
        });
      }
      return result;
    };

max函数用来获取obj中最大的元素，支持数组对象和普通对象。

*依赖*：  

- isArrayLike - Object Functions
- _.values - Object Functions
- cb - 重要的内部函数
- _.each - Collection Functions



### min
  
    // Return the minimum element (or element-based computation).
    _.min = function(obj, iteratee, context) {
      var result = Infinity, lastComputed = Infinity,
          value, computed;
      if (iteratee == null && obj != null) {
        obj = isArrayLike(obj) ? obj : _.values(obj);
        for (var i = 0, length = obj.length; i < length; i++) {
          value = obj[i];
          if (value < result) {
            result = value;
          }
        }
      } else {
        iteratee = cb(iteratee, context);
        _.each(obj, function(value, index, list) {
          computed = iteratee(value, index, list);
          if (computed < lastComputed || computed === Infinity && result === Infinity) {
            result = value;
            lastComputed = computed;
          }
        });
      }
      return result;
    };

该函数功能与max相反，用来获取obj中最小的元素。

*依赖*：  

- isArrayLike - Object Functions
- _.values - Object Functions
- cb - 重要的内部函数
- _.each - Collection Functions



### shuffle
  
    // Shuffle a collection, using the modern version of the
    // [Fisher-Yates shuffle](http://en.wikipedia.org/wiki/Fisher–Yates_shuffle).
    _.shuffle = function(obj) {
      var set = isArrayLike(obj) ? obj : _.values(obj);
      var length = set.length;
      var shuffled = Array(length);
      for (var index = 0, rand; index < length; index++) {
        rand = _.random(0, index);
        if (rand !== index) shuffled[index] = shuffled[rand];
        shuffled[rand] = set[index];
      }
      return shuffled;
    };
  
该函数用来随机打乱集合元素的顺序，如果obj是类数组对象，则对数组进行打乱；如果obj是普通对象，通过_.values获得所有的属性值组成的数组，然后进行打乱。

这里使用的打乱原理就是：每次都随机生成一个下标rand，如果下标不等于当前下标，把shuffed中其对应的元素放到当前位置上。然后shuffled其对应的位置上放置set中当前位置的元素。

*依赖*：  

- isArrayLike - Object Functions
- _.values - Object Functions
- _.random - Utility Functions

  

### sample
  
    // Sample **n** random values from a collection.
    // If **n** is not specified, returns a single random element.
    // The internal `guard` argument allows it to work with `map`.
    _.sample = function(obj, n, guard) {
      if (n == null || guard) {
        if (!isArrayLike(obj)) obj = _.values(obj);
        return obj[_.random(obj.length - 1)];
      }
      return _.shuffle(obj).slice(0, Math.max(0, n));
    };
  
通过obj这个集合元素，获取一个大小为n的随机样本：

1. 如果没有设置参数n，那么就返回一个长度为1的样本
2. 如果设置参数n，通过shuffle打乱数组，取前n个元素

*依赖*：  

- isArrayLike - Object Functions
- _.values - Object Functions
- _.random - Utility Functions
- _.shuffle - Collection Functions

  
### sortBy

    // Sort the object's values by a criterion produced by an iteratee.
    _.sortBy = function(obj, iteratee, context) {
      iteratee = cb(iteratee, context);
      return _.pluck(_.map(obj, function(value, index, list) {
        return {
          value: value,
          index: index,
          criteria: iteratee(value, index, list)
        };
      }).sort(function(left, right) {
        var a = left.criteria;
        var b = right.criteria;
        if (a !== b) {
          if (a > b || a === void 0) return 1;
          if (a < b || b === void 0) return -1;
        }
        return left.index - right.index;
      }), 'value');
    };
  
对集合obj中的所有元素进行排序，其实参数iteratee代表排序的标准。该函数的结构比较复杂，大致的流程如下：

1. 首先进行_.map().sort()
2. 然后通过_.pluck()提取所有的value属性，就得到了排序后的结果  

*依赖*：  

- cb - 重要的内部函数
- _.pluck - Collection Functions
- _.map - Collection Functions


  
### groupBy  
  
    // An internal function used for aggregate "group by" operations.
    var group = function(behavior) {
      // group的返回值是一个函数
      return function(obj, iteratee, context) {
        var result = {};
        // iteratee为分组的判断条件
        iteratee = cb(iteratee, context);
        _.each(obj, function(value, index) {
          var key = iteratee(value, index, obj);
          // behavior函数用来处理分组后的结果
          behavior(result, value, key);
        });
        // 返回结果
        return result;
      };
    };
    
    // Groups the object's values by a criterion. Pass either a string attribute
    // to group by, or a function that returns the criterion.
    _.groupBy = group(function(result, value, key) {
      if (_.has(result, key)) result[key].push(value); else result[key] = [value];
    });
  
groupBy函数的作用就是把一个集合分组为多个集合，代码中用到了group这个内部函数。

*依赖*：

- _.has - Object Functions
- cb - 重要的内部函数
- _.each - Collection Functions
- group - Collection Functions

  

### indexBy  

    // Indexes the object's values by a criterion, similar to `groupBy`, but for
    // when you know that your index values will be unique.
    _.indexBy = group(function(result, value, key) {
      result[key] = value;
    });
  
该函数同样依赖group函数，功能跟groupBy类似，为一不同的是对结果进行处理的behavior函数不同。

*依赖*：

- group - Collection Functions  



### countBy

    // Counts instances of an object that group by a certain criterion. Pass
    // either a string attribute to count by, or a function that returns the
    // criterion.
    _.countBy = group(function(result, value, key) {
      if (_.has(result, key)) result[key]++; else result[key] = 1;
    });

该函数类似groupBy函数，但是不是返回各个分组的值，而是返回各个分组中元素的数量。

这里也是只改变了behavior函数，重用了group这个内部函数。
  
*依赖*：

- _.has - Object Functions
- group - Collection Functions

  
  
### toArray
  
    // Safely create a real, live array from anything iterable.
    _.toArray = function(obj) {
      // 如果obj为空，直接返回[]
      if (!obj) return [];
      // 如果obj是数组类型，直接通过Array.prototype.slice生成一个数组
      if (_.isArray(obj)) return slice.call(obj);
      // 如果obj是类数组类型，通过_.map和_.identity函数生成一个新的数组
      if (isArrayLike(obj)) return _.map(obj, _.identity);
      // 对于普通的对象，创建一个数组，包含对象所有属性的值
      return _.values(obj);
    };

该函数的作用就是通过一个可迭代的对象创建一个数组。

*依赖*：

- _.isArray - Object Functions
- isArrayLike - Object Functions  
- _.map - Collection Functions
- _.identity - Utility Functions
- _.values - Object Functions


  
### size
  
    // Return the number of elements in an object.
    _.size = function(obj) {
      if (obj == null) return 0;
      return isArrayLike(obj) ? obj.length : _.keys(obj).length;
    };
    
该函数用来获得对象所包含的成员数量：

1. 如果obj是类数组对象，就通过`length`属性获取数组长度
2. 如果obj是普通对象类型，就通过_.keys获取所有的属性键，并取长度

*依赖*：

- isArrayLike - Object Functions
- _.keys - Object Functions


  
### partition
  
    // Split a collection into two arrays: one whose elements all satisfy the given
    // predicate, and one whose elements all do not satisfy the predicate.
    _.partition = function(obj, predicate, context) {
      predicate = cb(predicate, context);
      var pass = [], fail = [];
      _.each(obj, function(value, key, obj) {
        // 对每个元素进行条件判断，然后添加到不同的数组
        (predicate(value, key, obj) ? pass : fail).push(value);
      });
      return [pass, fail];
    };

该函数的作用是将一个数组拆分为两个数组：第一个数组所有元素都满足predicate条件；第二个数组的所有元素都不满足predicate条件。

*依赖*：

- cb - 重要的内部函数
- _.each - Collection Functions


