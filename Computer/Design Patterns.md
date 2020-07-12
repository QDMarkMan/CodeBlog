# Design Patterns

[toc]

> 设计模式是可复用面向对象软件的基础。设计模式的精髓在于"找到变化并封装之"。

## 六大原则

我们所有的设计原则都是为了实现**高内聚，低耦合**的终极目标。

- **开闭原则**：一个软件实体如类，函数或模块应该对扩展开放，对修改关闭。这是一个比较宽泛模糊的概念，这个是最基础的设计原则。

- **单一职责原则**： 不要存在多于一个导致类变更的原因。

- **里式替换原则**：所有引用基类的地方必须能够透明的使用其子类的对象。通俗的来讲：子类可以扩展父类的方法，但是不能改变父类原有的功能。

- **依赖倒置原则**：高层模块不应该依赖低层模块，二者都应该依赖其抽象；抽象不应该依赖细节，细节应该依赖抽象。这个原则的一个基准是： 相对于细节的多边性，抽象的东西要稳定的多。大部分情况下抽象指的就是**接口或者抽象类**，而细节指的就是**具体的实现类**。

- **接口隔离原则**：客户端不应该依赖他不需要的接口；一个类对另一个类的依赖应该建立在最小的接口上。避免写出臃肿的接口，导致一些类实现不需要的方法。可以把一个臃肿的接口，拆分成多个独立的接口，然后让使用类自由组合。 他和单一职责的区别是： 单一职责主要约束类，接口隔离主要约束接口。

- **迪米特法则**：一个对象应该保持对其他对象最小的了解。主要目的是为了降低类和类之前的**耦合**。

  ****

## 创建型模式

> 这类模式提供创建对象的机制， 能够提升已有代码的灵活性和可复用性。

## 结构型模式

> 这类模式介绍如何将对象和类组装成较大的结构， 并同时保持结构的灵活和高效。

###  代理模式

**代理模式**是一种结构型设计模式，让你能够提供对象的替代品或其占位符。代理控制着对于原对象的访问，并允许在将请求提交给对象前后进行一些处理。

一般场景：

如果我们有一个消耗大型资源的巨型对象，但是我们只是偶尔的需要使用它，你可以延迟初始化，在用的时候再去创建它，但是这样的话对象的所有客户端都要延迟执行该代码，这样就会带来很多的重复代码。

通过使用代理模式，我们可以新建一个与原服务对象接口相同的代理类，然后更新应用以将代理对象传递给所有原始对象客户端。那么代理对象在接收到客户端请求后会创建实际的服务对象，并将工作都委派给它。

结构：

1. **服务接口(Service Interface)**: 声明了服务接口，代理必须遵循该接口才能伪装成服务对象。
2. **服务(Service)**：提供具体业务逻辑。
3. **代理(Proxy)**：包含一个指向服务对象的的引用成员变量，代理完成它的任务后会将请求传递给服务对象。
4. **客户端(Client)**： 能够通过同一接口与服务或者代理进行交互，所以你可以在一切需要服务对象的接口中使用。

简易代码：

```ts
/**
 * The Subject interface declares common operations for both RealSubject and the
 * Proxy. As long as the client works with RealSubject using this interface,
 * you'll be able to pass it a proxy instead of a real subject.
 */
interface Subject {
  request(str?: string) :void
}

/**
 * The RealSubject contains some core business logic. Usually, RealSubjects are
 * capable of doing some useful work which may also be very slow or sensitive -
 * e.g. correcting input data. A Proxy can solve these issues without any
 * changes to the RealSubject's code.
 */
class RealSubject implements Subject {
  /**
   * doSomeLogic
   */
  public request(): void {
    console.log(`%c===> do some logic`, 'color: #409EFF;')
  }
}

class ProxyCore implements Subject {
  private realSubject: RealSubject
  constructor (realSubject: RealSubject) {
    this.realSubject = realSubject
  }

  private checkAccess(): boolean {
    console.log('you can choose weather call request')
    return true
  }

  private logging():void {
    console.log('some service called request')
  }

  /**
   * request
   */
  public request() {
    // call service
    if (this.checkAccess()) {
      this.realSubject.request()
      this.logging()
    }
  }
}

function client(realSubject: RealSubject): void {
  // ... do something
  realSubject.request()
}
console.log('Client: Executing the client code with a real subject:');
const realSubject = new RealSubject()
client(realSubject)

console.log('\n')

console.log('Client: Executing the same client code with a proxy:');
const proxySubject = new ProxyCore(realSubject)
client(proxySubject)
```

