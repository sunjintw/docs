#初识Jasmine
January 17, 2016

##Agendar:

1. 发人深省的引子
2. 简单粗暴的介绍
3. 那么我们开始吧
4. 我该如何使用呢
5. 举起一个小栗子 
6. Jasmine与他的好基友们

##引子

犹记得 Release 1c时大家紧张与郁闷的光景，我们为此做了很多，但请不要忘记故事的开始：

```javascript
ko.bindingHandlers.truncate = {
    update: function (element, valueAccessor, allBindingsAccessor) {
        var value = ko.utils.unwrapObservable(valueAccessor()),
        length = ko.utils.unwrapObservable(allBindingsAccessor().length) || ko.bindingHandlers.truncate.defaultLength,
        truncatedValue = value.length > length ? value.substring(0, Math.min(value.length, length)) + " ..." : value;
        $(element).attr('title', value);

        ko.bindingHandlers.text.update(element, function () { return truncatedValue; });
    },
    defaultLength: 15
};
```
尽管我们最终解决了代码本身的问题，但是如果我们不做些什么的话，可预见的将来一定像那可巧克力一样，让你不知道下一颗是什么滋味。我们不缺乏解决问题的能力，我们缺少的只是更早发现问题的方法，而自动化测试可以帮助我们，所以让我们欢迎 **Jasmine**.

##介绍

Jasmine 是一款 **JavaScript 测试框架**，它不依赖于其他任何 JavaScript 组件。它有**干净清晰的语**法，让你可以很简单的写出测试代码。

##开始

1. 前往 [Jasmine 官网](https://github.com/jasmine/jasmine/releases)下载standalone版本。
![image](http://)
2. 将jasmine-standalone-xxx.zip解压，运行SpecRunner.html，你会看到下面的界面：
![image](http://)
3. 打开SpecRunner.html，我们看看它的用法:
4. 我们看到页面中有1个css和7个js文件，它们有什么用呢？
```html
<html>
<head>
    <meta charset="utf-8">
    <title>Jasmine Spec Runner v2.4.1</title>

    <link rel="shortcut icon" type="image/png" href="lib/jasmine-2.4.1/jasmine_favicon.png">
    <link rel="stylesheet" href="lib/jasmine-2.4.1/jasmine.css">

    <script src="lib/jasmine-2.4.1/jasmine.js"></script>
    <script src="lib/jasmine-2.4.1/jasmine-html.js"></script>
    <script src="lib/jasmine-2.4.1/boot.js"></script>

    <!-- include source files here... -->
    <script src="src/Player.js"></script>
    <script src="src/Song.js"></script>

    <!-- include spec files here... -->
    <script src="spec/SpecHelper.js"></script>
    <script src="spec/PlayerSpec.js"></script>

</head>
<body>
</body>
</html>
```

```
jasmine.css -->测试界面css样式  
jasmine.js -->核心文件用于执行单元测试的类库  
jasmine-html.js -->用于显示单元测试结果的类库  
boot.js -->用于初始化单元测试所需的执行环境类库  
Player.js, Song.js --> 我们自己的JS源代码  
SpecHelper.js, PlayerSpec.js --> 单元测试代码  
```

