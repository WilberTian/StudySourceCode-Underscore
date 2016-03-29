

### first & head & take

    // Get the first element of an array. Passing **n** will return the first N
    // values in the array. Aliased as `head` and `take`. The **guard** check
    // allows it to work with `_.map`.
    _.first = _.head = _.take = function(array, n, guard) {
      if (array == null) return void 0;
      if (n == null || guard) return array[0];
      return _.initial(array, array.length - n);
    };
    
    
该函数的主要作用就是获取数组的前n个元素，n默认值为1。

函数内部首先对数组数否为空进行判断，如果数组非空，直接使用_.initial获取数组前n个元素。

guard用于确定只返回一个元素，当guard为true时，指定数量n无效。

*依赖*：

- _.initial - Array Functions



### initial
  
    // Returns everything but the last entry of the array. Especially useful on
    // the arguments object. Passing **n** will return all the values in
    // the array, excluding the last N.
    _.initial = function(array, n, guard) {
      return slice.call(array, 0, Math.max(0, array.length - (n == null || guard ? 1 : n)));
    };
  
该函数的常用形式为_.initial(array, n)，主要作用就是返回不包含最后n个元素的数组。如果没有传入参数n，那么n默认值为1。

函数中通过Math.max避免了负数的情况。

对于guard参数，当guard为true的时候，`(n == null || guard ? 1 : n)`的值将始终为1，也就是始终返回除了最后一个元素的数组。

  
  
### last
  
    // Get the last element of an array. Passing **n** will return the last N
    // values in the array.
    _.last = function(array, n, guard) {
      if (array == null) return void 0;
      if (n == null || guard) return array[array.length - 1];
      return _.rest(array, Math.max(0, array.length - n));
    };
    
该函数与_.first相反，返回数组的最后一个或倒序指定的n个元素    
    
*依赖*：

- _.rest - Array Functions    



### rest & tail & drop
    
    // Returns everything but the first entry of the array. Aliased as `tail` and `drop`.
    // Especially useful on the arguments object. Passing an **n** will return
    // the rest N values in the array.
    _.rest = _.tail = _.drop = function(array, n, guard) {
      return slice.call(array, n == null || guard ? 1 : n);
    };
    
该函数作用与_.initial相反，获取第n元素之后的元素构成的数组，n默认为1。
    
    
  
### compact
  
    // Trim out all falsy values from an array.
    _.compact = function(array) {
      return _.filter(array, _.identity);
    };
    
这个函数借助filter函数，返回数组中所有值能被转换为true的元素, 得到一个新的数组。不能被转换的值包括

不能被转换的值包括 false, 0, '', null, undefined, NaN, 这些值将被转换为false。

*依赖*：

- _.filter - Collection Functions
- _.identity - Utility Functions


 
### flatten
  
    // Internal implementation of a recursive `flatten` function.
    var flatten = function(input, shallow, strict, startIndex) {
      var output = [], idx = 0;
      for (var i = startIndex || 0, length = getLength(input); i < length; i++) {
        var value = input[i];
        // 判断对象是不是数组类型，类数组类型或者arguments
        if (isArrayLike(value) && (_.isArray(value) || _.isArguments(value))) {
          // 如果shallow为false，说明是深度展开，递归调用flatten函数
          if (!shallow) value = flatten(value, shallow, strict);
          var j = 0, len = value.length;
          output.length += len;
          while (j < len) {
            output[idx++] = value[j++];
          }
        } else if (!strict) {
          output[idx++] = value;
        }
      }
      return output;
    };
  
    // Flatten out an array, either recursively (by default), or just one level.
    _.flatten = function(array, shallow) {
      return flatten(array, shallow, false);
    };
    
这个函数用于将一个多维数展开成为一维数组, 支持深层展开，其中第二个参数shallow用于控制展开深度。

当shallow为true时, 只展开第一层, 默认进行深层展开。

对于strict和startIndex这两个参数，是给_.flatten之外的其他api使用的：

- strict是否是严格的，如果不是数组或者类数组，并且strict参数为false，直接拷贝value到output中。
- startIndex表示其实下标位置

*依赖*：

- isArrayLike - Object Functions
- _.isArray - Object Functions
- _.isArguments - Object Functions



### without
  
    // Return a version of the array that does not contain the specified value(s).
    _.without = function(array) {
      // 利用_.difference函数来剔除元素
      return _.difference(array, slice.call(arguments, 1));
    };
    
该函数的常用形式为`_.without(array, *values) `，返回一个删除所有values值后的array对象。

*依赖*：

- _.difference - Array Functions

    
  
