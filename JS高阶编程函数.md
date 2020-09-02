### 1. JS高阶编程函数：惰性函数

```javascript
/*
 * JS高阶编程函数：惰性函数
 *   + 懒
 *   + 能执行一次的绝对不会执行第二次 
 */

// 这样编写，我们每一次执行方法都需要处理兼容：其实这种操作是没必要的，第一次执行已经知道兼容情况了，后期再执行方法（浏览器也没有刷新、也没有换浏览器）兼容校验是没必要处理的
/* function getCss(element, attr) {
    if (window.getComputedStyle) {
        return window.getComputedStyle(element)[attr];
    }
    // IE6~8
    return element.currentStyle[attr];
} */

// 这是一种方案：页面一加载就把是否兼容处理了，后期执行方法直接基于变量的值判断使用哪个办法即可
/* let isCompatible = 'getComputedStyle' in window;
function getCss(element, attr) {
    if (isCompatible) {
        return window.getComputedStyle(element)[attr];
    }
    // IE6~8
    return element.currentStyle[attr];
} */

// 惰性思想来解决:函数重构
// + getCss是全局函数
// + 第一次执行方法会产生闭包
function getCss(element, attr) {
    /*
     * EC(1) 闭包
     *    element = BODY
     *    attr = 'width'
     *    作用域链:<EC(1),EC(G)>
     */
    if (window.getComputedStyle) {
        // 把全局的getCss重构成为具体的小函数
        getCss = function (element, attr) {
            return window.getComputedStyle(element)[attr];
        };
    } else {
        getCss = function (element, attr) {
            return element.currentStyle[attr];
        };
    }
    // 重构后首先执行一次：确保第一次调用getCss也可以获取到自己想要的结果
    return getCss(element, attr);
}

console.log(getCss(document.body, 'width'));
console.log(getCss(document.body, 'padding'));
// 第二次执行：执行是的重构后的小函数，这样告别了兼容校验的操作
console.log(getCss(document.body, 'margin'));
```



###  2.JS高阶函数：柯里化函数

```javascript
/*
 * JS高阶函数：柯里化函数
 *   + 预处理思想 
 *   + 应用的也是闭包的机制
 */
function fn(...args) {
    // ES6中的剩余运算符: ...args，把传递进来的实参获取到（排除基于其他形参获取的信息）
    console.log(args); //数组集合
    console.log(arguments); //类数组集合
    return function anonymous() {

    };
}

// 柯里化函数：第一次执行大函数，形成一个闭包（原因：返回一个小函数），把一些信息存储到闭包中（传递的实参信息或当前闭包中声明的一些私有变量等信息）；等到后面需要把返回的小函数anonymous执行，遇到一些非自己私有变量，则向其上级上下文中中查找（也就是把之前存储在闭包中的信息获取到）；
function fn(...outerArgs) {
    // outerArgs存放第一次执行fn传递的实参信息 [1,2]

    return function anonymous(...innerArgs) {
        // innerArgs存放第二次执行匿名函数传递的实参信息 [3]

        let arr = outerArgs.concat(innerArgs);
        // let arr=[1,2,3];
        return arr.reduce(function (total, item) {
            return total + item;
        });
    };
}

let res = fn(1, 2)(3);
console.log(res); //=>6  1+2+3
```

### 3 组合函数  compose函数