适用场景：

1. 延迟初始化（虚拟代理）。 当有一个偶尔使用的重量级服务对象。一直保持对该对象的运行会消耗资源时。
2. 访问控制（保护代理）。如果你只希望特定的客户端使用服务对象，这里的对象可以是操作系统中的非常重要的部分，而客户端则是各种已经启动的程序。**代理可以在客户端满足某些凭证的时候将请求传递给服务对象**。
3. 本地执行远程服务（远程代理）。适用于服务对象位于远程服务器上的情形。
4. 记录日志请求（日志记录代理）。适用于当你需要保存对于服务对象的请求历史记录时，代理可以在向服务传递请求前进行记录。

## 行为模式

> 这类模式负责对象间的高效沟通和职责委派。

### 迭代器模式

重要程度：⭐️

**迭代器模式**是一种行为设计模式， 让你能够在不暴露集合底层表现形式的情况下遍历集合中的所有元素。

一般场景：

### 策略模式

重要程度：⭐️⭐️⭐️

 **策略模式**是一种行为设计模式，它能让你定义一系列的算法，并将每种算法分别放入独立的类中，以使得算法的对象可以相互替换。 

一般场景：当程序需要不同的算法去完成的时候，我们可以把其中的算法抽离到一组被称为策略的类中，通过暴露给客户端的**上下文**去调用不同的算法。

结构：

1. **上下文(Context)**：维护着指向具体策略的引用，而且仅仅通过策略接口与该对象进行交流。
2. **策略(Strategy)**: 是所有具体策略的通用接口，它声明了一个上下文用于执行策略的方法。
3. **具体策略(Concrete Stategy)**: 实现了上下文所用算法的不同变体。当上下文需要运行算法时， 它会在其已连接的策略对象上调用执行方法。 上下文不清楚其所涉及的策略类型与算法的执行方式。
4. **客户端(Client)**: 创建一个特定的策略对象并传递给上下文，上下文则会提供一个设置器以便客户端在运行时替换相关联的策略。

简易代码：

```ts
// 策略接口
interface Strategy {
  doAlgorithm (data: number[]): number
}
/**
 * 策略接口
 * @class Context
 */
class Context {
  private strategy: Strategy // 上下文中的策略类
  constructor (strategy: Strategy) {
    this.strategy = strategy;
  }
  
  public setStrategy (strategy: Strategy) :void {
    this.strategy = strategy
  }
  
  public doSomeLogic (): void {
    console.log('Context: Counting data using the strategy (not sure how it\'ll do it)');
    const result = this.strategy.doAlgorithm([1,2,3,4])
    console.log(result)
  }
}
/**
 * 具体策略
 * @class MultiStrategy
 * @implements {Strategy}
 */
class MultiStrategy implements Strategy {
  /**
   * doAlgorithm
   */
  public doAlgorithm(data: number[]): number {
    if (data.length === 0) return 0
    let result = data.reduce((pre: number, next:number) => {
      pre = pre * next
      return pre
    }, 1)
    return result
  }
}
class AddStrategy implements Strategy {
  /**
   * doAlgorithm
   */
  public doAlgorithm(data: number[]): number {
    if (data.length === 0) return 0
    let result = data.reduce((pre: number, next:number) => {
      pre = pre + next
      return pre
    }, 0)
    return result
  }
}

// ------------------------------ usage ------------------------------ //
const context = new Context(new MultiStrategy())
context.doSomeLogic()
// update strategy
context.setStrategy(new AddStrategy())
context.doSomeLogic()
```

