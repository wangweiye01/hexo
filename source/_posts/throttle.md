# 函数节流

## 应用场景

在浏览器 DOM 事件里面，有一些事件会随着用户的操作不间断触发。比如：重新调整浏览器窗口大小（resize），浏览器页面滚动（scroll），鼠标移动（mousemove）。也就是说用户在触发这些浏览器操作的时候，如果脚本里面绑定了对应的事件处理方法，这个方法就不停的触发。

这并不是我们想要的，因为有的时候如果事件处理方法比较庞大，DOM 操作比如复杂，还不断的触发此类事件就会造成性能上的损失，导致用户体验下降（UI 反映慢、浏览器卡死等）。所以通常来讲我们会给相应事件添加延迟执行的逻辑。

## 代码

``` html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>函数节流or函数防抖</title>
</head>

<body>
    <div>《JavaScript高级程序设计》中把函数防抖的命名错了，导致以后很多人都理解反了。<br />
        问题：我们要做一个input，根据用户的输入实时搜索，该怎么做？
        <br />
        <h1>实例1：</h1>
        <img src="a.gif" />
        <br />
        <h1>实例2：</h1>
        <img src="b.gif" />
    </div>
    <hr />
    <div id="demo"></div>
    <script>
        var COUNT = 0;
        var demo = document.getElementById('demo');

        function testFn() {
            demo.innerHTML += 'testFn 被调用了 ' + ++COUNT + '次<br>';
        }

        // version0: 《JavaScript高级程序设计》中的方法，把定时器ID存为函数的一个属性
        /*
        function throttle(method, context) {
            clearTimeout(method.tid);
            method.tid = setTimeout(function () {
                method.call(context);
            }, 100);
        }

        window.onresize = function () {
            throttle(testFn);
        }
        */

        // version1: -> 错误 timer不是相对全局的变量每次resize会生成一个timer
        /*
        window.onresize = function () {
            var timer = null;
            clearTimeout(timer);

            timer = setTimeout(function () {
                testFn();
            }, 100);
        };
        */

        // version2: -> 正确， 但是会多添加一个相对全局的变量，有可能影响业务逻辑
        /*
        var timer = null;
        window.onresize = function () {
            clearTimeout(timer);
            timer = setTimeout(function() {
                testFn();
            }, 100);
        };
        */

        // version3: -> 正确，使用闭包。但是到目前为至还有一个问题，如果不间断的触发resize的话fn是永远不会执行的
        /**
         * 函数节流方法
         * @param Function fn 延时调用函数
         * @param Number delay 延迟多长时间
         * @return Function 延迟执行的方法
         */

        /*
        var throttle = function (fn, delay) {
            var timer = null;

            return function () {
                clearTimeout(timer);
                timer = setTimeout(function () {
                    fn();
                }, delay);
            }
        };
        */

        // 第一种调用方式
        /*
        var f = throttle(testFn, 200);
        window.onresize = function () {
            f();
        };
        */

        // 第二种调用方式
        /* window.onresize = throttle(testFn, 200);*/

        // versin4：最终模式
        var throttle = function (fn, delay, atleast) {
            var timer = null;
            var previous = null;

            return function () {
                var now = +new Date();

                if (!previous) previous = now;

                if (atleast && now - previous > atleast) {
                    fn();
                    // 重置上一次开始时间为本次结束时间
                    previous = now;
                    clearTimeout(timer);
                } else {
                    clearTimeout(timer);
                    timer = setTimeout(function () {
                        fn();
                        previous = null;
                    }, delay);
                }
            }
        };

        // atleast参数选填
        window.onresize = throttle(testFn, 200, 1000);
        // window.onresize = throttle(testFn, 200);
    </script>
</body>

</html>
```

testcase2

``` html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>函数节流or函数防抖测试用例</title>
</head>

<body>
    <div style="height:5000px">
        <div id="demo" style="position:fixed;"></div>
    </div>
    <script>
        var COUNT = 0,
            demo = document.getElementById('demo');

        function testFn() {
            demo.innerHTML += 'testFn 被调用了 ' + ++COUNT + '次<br>';
        }

        var throttle = function (fn, delay, atleast) {
            var timer = null;
            var previous = null;

            return function () {
                var now = +new Date();

                if (!previous) previous = now;
                if (atleast && now - previous > atleast) {
                    fn();
                    // 重置上一次开始时间为本次结束时间
                    previous = now;
                    clearTimeout(timer);
                } else {
                    clearTimeout(timer);
                    timer = setTimeout(function () {
                        fn();
                        previous = null;
                    }, delay);
                }
            }
        };

        // testCase1
        window.onscroll = throttle(testFn, 200);

        // testCase2
        // window.onscroll = throttle(testFn, 500, 1000);
    </script>
</body>

</html>
```