```javascriptj
/* 
    在函数式编程当中有一个很重要的概念就是函数组合， 实际上就是把处理数据的函数像管道一样连接起来， 然后让数据穿过管道得到最终的结果。 例如：
    const add1 = (x) => x + 1;
    const mul3 = (x) => x * 3;
    const div2 = (x) => x / 2;
    div2(mul3(add1(add1(0)))); //=>3
​
    而这样的写法可读性明显太差了，我们可以构建一个compose函数，它接受任意多个函数作为参数（这些函数都只接受一个参数），然后compose返回的也是一个函数，达到以下的效果：
    const operate = compose(div2, mul3, add1, add1)
    operate(0) //=>相当于div2(mul3(add1(add1(0)))) 
    operate(2) //=>相当于div2(mul3(add1(add1(2))))
​
    简而言之：compose可以把类似于f(g(h(x)))这种写法简化成compose(f, g, h)(x)，请你完成 compose函数的编写 
*/

const add1 = x => x + 1;
const mul3 = x => x * 3;
const div2 = x => x / 2;

/* function compose(...funcs) {
    // funcs存储的是最后需要处理的函数集合
    return function operate(x) {
        // x存储初始第一个函数执行需要的实参
        if (funcs.length === 0) return x;
        if (funcs.length === 1) return funcs[0](x);

        // 把数据倒过来或者用reduceRight
        funcs.reverse();
        let first = funcs[0](x);
        funcs = funcs.slice(1);
        return funcs.reduce((result, item) => {
            return item(result);
        }, first);
    };
} */

/* function compose(...funcs) {
    return function operate(x) {
        if (funcs.length === 0) return x;
        if (funcs.length === 1) return funcs[0](x);

        let n = 0;
        funcs.reverse();
        return funcs.reduce((result, item) => {
            if ((++n) === 1) {
                // 第一次处理 result第一个函数 item第二个函数
                return item(result(x));
            }
            return item(result);
        });
    };
} */

// redux源码中的处理方案
function compose(...funcs) {
    if (funcs.length === 0) {
        return x => {
            return x;
        };
    }

    if (funcs.length === 1) {
        return funcs[0];
    }

    return funcs.reduce((a, b) => {
        return (...args) => {
            return a(b(...args));
        };
    });
}

let operate = compose(div2, mul3, add1, add1);
console.log(operate(0)); //=>3

operate = compose();
console.log(operate(0)); //=>0

operate = compose(add1);
console.log(operate(0)); //=>1
```

### 4.函数防抖和节流

```javascript
点击事件一般用防抖
滚动事件和输入过程（keydown）用节流
/*
 * 函数的防抖（防止老年帕金森）：对于频繁触发某个操作，我们只识别一次(只触发执行一次函数)
 *   @params
 *      func[function]:最后要触发执行的函数
 *      wait[number]:“频繁”设定的界限
 *      immediate[boolean]:默认多次操作，我们识别的是最后一次，但是immediate=true，让其识别第一次
 *   @return
 *      可以被调用执行的函数
 */
// 主体思路：在当前点击完成后，我们等wait这么长的时间，看是否还会触发第二次，如果没有触发第二次，属于非频繁操作，我们直接执行想要执行的函数func；如果触发了第二次，则以前的不算了，从当前这次再开始等待...
/* function debounce(func, wait = 300, immediate = false) {
    let timer = null;
    return function anonymous(...params) {
        let now = immediate && !timer;

        // 每次点击都把之前设置的定时器清除
        clearTimeout(timer);

        // 重新设置一个新的定时器监听wait时间内是否触发第二次
        timer = setTimeout(() => {
            // 手动让其回归到初始状态
            timer = null;
            // wait这么久的等待中，没有触发第二次
            !immediate ? func.call(this, ...params) : null;
        }, wait);
console.log(timer);
        // 如果是立即执行
        now ? func.call(this, ...params) : null;
    };
} */

/*
 * 函数节流：在一段频繁操作中，可以触发多次，但是触发的频率由自己指定
 *   @params
 *      func[function]:最后要触发执行的函数
 *      wait[number]:触发的频率
 *   @return
 *      可以被调用执行的函数
 */
/* function throttle(func, wait = 300) {
    let timer = null,
        previous = 0; // 记录上一次操作的时间
    return function anonymous(...params) {
        let now = new Date(),
            remaining = wait - (now - previous); //记录还差多久达到我们一次触发的频率
        if (remaining <= 0) {
            // 两次操作的间隔时间已经超过wait了
            window.clearTimeout(timer);
            timer = null;
            previous = now;
            func.call(this, ...params);
        } else if (!timer) {
            // 两次操作的间隔时间还不符合触发的频率
            timer = setTimeout(() => {
                timer = null;
                previous = new Date();
                func.call(this, ...params);
            }, remaining);
        }
    };
} */

function handle() {
    console.log('OK');
}
// window.onscroll = handle; //每一次滚动过程中，浏览器有最快反应时间(5~6ms 13~17ms)，只要反应过来就会触发执行一次函数（此时触发频率5ms左右）
window.onscroll = throttle(handle); // 我们控制频率为300ms触发一次
// window.onscroll = function anonymous() {}; //每隔5ms触发一次匿名函数,我们自己可以匿名函数中控制执行handle的频率




// submit.onclick = handle;  //点击一次触发执行一次handle，频繁触发，频繁执行handle
// submit.onclick = function anonymous() {
//     // 在匿名函数中我们控制hanlde只执行一次
// };
submit.onclick = debounce(handle, 1000, true);
```