### uniq & unique

    // Produce a duplicate-free version of the array. If the array has already
    // been sorted, you have the option of using a faster algorithm.
    // Aliased as `unique`.
    _.uniq = _.unique = function(array, isSorted, iteratee, context) {
      if (array == null) return [];
      // 对于未传入isSorted的情况，对函数的参数进行调整
      if (!_.isBoolean(isSorted)) {
        context = iteratee;
        iteratee = isSorted;
        isSorted = false;
      }
      // 通过cb函数，结合iteratee生成回调函数
      if (iteratee != null) iteratee = cb(iteratee, context);
      var result = [];
      var seen = [];
      for (var i = 0, length = array.length; i < length; i++) {
        var value = array[i],
            computed = iteratee ? iteratee(value, i, array) : value;
        // 逻辑1
        if (isSorted) {
          if (!i || seen !== computed) result.push(value);
          seen = computed;
        } 
        // 逻辑2
        else if (iteratee) {
          if (!_.contains(seen, computed)) {
            seen.push(computed);
            result.push(value);
          }
        } 
        // 逻辑3
        else if (!_.contains(result, value)) {
          result.push(value);
        }
      }
      return result;
    };
  
该函数主要用来剔除数组中的重复元素，如果数组是已经拍好序的，那么可以设置isSorted参数为true来进行加速处理。

该函数中在for循环遍历数组的代码中有三段主要的逻辑：

1. 当设置isSorted为true的时候，表示array对象是拍好序的，所以可以使用seen来保存前一次的结果进行比较，而不用通过_.contains
2. 如果参数iteratee是函数类型，那么computed就是通过回调函数处理之后的结果，seen数组用来保存已经处理过的元素

        _.uniq([{name : 'wilber'}, {name:'wilber', age:29}, 3, 3, {name:'willl'}], function(value){
            if(typeof value === 'object'){
                return value.name;
            }
            return value;
        });
        
        // [{name : 'wilber'}, 3, {name:'will'}]

3. 如果value不包含在result中，就把value添加到result中

*依赖*：

- _.isBoolean - Object Functions
- cb - 重要的内部函数  
- _.contains - Collection Functions  
  
  

### union  

  // Produce an array that contains the union: each distinct element from all of
  // the passed-in arrays.
  _.union = function() {
    return _.uniq(flatten(arguments, true, true));
  };
  
union函数可以接受多个数组作为参数，然后对多个数组进行合并，也就是获得多个数组的并集。

该函数中使用了flatten函数进行数组展开（注意这里的参数为true，是浅展开），然后又通过了_.uniq函数进行去重。

*依赖*：

- _.uniq - Collection Functions
- flatten - Array Functions

  

  
### intersection
  
    // Produce an array that contains every item shared between all the
    // passed-in arrays.
    _.intersection = function(array) {
      // 如果array为null，直接返回[]
      if (array == null) return [];
      var result = [];
      // 获取参数长度，也就是一共有多少个数组对象参数
      var argsLength = arguments.length;
      // 遍历参数中的第一个数组对象array
      for (var i = 0, length = array.length; i < length; i++) {
        var item = array[i];
        // 如果result中包含item，那么就continue
        if (_.contains(result, item)) continue;
        for (var j = 1; j < argsLength; j++) {
          // 遍历其他数组对象，如果其他数组有一个没有此元素，循环结束
          if (!_.contains(arguments[j], item)) break;
        }
        // 如果j等argsLength说明上面for循环没有被break，所有数组里都有此值
        if (j === argsLength) result.push(item);
      }
      return result;
    };
  
intersection函数可以接受多个数组作为参数，然后获得数组的交集。
  
*依赖*：

- _.contains- Array Functions


  
### difference

  // Take the difference between one array and a number of other arrays.
  // Only the elements present in just the first array will remain.
  _.difference = function(array) {
    // 通过flatten展开参数列表中除了array的其他数组参数
    var rest = flatten(arguments, true, true, 1);
    // 通过_.filter对数组进行条件过滤
    return _.filter(array, function(value){
      return !_.contains(rest, value);
    });
  };
  
该函数支持多个数组作为参数，然后从第一个数组中剔除其他数组中已经包含的元素。  
  
*依赖*：

- flatten - Array Functions
- _.filter - Collection Functions
- _.contains- Array Functions  



### zip
  
    // Zip together multiple lists into a single array -- elements that share
    // an index go together.
    _.zip = function() {
      return _.unzip(arguments);
    };
  
zip函数可以接受多个多个数组对象，将所有数组对象中相应位置的值合并在一起，具体细节参考_.unzip。  
  
*依赖*：

- unzip - Array Functions  
  
  
  
