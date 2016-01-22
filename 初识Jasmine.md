# 初识Jasmine
<small><small>*Jin Sun, January 17, 2016*</small></small>
## 我们要聊些什么:
1. 一个不错的引子
2. 简单粗暴的介绍
3. 那么我们开始吧
4. 我该如何使用呢
5. 与KnockoutJS不得不说的故事

## 引子

犹记得 Release 1c时大家紧张与郁闷的光景，之后我们为此做了很多，但请不要p忘记故事的开始：

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
尽管我们最终解决了代码本身的问题，但是如果我们没有一些新的措施防范相同的错误出现的话，可预见的将来一定像巧克力一样，让你不知道下一颗是什么滋味。我们不缺乏解决问题的能力，我们缺少的只是更早发现问题的方法，而自动化测试可以帮助我们，所以让我们欢迎 <big><big>**Jasmine**</big></big>.

## 介绍

Jasmine 是一款 **JavaScript 测试框架**，它不依赖于其他任何 JavaScript 组件。它有**干净清晰的语**法，让你可以很简单的写出测试代码。

## 开始

1. 前往 [Jasmine 官网](https://github.com/jasmine/jasmine/releases)下载standalone版本。
![image](https://raw.githubusercontent.com/sunjintw/docs/master/img/jasmine_download.png)
2. 将jasmine-standalone-xxx.zip解压，运行SpecRunner.html，你会看到下面的界面：
![image](https://raw.githubusercontent.com/sunjintw/docs/master/img/jasmine_runner.png)

3. 打开SpecRunner.html，我们看看它的用法:

```html
<html>
<head>
    <meta charset="utf-8">
    <title>Jasmine Spec Runner v2.4.1</title>

    <link rel="shortcut icon" type="image/png" href="lib/jasmine-2.4.1/jasmine_favicon.png">
    <link rel="stylesheet" href="lib/jasmine-2.4.1/jasmine.css">
    <!-- 测试界面css样式 -->
    <script src="lib/jasmine-2.4.1/jasmine.js"></script>
    <!-- 核心文件用于执行单元测试的类库 -->
    <script src="lib/jasmine-2.4.1/jasmine-html.js"></script>
    <!-- 用于显示单元测试结果的类库 -->
    <script src="lib/jasmine-2.4.1/boot.js"></script>
    <!-- 用于初始化单元测试所需的执行环境类库 -->

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

## 使用

打开测试文件PlayerSpec.js，我们会看到describe，it，beforeEach，afterEach，expect，toBe....都是些什么意思呢？

    Jasmine有四个核心概念：*分组(Suites)*、用例(Specs)、期望(Expectations)、匹配(Matchers).

#### Suites
Suites可以理解为一组测试用例，使用全局的Jasmin函数describe 创建。describe函数接受两个参数，一个字符串和一个函数。字符串是这个Suites的名字或标题（通常描述下测试内容），函数是实现Suites的代码块。
#### Specs
Specs可以理解为一个测试用例，使用全局的Jasmin函数it创建。和describe一样接受两个参数，一个字符串和一个函数，函数就是要执行的测试代码，字符串就是测试用例的名字。一个Spec可以包含多个expectations来测试代码。
#### Expectations
Expectations由expect 函数创建。接受一个参数。和Matcher一起联用，设置测试的预期值。

     在分组(describe)中可以写多个测试用例(it)，也可以再进行分组(describe)，在测试用例(it)中定义期望表达式(expect)和匹配判断(toBe*)。看一个简单的Demo:
```javascript
describe("A suite", function() {//suites  
    var a;
    it("A spec", function() {//spec
      a = true;
      expect(a).toBe(true);//expectations
    });

    describe("a suite", function() {//inner suites
           it("a spec", function() {//spec
           expect(a).toBe(true);//expectations
        });
  });
});
```

#### Matchers
Matcher实现一个“期望值”与“实际值”的对比，如果结果为true，则通过测试，反之，则失败。每一个matcher都能通过not执行否定判断。

简单的matchers：
```javascript
expect(a).toBe(true);//期望变量a为true  
expect(a).toEqual(true);//期望变量a等于true  
expect(a).toMatch(/reg/);//期望变量a匹配reg正则表达式，也可以是字符串  
expect(a.foo).toBeDefined();//期望a.foo已定义  
expect(a.foo).toBeUndefined();//期望a.foo未定义  
expect(a).toBeNull();//期望变量a为null  
expect(a.isMale).toBeTruthy();//期望a.isMale为真  
expect(a.isMale).toBeFalsy();//期望a.isMale为假  
expect(true).toEqual(true);//期望true等于true  
expect(a).toBeLessThan(b);//期望a小于b  
expect(a).toBeGreaterThan(b);//期望a大于b  
expect(a).toThrowError(/reg/);//期望a方法抛出异常，异常信息可以是字符串、正则表达式、错误类型以及错误类型和错误信息  
expect(a).toThrow();//期望a方法抛出异常  
expect(a).toContain(b);//期望a(数组或者对象)包含b  
```

#### 其他matchers：
    
**jasmine.any(Class)**--传入构造函数或者类返回数据类型作为期望值，返回true表示实际值和期望值数据类型相同:
```javascript
it("matches any value", function() {  
    expect({}).toEqual(jasmine.any(Object));
    expect(12).toEqual(jasmine.any(Number));
});
```
**jasmine.anything()**--如果实际值不是null或者undefined则返回true:
```javascript
it("matches anything", function() {  
    expect(1).toEqual(jasmine.anything());
});
```
**jasmine.objectContaining({key:value})**--实际数组只要匹配到有包含的数值就算匹配通过:
```javascript
foo = {  
      a: 1,
      b: 2,
      bar: "baz"
};
expect(foo).toEqual(jasmine.objectContaining({bar: "baz"}));
```
**jasmine.arrayContaining([val1,val2,...])**--stringContaining可以匹配字符串的一部分也可以匹配对象内的字符串:
```javascript
expect({foo: 'bar'}).toEqual({foo: jasmine.stringMatching(/^bar$/)});  
expect('foobarbaz').toEqual({foo: jasmine.stringMatching('bar')}); 
```
<small>*Jasmine还支持自定义Matchers，今天我们先不展开了.*</small>
#### Setup and Teardown
为了在复杂的测试用例中更加便于组装和拆卸，Jasmine提供了四个函数：
```javascript
beforeEach(function)  //在每一个测试用例(it)执行之前都执行一遍beforeEach函数；  
afterEach(function)  //在每一个测试用例(it)执行完成之后都执行一遍afterEach函数；  
beforeAll(function)  //在所有测试用例执行之前执行一遍beforeAll函数；  
afterAll(function)  //在所有测试用例执行完成之后执行一遍afterAll函数； 
```
对照一下结果看一下下面的例子就一目了然啦：
```javascript
describe("A spec using beforeEach and afterEach", function() {  
  var foo = 0;
  beforeEach(function() {
    foo += 1;
    
    le.log("I am beforEach");
  });
  afterEach(function() {
    foo = 0;
    console.log("I am afterEach");
  });
  it("A spec1", function() {
    expect(foo).toEqual(1);
    console.log("I am spec1");
  });
  it("A spec2", function() {
    expect(foo).toEqual(1);
    expect(true).toEqual(true);
    console.log("I am spec2");
  });
});
describe("A spec using beforeAll and afterAll", function() {  
  var foo;
  beforeAll(function() {
    foo = 1;
    console.log("I am beforAll");
  });
  afterAll(function() {
    foo = 0;
    console.log("I am afterAll");
  });
  it("A spec1", function() {
    expect(foo).toEqual(1);
    foo += 1;
    console.log("I am A spec1");
  });
  it("A spec2", function() {
    expect(foo).toEqual(2);
    console.log("I am A spec2");
  });
});
```
最终输出：
![image](https://raw.githubusercontent.com/sunjintw/docs/master/img/jasmine_test_result.png)
#### Suites禁用和Specs挂起
Jasmine提供xdescrib和xit方法用于屏蔽测试用例，xdescribe里面的定义的it、beforeEach、afterEach等之类的方法不会执行，describe里面定义的xit不会执行。
测试文档遇到用例挂起的地方就不会执行期望的匹配表达式，有三种使用方法：
```javascript
describe("Pending specs", function() {  
  xit("can be declared 'xit'", function() {//第一种，使用xit将测试用例直接屏蔽
    expect(true).toBe(false);
  });
 it("can be declared with 'it' but without a function");//第二种，只声明it，不定义回调函数
 it("can be declared by calling 'pending' in the spec body", function() {
    expect(true).toBe(false);
    pending('this is why it is pending');//第三种，使用pending函数
  });
});
```
#### Spy追踪
Jasmine具有函数的追踪和反追踪的双重功能，这东西就是Spy！
Spy能够存储任何函数调用记录和传入的参数，Spy只存在于describe和it中，在spec执行完之后销毁。说的这么晦涩，还是直接上例子吧：
```javascript
describe("A spy", function() {  
  var foo, bar = null;
  beforeEach(function() {
    foo = {
      setBar: function(value) {
        bar = value;
      }
    };
    spyOn(foo, 'setBar');//给foo对象的setBar函数绑定追踪
    foo.setBar(123);
    foo.setBar(456, 'another param');
  });
  it("tracks that the spy was called", function() {
    expect(foo.setBar).toHaveBeenCalled();//toHaveBeenCalled用来匹配测试函数是否被调用过
  });
  it("tracks all the arguments of its calls", function() {
    expect(foo.setBar).toHaveBeenCalledWith(123);//toHaveBeenCalledWith用来匹配测试函数被调用时的参数列表
    expect(foo.setBar).toHaveBeenCalledWith(456, 'another param');//期望foo.setBar已经被调用过，且传入参数为[456, 'another param']
  });
  it("stops all execution on a function", function() {
    expect(bar).toBeNull();//用例没有执行foo.setBar,bar为null
  });
});
```
**and.callThrough**--spy链式调用and.callThrough后，在获取spy的同时，调用实际的函数，看示例：
```javascript
describe("A spy, when configured to call through", function() {  
  var foo, bar, fetchedBar;
  beforeEach(function() {
    foo = {
      setBar: function(value) {
        bar = value;
      },
      getBar: function() {
        return bar;
      }
    };
    spyOn(foo, 'getBar').and.callThrough();//调用and.callThrough方法
    foo.setBar(123);
    fetchedBar = foo.getBar();//因为and.callThrough，这里执行的是foo.getBar方法，而不是spy的方法
  });
  it("tracks that the spy was called", function() {
    expect(foo.getBar).toHaveBeenCalled();
  });
  it("should not effect other functions", function() {
    expect(bar).toEqual(123);
  });
  it("when called returns the requested value", function() {
    expect(fetchedBar).toEqual(123);
  });
});
```
**and.returnValue**--spy链式调用and.returnValue 后，任何时候调用该方法都只会返回指定的值，比如：
```javascript
describe("A spy, when configured to fake a return value", function() {  
  var foo, bar, fetchedBar;
  beforeEach(function() {
    foo = {
      setBar: function(value) {
        bar = value;
      },
      getBar: function() {
        return bar;
      }
    };
    spyOn(foo, "getBar").and.returnValue(745);//指定返回值为745
    foo.setBar(123);
    fetchedBar = foo.getBar();
  });
  it("tracks that the spy was called", function() {
    expect(foo.getBar).toHaveBeenCalled();
  });
  it("should not effect other functions", function() {
    expect(bar).toEqual(123);
  });
  it("when called returns the requested value", function() {
    expect(fetchedBar).toEqual(745);//默认返回指定的returnValue值
  });
});
```
**and.callFake**--spy链式添加and.callFake相当于用新的方法替换spy的方法，比如：
```javascript
describe("A spy, when configured with an alternate implementation", function() {  
  var foo, bar, fetchedBar;
  beforeEach(function() {
    foo = {
      setBar: function(value) {
        bar = value;
      },
      getBar: function() {
        return bar;
      }
    };
    spyOn(foo, "getBar").and.callFake(function() {//指定callFake方法
      return 1001;
    });
    foo.setBar(123);
    fetchedBar = foo.getBar();
  });
  it("tracks that the spy was called", function() {
    expect(foo.getBar).toHaveBeenCalled();
  });
  it("should not effect other functions", function() {
    expect(bar).toEqual(123);
  });
  it("when called returns the requested value", function() {
    expect(fetchedBar).toEqual(1001);//执行callFake方法，返回1001
  });
});
```
**and.throwError**--spy链式调用and.callError后，任何时候调用该方法都会抛出异常错误信息:
```javascript
describe("A spy, when configured to throw an error", function() {  
  var foo, bar;
  beforeEach(function() {
    foo = {
      setBar: function(value) {
        bar = value;
      }
    };
    spyOn(foo, "setBar").and.throwError("error");//指定throwError
  });
  it("throws the value", function() {
    expect(function() {
      foo.setBar(123)
    }).toThrowError("error");//抛出错误异常
  });
});
```
**and.stub**--spy恢复到原始状态，不执行任何操作。直接看下代码:
```javascript
describe("A spy", function() {  
  var foo, bar = null;
  beforeEach(function() {
    foo = {
      setBar: function(value) {
        bar = value;
      }
    };
    spyOn(foo, 'setBar').and.callThrough();
  });
  it("can call through and then stub in the same spec", function() {
    foo.setBar(123);
    expect(bar).toEqual(123);
    foo.setBar.and.stub();//把foo.setBar设置为原始状态，and.callThrough无效
    bar = null;
    foo.setBar(123);//执行赋值无效
    expect(bar).toBe(null);
  });
});
```

##### Spy的其他方法
```javascript
.calls.any():记录spy是否被访问过，如果没有，则返回false，否则，返回true；
.calls.count():记录spy被访问过的次数；
.calls.argsFor(index):返回指定索引的参数；
.calls.allArgs():返回所有函数调用的参数记录数组；
.calls.all ():返回所有函数调用的上下文、参数和返回值；
.calls.mostRecent():返回最近一次函数调用的上下文、参数和返回值；
.calls.first():返回第一次函数调用的上下文、参数和返回值；
.calls.reset():清除spy的所有调用记录；
```

#### 虚拟定时器
Jasmine Clock 使用setTimeout 和setInterval 来声明定时的回调操作。它使回调函数同步执行，当Clock的tick时间超过timer的时间，回调函数会被触发一次。
Jasmine Clock使用jasmine.clock().install 在需要调用timer函数的spec和suite中初始化。在执行完测试的时候，一定要卸载Clock来还原timer函数。使用jasmine.clock().tick来推进时间以使注册的回调触发。
**Install**--在Spec或者Suite中安装Jasmine Clock:
```javascript
beforeEach(function() {  
    timerCallback = jasmine.createSpy("timerCallback");
    jasmine.clock().install();
});
```
**Uninstall**--保证使用完成后，切记要关闭Jasmine Clock：
```javascript:
afterEach(function() {  
    jasmine.clock().uninstall();
});
```
**Tick**--使用jasmine.clock().tick来计时，一旦累计的时间达到setTimeout或者setInterval中指定的延时时间，则触发回调函数：
```javascript
describe("Manually ticking the Jasmine Clock", function() {
  var timerCallback;
  beforeEach(function() {
    timerCallback = jasmine.createSpy("timerCallback");
    jasmine.clock().install();
  });
  afterEach(function() {
    jasmine.clock().uninstall();
  });
  it("causes a timeout to be called synchronously", function() {
    setTimeout(function() {
      timerCallback();
    }, 100);//声明回调函数tick到100ms就触发

    expect(timerCallback).not.toHaveBeenCalled();


    jasmine.clock().tick(101);//tick 101 会触发上面注册的setTimeout

    expect(timerCallback).toHaveBeenCalled();
  });
});
```
**Mock Date**
```javascript
describe("Mocking the Date object", function(){  
    it("mocks the Date object and sets it to a given time", function() {
      var baseTime = new Date(2016, 1, 27);//new一个指定的时间，没有参数则返回当前时间
        jasmine.clock().mockDate(baseTime);//构造一个虚拟的当前时间
        jasmine.clock().tick(50);//让虚拟的当前时间快进50ms
        expect(new Date().getTime()).toEqual(baseTime.getTime() + 50);
    });
  });
});
```
#### 异步支持
Jasmine支持测试需要执行异步操作的specs，调用beforeEach , it , 和afterEach 的时候，可以带一个可选的参数done ，当spec执行完成之后需要调用done 来告诉Jasmine异步操作已经完成。
默认Jasmine的超时时间是5s，可以通过全局的jasmine.DEFAULTTIMEOUTINTERVAL 设置。
```javascript
describe("Asynchronous specs", function() {  
  var value;
  beforeEach(function(done) {//传入done参数表示要执行异步操作
    setTimeout(function() {
      value = 0;
      done();//执行done()函数通知it异步操作已经执行完毕，必须执行
    }, 10000);
  });
 it("should support async execution of test preparation and expectations", function(done) {//传入done参数表示要执行异步操作
    value++;
    expect(value).toBeGreaterThan(0);
    done();//执行done()函数通知it异步操作已经执行完毕，必须执行
  });
});
```
#### Ajax
Jasmine拥有一个用于测试Ajax请求的plug-in：
```javascript
describe("mocking ajax", function() {
  describe("suite wide usage", function() {
    beforeEach(function() {
      jasmine.Ajax.install();
    });
    afterEach(function() {
      jasmine.Ajax.uninstall();
    });

    it("specifying response when you need it", function() {
      var doneFn = jasmine.createSpy("success");
      var xhr = new XMLHttpRequest();
      xhr.onreadystatechange = function(args) {
        if (this.readyState == this.DONE) {
          doneFn(this.responseText);
        }
      };

      xhr.open("GET", "/some/cool/url");
      xhr.send();
      expect(jasmine.Ajax.requests.mostRecent().url).toBe('/some/cool/url');
      expect(doneFn).not.toHaveBeenCalled();
      jasmine.Ajax.requests.mostRecent().response({
        "status": 200,
        "contentType": 'text/plain',
        "responseText": 'awesome response'
      });
      expect(doneFn).toHaveBeenCalledWith('awesome response');
    });
    it("allows responses to be setup ahead of time", function () {
      var doneFn = jasmine.createSpy("success");
      jasmine.Ajax.stubRequest('/another/url').andReturn({
        "responseText": 'immediate response'
      });
      var xhr = new XMLHttpRequest();
      xhr.onreadystatechange = function(args) {
        if (this.readyState == this.DONE) {
          doneFn(this.responseText);
        }
      };

      xhr.open("GET", "/another/url");
      xhr.send();

      expect(doneFn).toHaveBeenCalledWith('immediate response');
    });
  });
  it("allows use in a single spec", function() {
    var doneFn = jasmine.createSpy('success');
    jasmine.Ajax.withMock(function() {
      var xhr = new XMLHttpRequest();
      xhr.onreadystatechange = function(args) {
        if (this.readyState == this.DONE) {
          doneFn(this.responseText);
        }
      };

      xhr.open("GET", "/some/cool/url");
      xhr.send();

      expect(doneFn).not.toHaveBeenCalled();

      jasmine.Ajax.requests.mostRecent().response({
        "status": 200,
        "responseText": 'in spec response'
      });

      expect(doneFn).toHaveBeenCalledWith('in spec response');
    });
  });
});
```
## KnockoutJS
#### What should and shouldn’t be tested?
用人不疑，疑人不用。所以你不需要测试Knockout正确的将一个`ko.observable()`对象显示在一个前台`data-bind="text: ..."`的span对象上，同样我们也不需要测试`ko.computed`会在任何依赖发生变化时运行。那么我们测试什么呢？答案就是逻辑，我们应该更多的关注自己所写的逻辑代码，这些才是测试的重点。
#### 一个简单栗子
```javascript
var addressBookViewModel = {
  entries : ko.observableArray([]),
  newEntryFirstName : ko.observable(),
  newEntrySurname : ko.observable()
    addNewEntry : function() {
        var newEntry = {
          firstName : this.newEntryFirstName(),
          surname : this.newEntrySurname()
      };
      this.entries.push(newEntry);
      // clear form
      this.newEntryFirstName('');
      this.newEntrySurname('');
  }
};

ko.applyBindings(addressBookViewModel);
```
##### 分析
可以想象一下这个`ViewModel`的应用场景，用户输入`firstname`,一个`surname`并且点击绑定了'addNewEntry'的按钮。分析逻辑我们发现，此时一个`entry`被加入了`entries`（可能会用在一个`table`里通过`foreach`显示），然后清空。
在开始测试之前我们首先要对代码进行一些重构，因为首先这个`ViewModel`是单例的，并且当前方法验证点太多我们需要拆分一下。重构后的代码如下：
```javascript
function AddressBookViewModel() {
  this.entries = ko.observableArray([]);
  this.newEntryFirstName = ko.observable();
  this.newEntrySurname = ko.observable();
  this.addNewEntry = function() {
      addAddressBookEntry(this.newEntryFirstName, this.newEntrySurname, this.entries);
      clearObservables([this.newEntryFirstName, this.newEntrySurname]);
  };
}

function addAddressBookEntry(firstName, surname, list) {
  var newEntry = {
      firstName : ko.toJS(firstName),
      surname : ko.toJS(surname)
  };
  list.push(newEntry);
}

function clearObservables(observables) {
  observables.forEach(function(observable){
      observable(null);
  });
}

var addressBookViewModel = new AddressBookViewModel();
ko.applyBindings(addressBookViewModel);
```
##### 开始测试
- addAddressBookEntry
```javascript
// 'Describe' creates a Jasmine test. A describe block contains assertions, using the 'it' function.
describe('addAddressBookEntry', function(){

  var newEntryFirstName, newEntryLastName, list;

  // 'beforeEach' performs setup before each 'it' test
  beforeEach(function(){
      newEntryFirstName = ko.observable('Peggy');
      newEntryLastName = ko.observable('Hill');
      list = ko.observableArray([]);
  });
  
  it('Adds an entry to the provided list', function(){       
      var initialListLength = list().length;
      addAddressBookEntry(newEntryFirstName, newEntryLastName, list);     
      var newListLength = list().length;

      // Jasmine uses the 'expect' function for assertions. Its format is very human-readable.
      // If an expection proves false, it will throw an exception and the assertion will be reported as failed.
      expect(newListLength).toBe(initialListLength + 1);
  });

  it('Adds an entry containing the supplied firstname and surname', function(){
      addAddressBookEntry(newEntryFirstName, newEntryLastName, list);
      var unwrappedList = list();
      var expectedNewEntry = {firstName: 'Peggy', surname: 'Hill'};
      // Jasmine's toContain will, amongst other things, test whether an array contains an object with fields matching a supplied object
      expect(unwrappedList).toContain(expectedNewEntry);
  });

  it('Adds the entry to the end of the list', function(){
      addAddressBookEntry(newEntryFirstName, newEntryLastName, list);
      var unwrappedList = list();
      var lastEntry = unwrappedList[unwrappedList.length - 1];

      // You can have multiple expectations in a Jasmine test
      expect(lastEntry.firstName).toBe('Peggy');
      expect(lastEntry.surname).toBe('Hill');
  });

});
```

#### 一个普通栗子
```javascript
function FormatterBinding(formatter) {
  this.update = function update(element, valueAccessor) {
      var newModelValue = ko.unwrap(valueAccessor());
      var formattedText = formatter(newModelValue);
      // let's assume we don't need to support IE8
      element.textContent = formattedText;
  };
}

function formatNumberAsDollars(number) {
  return number.toLocaleString('en-US',{style: 'currency', currency: 'USD', maximumFractionDigits: 2});
}
```
##### 分析
这个`FormatterBinding`是用来将数字格式化为货币。对于下面的`formatNumberAsDollars`我们不需要测试，我们只需关注`FormatterBinding`。
```javascript
var mockFormatter, customBinding, mockValueAccessor;
// 'beforeEach' performs setup before each 'it' test
beforeEach(function(){
  mockFormatter = jasmine.createSpy('mockFormatter');
  customBinding = new FormatterBinding(mockFormatter);
  mockElement = document.createElement('p');
  mockValueAccessor = function() {
      return ko.observable('someMockValue');
  }
});

it('Calls the formatter with the value in the valueAccessor', function(){
  // let's assume we've created all our mocks as part of a Jasmine 
  // beforeEach block that runs before each set of assertions

  customBinding.update(mockElement, mockValueAccessor);
  expect(mockFormatter).toHaveBeenCalledWith('someMockValue');
});

it('Prints the output of the formatter to the element', function(){
  // Setting up the spy is a little bit awkward
  var mockFunctions = {
      mockFormatter : function() { return 'I am the mockFormatter return value';}
  };
  spyOn(mockFunctons, 'mockFormatter');

  // But everything else is dead easy
  var customBinding = new FormatterBinding(mockFunctions.mockFormatter);
  customBinding.update(mockElement, mockValueAccessor);
  var mockElementContent = mockElement.textContent;
  expect(mockElementContent).toBe('I am the mockFormatter return value');
});
```

*然而并没有时间再去准备高级栗子了...*
