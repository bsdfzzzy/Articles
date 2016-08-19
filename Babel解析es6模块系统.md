##Babel解析es6模块系统

    在这里分析的内容基于babel配置如下：
    {
    	"presets": [
			"es2015",
	   		"react",
			"stage-2"
        ],
        "plugins":[],
    }
    并且假设您掌握export和import的用法，已经全局安装babel-cli并掌握基本用法

###最简单的export和import
我们创建两个文件，export.js和import.js。
首先，在export.js中写入如下代码：

    export const a = 'a'
在import.js中写入如下代码：

    import {a} from './export'
    console.log(a)
然后分别运行babel export.js

    'use strict';

    Object.defineProperty(exports, "__esModule", {
        value: true
    });
    var a = exports.a = 'a';
和babel import.js
    
    'use strict';

    var _export = require('./export');

    console.log(_export.a);
在这里我们用最简单的export和import开头，可以看到在编译之后，除了加上了生产环境推荐使用的'use strict'之外，babel只是帮助我们把es6的模块标准转化为了commonJS的写法，我们先不看export里面的__esModule，下面会讲。
###多加几个export
这次我们加一段代码在export.js

    const b = 'b'
    export default b
同样在import.js里加入

    import b from './export'
    console.log(b)
先使用babel-node import.js看看输出来确保代码没有问题：

    a
    b
OK，输出没有问题，再来看看这次babel把代码变成什么样了，
首先是export.js

    'use strict';

    Object.defineProperty(exports, "__esModule", {
        value: true
    });
    var a = exports.a = 'a';

    var b = 'b';
    exports.default = b;
代码没有怎么变，只是多加了一个变量b，并且在exports里加入了defualt属性，将b的值赋给它。再来看看import.js

    'use strict';

    var _export = require('./export');

    var _export2 = _interopRequireDefault(_export);

    function _interopRequireDefault(obj) { 
        return obj && obj.__esModule ? obj : { default: obj }; 
    }

    console.log(_export.a);
    console.log(_export2.default);
我们发现import里面，babel添加了一个函数，它大概是个什么意思呢？这里就会用到export里面添加的对象属性__esModule。
在export里，babel用Object.defineProperty来为默认对象exports添加了一个不可变的属性`__`esModule，然后在import里进行判断：如果判断在引入的对象中这个属性的值存在且为true，那么直接返回，否则将返回一个带有default属性的对象，之后的调用采用default来调用需要的值。那为什么一定要要加入default属性呢？在我们开发时，我们是不会用babel将所有第三方包也编译一遍的，这个时候有些包被引入时没有default对象但是es6还是会通过default去调用，因此会抛出error，此时，babel将没有被babel编译的引入对象添加到default属性中来避免此错误。
###再来看看function怎么处理
在export.js中加入
    
    export function foo(){
        console.log('hello')
    }
在import.js中加入

    import {foo} from './export.js'
    foo()
跑一边没有问题，看看babel编译如何：
export.js

    'use strict';

    Object.defineProperty(exports, "__esModule", {
        value: true
    });
    exports.foo = foo;
    var a = exports.a = 'a';

    var b = 'b';
    exports.default = b;
    function foo() {
        console.log('hello');
    }
import.js

    'use strict';

    var _export = require('./export');

    var _export2 = _interopRequireDefault(_export);

    function _interopRequireDefault(obj) { 
        return obj && obj.__esModule ? obj : { default: obj }; 
    }

    (0, _export.foo)();

    console.log(_export.a);
    console.log(_export2.default);
 这里想说的一点是在import里面的(0, _export.foo)()，这个逗号表达式等价于执行_export.foo()但是执行时的上下文环境会被绑定到全局对象身上，所以实际上真正等价于执行 _export.foo.call(global)
###最后看一下import *
在import.js中把所有的import去掉改为

    import * as item from './export'
下面的调用也要改，就不细说，当然在输出没有问题的情况下，看一下babel对import编译后的代码：
    
    'use strict';

    var _export = require('./export');

    var item = _interopRequireWildcard(_export);

    function _interopRequireWildcard(obj) { 
        if (obj && obj.__esModule) { 
            return obj; 
        } else { 
            var newObj = {}; 
            if (obj != null) { 
                for (var key in obj) { 
                    if (Object.prototype.hasOwnProperty.call(obj, key)) {
                        newObj[key] = obj[key];
                    } 
                } 
            } 
            newObj.default = obj; return newObj; 
        } 
    }

    item.foo(); //import {a} from './export'

    console.log(item.a);
    console.log(item.default);
这个只是作为参考，相信看了上面的解释，这段代码也很容易懂啦，就留给大家作为复习！
最后，在模块这里还有许多细节，希望大家在测试代码时自行打印exports和module来学习。