### unzip
    var getLength = property('length');
    
    // Complement of _.zip. Unzip accepts an array of arrays and groups
    // each array's elements on shared indices
    _.unzip = function(array) {
      // 通过_.max(array, getLength).length获取所有数组对象中的最大长度
      var length = array && _.max(array, getLength).length || 0;
      var result = Array(length);
  
      for (var index = 0; index < length; index++) {
        // 通过_.pluck函数提取每个数组对应位置的元素
        result[index] = _.pluck(array, index);
      }
      return result;
    };
  
unzip函数的功能正好与zip相反，zip的功能是通过直接调用unzip进行实现的。
  
*依赖*：

- _.max - Collection Functions    
  
  
  
### object  

    // Converts lists into objects. Pass either a single array of `[key, value]`
    // pairs, or two parallel arrays of the same length -- one of keys, and one of
    // the corresponding values.
    _.object = function(list, values) {
      var result = {};
      for (var i = 0, length = getLength(list); i < length; i++) {
        if (values) {
          result[list[i]] = values[i];
        } else {
          result[list[i][0]] = list[i][1];
        }
      }
      return result;
    };
    
该函数的主要功能就是将数组转换成对象，values参数是可选的，所以该函数会处理两种情况：

1. values参数为空，直接传递一个单独[key,value]这种键/值对的列表

        _.object(['moe', 'larry', 'curly'], [30, 40, 50]);
        => {moe: 30, larry: 40, curly: 50}

2. 使用values参数代表一个值列表，list代表一个键列表

        _.object([['moe', 30], ['larry', 40], ['curly', 50]]);
        => {moe: 30, larry: 40, curly: 50}
        
        
### findIndex & findLastIndex

    // Generator function to create the findIndex and findLastIndex functions
    function createPredicateIndexFinder(dir) {
      return function(array, predicate, context) {
        predicate = cb(predicate, context);
        var length = getLength(array);
        var index = dir > 0 ? 0 : length - 1;
        for (; index >= 0 && index < length; index += dir) {
          if (predicate(array[index], index, array)) return index;
        }
        return -1;
      };
    }
  
    // Returns the first index on an array-like that passes a predicate test
    _.findIndex = createPredicateIndexFinder(1);
    _.findLastIndex = createPredicateIndexFinder(-1);
        
findIndex和findLastIndex函数通过不同的参数来确定查找的顺序，从而获得元素的index。    

createIndexFinder函数的返回值是一个函数，这个函数通过predicate对array进行判断，元素满足条件就返回其对于index。

createPredicateIndexFinder函数使用了cb函数，根据predicate值的不同，会得到不同的回调函数：

1. 如果predicate是一个普通值，那么会得到`_.property(value)`函数，用来获得特定下标的数组元素
2. 如果predicate是一个判断函数，那么会得到`optimizeCb(value, context, argCount)`函数

*依赖*：

- cb - 重要的内部函数
    
  

### sortedIndex  

    // Use a comparator function to figure out the smallest index at which
    // an object should be inserted so as to maintain order. Uses binary search.
    _.sortedIndex = function(array, obj, iteratee, context) {
      iteratee = cb(iteratee, context, 1);
      var value = iteratee(obj);
      var low = 0, high = getLength(array);
      while (low < high) {
        var mid = Math.floor((low + high) / 2);
        if (iteratee(array[mid]) < value) low = mid + 1; else high = mid;
      }
      return low;
    };
    
该函数在一个已经排序的数组对象中查找obj对象所在位置的index，使用二分查找法。

1. 如果iterator是一个函数，iterator将作为array排序的依据
2. 如果iterator是一个字符串，那么iteratee将会是`_.property(value)`，也就表示通过属性键来排序    

*依赖*：

- cb - 重要的内部函数
    
    
  
### range
  
    // Generate an integer Array containing an arithmetic progression. A port of
    // the native Python `range()` function. See
    // [the Python documentation](http://docs.python.org/library/functions.html#range).
    _.range = function(start, stop, step) {
      // 对参数进行判断，如果stop为null，则设置stop为start的值或者0
      if (stop == null) {
        stop = start || 0;
        start = 0;
      }
      // step的默认值为1
      step = step || 1;
  
      // 通过start/stop/step计算数组的长度
      var length = Math.max(Math.ceil((stop - start) / step), 0);
      // 创建数组
      var range = Array(length);
  
      for (var idx = 0; idx < length; idx++, start += step) {
        // 给数组元素进行赋值
        range[idx] = start;
      }
  
      return range;
    };
    
该函数主要用来创建整数列表的数组。

函数中使用了Math.ceil，是为了处理step不能整除的情况，比如(31-0)/5。