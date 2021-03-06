# Ch2 词法作用域链及跳出

## 词法作用域链

在大多数情况下，JavaScript 采用词法作用域来对标识符进行查找（LHS 和 RHS）。
所谓词法作用域即根据代码所处位置来决定其作用域，其作用域不可改变。当引擎需要对标识符进行查找时，从标识符所处的
作用域开始，逐层向外层作用域进行查找，直到查找成功或查找到最外层的作用域为止。

这样的工作方式可能会导致几个问题： `标识符覆盖` 和 `意外创建的全局变量(LHS)`

### 标识符覆盖

当内部作用域存在与外部作用域相同名的标识符时，就会产生只能调用内部标识符的情况。这便是引擎对于标识符的查找方式所致，
当引擎在内部查找到同名的标识符，就会停止查找。如果外层作用域的标识符才是真正需要的，就会直接导致外层标识符被忽略。

```javascript
let a = 10;
function readOutsideA() {
    // 内层作用域存在和外层同名的变量，查找a时，在内层查找成功便停止查找
    let a = 8;
    // 访问的是内部的 a 
    console.log(a);
} 
readOutsideA();
```

### 意外创建的全局变量（LHS)

这种情况只出现在非严格模式下对左值的查找（LHS)，大部分情况下表现为对未声明的标识符直接赋值。具体参考 [CH1](../chapter1/readme.md)

```javascript
function func() {
    a = 10;
}
func();
console.log(a);
```

## 词法作用域的跳出

绝大部分情况下，只使用词法作用域对变量的查找方式即可，使用跳出的方式，不仅会对性能产生影响，也容易产生人为的错误。基本上没有必要使用。

### eval

一句话来概括 `eval`：将字符串作为代码解释运行。 关于 `eval` 详细使用和坑点，可以参考[阮一峰ES5教程eval的使用](https://wangdoc.com/javascript/types/function.html#eval-%E5%91%BD%E4%BB%A4)

如果使用 `window.eval()` 进行动态解释执行，则会将作用域切换至 window 下。

```javascript
let a = 10;
function readOutside(){
    let a = 0;
    // 访问内部
    eval('console.log(a)');
    // 访问外部
    window.eval('console.log(a);')
}
readOutside();
```

其实个人感觉`eval`的使用场景并不多，这种将动态代码嵌入应用的场景也不是没有。比如 Android 平台下的应用，
如果开发者需要对应用进行紧急功能推送（比如双11）。现实情况下，用户可能不能及时更新最新版的 App，导致无法使用新功能（新页面）。这时可以在 App 提前内置好对紧急推送更新 url 的监听。当有紧急更新推送时，可在应用内部（不是通过平台应用商店）下载这部分更新代码（Android 下应该是下载编译好的 `.class` 文件），再动态执行这些代码，就可以做到不重新安装新 App 的情况下为旧 App 添加新的功能。

但这一功能对于运行在浏览器的 Web App 来说几乎没什么用，有什么问题不能直接部署更新解决呢？

我能想到使用 `eval` 的场合可能就是需要编写一个网页端的 JavaScript playground，使用 `eval` 来执行学习者在 `input` 中输入的代码。不过有这个需求的人应该不多吧。

### with

`with` 就更坑了，`with` 的想法本是将对象转为新的作用域，以减少代码量(`obj.a` 可用 `a` 替代)。但由于非严格模式下引擎 LHS 的原因，当访问不存在的标识符时，会在全局模式创建变量：

```javascript
const obj = {
    a: 1,
    b: 2
};
with(obj) {
    a = 2;
    b = 3;
    // 非严格模式的 LHS 查找失败，创建全局变量
    c = '变量泄露';
    console.log(obj);
}
// 变量泄露污染全局
console.log(c);
```

使用 `with` 直接带来污染全局空间的风险，严格模式下直接禁止了 `with` 的使用，因此不要在任何代码中使用 `with` 。

