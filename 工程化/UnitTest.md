# UnitTest

单元测试实战总结。

## Sinon

>  Standalone test spies, stubs and mocks for JavaScript. Works with any unit testing framework.

Sion是具有独立spies, stubs and mocks功能的测试框架，可以集成到任意的单元测试架构中， 例如`Mocha`等。Sion的主要作用是通过**测试替代**轻松消除测试的复杂度。测试替代是在测试中使用真是代码逻辑的替代品。比如Ajax等。

### spies

俗称间谍函数，主要作用是收集有关函数调用的信息。可以使用它来帮助我们验证事物，例如是否调用了函数。同样spies实现的基础上是不会影响函数本身的正常调用。

使用场景。

- 验证函数是否被调用。

  ```js
  it('should call save once', function() {
    var save = sinon.spy(Database, 'save');
  
    setupNewUser({ name: 'test' }, function() { });
  
    save.restore();
    sinon.assert.calledOnce(save);
  });
  ```

- 检查传递给函数的参数。

  ```js
  it('should pass object with correct values to save', function() {
    var save = sinon.spy(Database, 'save');
    var info = { name: 'test' };
    var expectedUser = {
      name: info.name,
      nameLowercase: info.name.toLowerCase()
    };
  
    setupNewUser(info, function() { });
  
    save.restore();
    sinon.assert.calledWith(save, expectedUser);
  });
  ```

  

### stub

### mock