适用场景：

1. 当你想使用不同的算法，并能够在运行时切换不同的算法时。
2. 当你有许多只有在执行某些行为时略有不同的相似类的时候。
3. 算法在上下文的逻辑中不是特别重要，使用策略模式可以把算法细节和业务逻辑隔离开。
4. 当类中使用了复杂条件运算符以在同一算法的不同变体中切换时。

###  观察者模式

重要程度：⭐️⭐️⭐️⭐️

**观察者模式**是一种行为设计模式， 允许你定义一种订阅机制，可在对象事件发生时通知多个“观察”该对象的其他对象。

一般场景：假如你有两种类型的对象，`商店`和`顾客`。顾客对某个特定品牌的手机很感兴趣，而且手机**很快就会发售**，顾客每天打电话或者去店里询问是否开售。这显然是很愚蠢的。那么这个时候商店就要发布信息给客户来解决这个问题更合适。

结构：

1. **发布者(Publisher)**: 会向其他对象发布值得关注的事件。发布者会在自身状态发生改变和执行特定行为的时候发布事件，发布者维护着新订阅者加入和当前存在订阅者离开的订阅构架。当事件发布时，发布者会遍历订阅者列表并且调用每个订阅者对象的通知方法。
2. **订阅者接口(Subscriber)**:  声明了通知接口，在大多数情况下都包含了一个`update`方法，该方法可以拥有多个参数，使发布者可以在更新时传递事件的详细信息。
3. **具体订阅者(Concrete Subscriber)**：执行一些操作来回应发布者的通知，所有的具体订阅者类都实现了同样的接口，因此发布者不需要与具体类相耦合。
4. **客户端(Client)**: 分别创建发布者和订阅者，然后为订阅者注册发布者更新。

简易代码：

```ts
interface Observer {
  update (ObserverSubject: ObserverSubject): void; 
}
// The ObserverSubject interface declares a set of methods for managing subscribers.ß
interface ObserverSubject {
  attach (observer: Observer): void;
  detach (observer: Observer): void;
  notify ():void;
}
class ConcreteSubject implements ObserverSubject {
  public state:number;
  private observers: Observer[] = [];

  /**
   * attach
   */
  public attach(observer: Observer): void {
    const isExit = this.observers.includes(observer)
    if (isExit) {
      return console.log('ObserverSubject: Observer has been attached already.');
    }
    this.observers.push(observer)
  }

  /**
   * detach
   */
  public detach(observer: Observer): void {
    const index = this.observers.indexOf(observer)
    if (index === -1) {
      return console.log('ObserverSubject: Nonexistent observer.');
    }
    this.observers.splice(index, 1)
    console.log('ObserverSubject: Detached an observer.');
  }

  /**
   * notify
   */
  public notify(): void {
    for (const observer of this.observers) {
      observer.update(this)
    }
  }

  /**
   * logic
   */
  public logic() {
    this.state = Math.floor(Math.random() * (100  + 1))
    console.log(`%c===> Put some logic here`, 'color: #409EFF;')
    this.notify()
  }
}

class Observer1 implements Observer {
  update (subject: ObserverSubject): void {
    if (subject instanceof ConcreteSubject) {
      console.log(`%c===> Observer1 state ${subject.state}`, 'color: #409EFF;')
    }
  }
}

const subject = new ConcreteSubject()
const observer1 = new Observer1()
subject.attach(observer1) // 监听事件
subject.logic() // notify
subject.detach(observer1) // 卸载事件
subject.logic() // notify
```

适用场景：

1. 当一个对象的改变需要改变其他对象，或者对象是事先未知或动态变化时，可使用观察者模式。
2. 当应用中一些对象必须观察其他对象时，但仅能在有限的时间内或者特定情况下使用。

### 命令模式

重要程度：