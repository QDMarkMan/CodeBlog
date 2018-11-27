# 构建一个Ionic3.X Hybrid App(未完成。。。)
通过`Ionic3 + Angular5`构建一个`Hybird App`需要注意的地方。

## 模块解析
首先我们来看看`NgModule`
```ts
interface NgModule {
    // providers: 这个选项是一个数组,需要我们列出我们这个模块的一些需要共用的服务
    //            然后我们就可以在这个模块的各个组件中通过依赖注入使用了.
  providers : Provider[]
    // declarations: 数组类型的选项, 用来声明属于这个模块的指令,管道等等.
    //               然后我们就可以在这个模块中使用它们了.
  declarations : Array<Type<any>|any[]>
    // imports: 数组类型的选项,我们的模块需要依赖的一些其他的模块,这样做的目的使我们这个模块
    //          可以直接使用别的模块提供的一些指令,组件等等.
  imports : Array<Type<any>|ModuleWithProviders|any[]>
    // exports: 数组类型的选项,我们这个模块需要导出的一些组件,指令,模块等;
    //          如果别的模块导入了我们这个模块,
    //          那么别的模块就可以直接使用我们在这里导出的组件,指令模块等.
  exports : Array<Type<any>|any[]>
  // entryComponents: 数组类型的选项,指定一系列的组件,这些组件将会在这个模块定义的时候进行编译
  //                  Angular会为每一个组件创建一个ComponentFactory然后把它存储在ComponentFactoryResolver
  entryComponents : Array<Type<any>|any[]>
  // bootstrap: 数组类型选项, 指定了这个模块启动的时候应该启动的组件.当然这些组件会被自动的加入到entryComponents中去
  bootstrap : Array<Type<any>|any[]>
  // schemas: 不属于Angular的组件或者指令的元素或者属性都需要在这里进行声明.
  schemas : Array<SchemaMetadata|any[]>
  // id: 字符串类型的选项,模块的隐藏ID,它可以是一个名字或者一个路径;用来在getModuleFactory区别模块,如果这个属性是undefined
  //     那么这个模块将不会被注册.
  id : string
}
```
- `app.module.ts`
```
app.module.ts
└───@NgModule
    └───declarations                // 告诉Angular哪些模块属于NgModule
    │───imports                     // 导入的模块
    │───bootstrap                   // 启动模块
    │───entryComponents             // 定义组建时应该被编译的组件
    └───providers                   // 服务配置
```

`entryComponents`:`Angular`使用`entryComponents`来启用“树震动”，即只编译项目中实际使用的组件，而不是编译所有在`ngModule`中声明但从未使用的组件。离线模板编译器`（OTC）`只生成实际使用的组件。如果组件不直接用于模板，`OTC`不知道是否需要编译。有了`entryComponents`，你可以告诉`OTC`也编译这些组件，以便在运行时可用。

## 生命周期

生命周期的重要性不用多说，这是`Ionic`官网的[介绍](https://ionicframework.com/docs/api/navigation/NavController/)
- `constrctor => void`: 构造函数启动，构造函数在ionViewDidLoad之前被触发
- `ionViewDidLoad => void`：资源加载完毕时触发。ionViewDidLoad只在第一次进入页面时触发**只触发一次**
- `ionViewWillEnter => void`：页面即将给进入时触发**每次都会触发**
- `ionViewDidEnter => void`：进入页面之后出发**每次都会触发**
- `ionViewWillLeave => void`：即将离开（仅仅是触发要离开的动作）时触发**每次都会触发**
- `ionViewDidLeave => void`：已经离开页面时触发**每次都会触发**
- `ionViewWillUnload => void`： 在页面即将被销毁并删除其元素时触发
- `ionViewCanEnter => boolean`：在视图可以进入之前运行。 这可以在经过身份验证的视图中用作一种“保护”，您需要在视图可以进入之前检查权限
- `ionViewCanLeave => boolean`：在视图可以离开之前运行。 这可以在经过身份验证的视图中用作一种“防护”，您需要在视图离开之前检查权限
```js
constructor(public navCtrl: NavController) {
  console.log('触发构造函数')
}
/**
 * 关于Ionic3.X的声明
 */

/**
 * 页面加载完成触发，这里的“加载完成”指的是页面所需的资源已经加载完成，但还没进入这个页面的状态（用户看到的还是上一个页面）。全程只会调用一次
 */
ionViewDidLoad () {
  console.log(`Ionic触发ionViewDidLoad`);
  // Step 1: 创建 Chart 对象
  const chart = new F2.Chart({
    id: 'myChart',
    pixelRatio: window.devicePixelRatio // 指定分辨率
  })
  // Step 2: 载入数据源
  chart.source(data)
  chart.interval().position('genre*sold').color('genre')
  chart.render()
}
/**
 * 即将进入Ionic视图  这时对页面的数据进行预处理 每次都会触发
 */
ionViewWillEnter(){
  console.log(`Ionic触发ionViewWillEnter`)
}
/**
 * 已经进入Ionic视图 每次都会触发
 */
ionViewDidEnter(){
  console.log(`Ionic触发ionViewDidEnter`)
}
/**
 * 页面即将 (has finished) 离开时触发 每次都会触发
 */
ionViewWillLeave(){
  console.log(`Ionic触发ionViewWillLeave`)
}
/**
 * 页面已经 (has finished) 离开时触发，页面处于非激活状态了。 每次都会触发
 */
ionViewDidLeave(){
  console.log(`Ionic触发ionViewDidLeave`)
}
/**
 * 页面中的资源即将被销毁 一般用处不大
 */
ionViewWillUnload(){
  console.log(`Ionic触发ionViewWillUnload`)
}
//守卫导航钩子： 返回true或者false
/**
 * 在视图可以进入之前运行。 这可以在经过身份验证的视图中用作一种“保护”，您需要在视图可以进入之前检查权限
 */
ionViewCanEnter(){
  console.log(`Ionic触发ionViewCanEnter`)
  const date = new Date().getHours()
  console.log(date)
  if (date > 22) {
    return false
  }
  return true
}
/**
 * 在视图可以离开之前运行。 这可以在经过身份验证的视图中用作一种“防护”，您需要在视图离开之前检查权限
 */
ionViewCanLeave(){
  console.log(`Ionic触发ionViewCanLeave`)
  const date = new Date().getHours()
  console.log(date)
  if (date > 10) {
    return false
  }
  return true
}
```
## 配置文件
Ionic3.X中并没有提供相应的的配置文件，所以我们需要自己按照下面步骤手动去添加配置文件来对项目进行配置。
1. 新增`config`目录
```
src
  |__config
      |__config.dev.ts
      |__config.prod.ts
```
`config.dev.ts` / `config.prod.ts`
```js
export const CONFIG = {
  BASE_URL            : 'http://172.16.255.254:3000/api', // API地址
  VERSION             : '1.0.0'
}
```
2. 在根目录下新增`build`文件夹，在文件夹中新增`webpack.config.js` config文件
```js
const fs = require('fs')
const chalk =require('chalk')
const path = require('path')
// ionic default option
const defaultConfig = require('@ionic/app-scripts/config/webpack.config.js');
const env = process.env.IONIC_ENV
function environmentPath(env) {
  const filePath = `./src/config/config.${env}.ts`
  if (!fs.existsSync(filePath)) {
    console.log(chalk.red('\n' + filePath + ' does not exist!'));
  } else {
    return filePath;
  }
}
console.log("当前环境为："+env)
// 设置环境变量别名
defaultConfig.prod.resolve.alias = {
  "@app/env": path.resolve(environmentPath('prod'))
}
defaultConfig.dev.resolve.alias = {
  "@app/env": path.resolve(environmentPath('dev'))
}
if (env !== 'prod' && env !== 'dev') {
  defaultConfig[env] = defaultConfig.dev
  defaultConfig[env].resolve.alias = {
    "@app/env": path.resolve(environmentPath(env))
  }
}
module.exports = function () {
  return defaultConfig
}
```
3. `tsconfig.json`配合,配置中新增如下内容 这个地方很扯  **这个`path`相关的需要放在`tsconfig.json`的最上面**
```json
"baseUrl": "./src",
  "paths": {
    "@app/env": [
      "environments/environment"
    ]
  }
```
4. 修改`package.json`。配置末尾新增如下内容
```bash
"config": {
  "ionic_webpack": "./config/webpack.config.js"
}
```
5. 使用配置变量
```js
import {CONFIG} from "@app/env"
```

## 路由部分
- 首页设置
  有时候我们需要设置我们第一次显示得页面。那这样我们就需要使用`NavController`来设置
  ```ts
  public rootPage: any = StartPage; // 
  ```
- 路由跳转
  1. `href方式跳转`：直接在dom中指定要跳转的页面，以`tabs`中的代码为例
  ```html
  <!-- 单个跳转按钮  [root]="HomeRoot" 是最重要的 -->
  <ion-tab [root]="HomeRoot" tabTitle="Home" tabIcon="home"></ion-tab>
  ```
  ```ts
  import { HomePage } from '../home/home'
  export class TabsPage {
    // 声明变量地址
    HomeRoot = HomePage
    constructor() {
      
    }
  }
  ```
  2. 编程式导航：编程式导航我们可能会用的更多，下面是一个基础的例子
  
  编程式导航是由`NavController`控制
  > NavController是Nav和Tab等导航控制器组件的基类。 您可以使用导航控制器导航到应用中的页面。 在基本级别，导航控制器是表示特定历史（例如Tab）的页面数组。 通过推送和弹出页面或在历史记录中的任意位置插入和删除它们，可以操纵此数组以在整个应用程序中导航。当前页面是数组中的最后一页，如果我们这样想的话，它是堆栈的顶部。 将新页面推送到导航堆栈的顶部会导致新页面被动画化，而弹出当前页面将导航到堆栈中的上一页面。

  除非您使用`NavPush`之类的指令，或者需要特定的`NavController`，否则大多数时候您将注入并使用对最近的`NavController`的引用来操纵导航堆栈。

  ```ts
  // 引入NavController
  import { NavController } from 'ionic-angular';
  import { NewsPage } from '../news/news'
  export class HomePage {
    // 注入NavController
  constructor(public navCtrl: NavController) {
    // this.navCtrl.push(LoginPage)
  }
  goNews () {
      this.navCtrl.push(NewsPage, {
        title : '测试传参'
      })
    }
  }
  ```
  相关常用API
    1. `navCtrl.push(OtherPage, param)`： 跳转页面
    2. `navCtrl.pop()`: `Removing a view` 移除当前View，相当于返回上一个页面
  
  3. 路由中参参数相关
    - `push(Page, param)`传参： 这个很简单也很明白
    ```ts
    this.navCtrl.push(NewsPage, {
      title : '测试传参'
    })
    ```
    - `[navParams]`属性：和`HTML`配合进行传参
    ```ts
    import {LoginPage } from'./login';
    @Component()
    class MyPage {
      params;
      pushPage: any;
      constructor(){
        this.pushPage= LoginPage;
        this.params ={ 
          id:123,
          name: "Carl"
        }
      }
    }
    ```
    ```html
    <button ion-button [navPush]="pushPage" [navParams]="params">
      Go
    </button>
    <!-- 同理在root page上传递参数就是下面这种方式 -->
    <ion-tab [root]="tab1Root"  tabTitle="home" tabIcon="home"  [rootParams]="userInfo">
    </ion-tab
    ```
    - 获取参数
    ```ts
    //NavController就是用来管理和导航页面的一个controller
    constructor(public navCtrl: NavController, public navParams: NavParams) {
      //1： 通过NavParams get方法获取到单个对象
      this.titleName = navParams.get('name')
      //2: 直接获取所有的参数
      this.para = navParams.data
    }
    ```
- 路由拦截

## provider(service)使用
> 当重复的需要一个类中的方法时，可封装它为服务类，以便重复使用，如http。

`provider`，也叫`service`。前者是`ionic`的叫法，后者是`ng`的叫法。建议仔细得学一下`Angular`
- 创建Provider
Ionic提供了创建指令
```bash
ionic g provider http 
```
自动创建的`Provider`会自主动在`app.module中导入`注意**这个需要在app.module中注入**
首先导入`装饰器`，再用装饰器装饰，这样，该类就可以作为提供者注入到其他类中以使用：
```js
import { Injectable } from '@angular/core';
@Injectable()

export class StorageService {
  constructor() {
    console.log('Hello StorageService');
  }
  myAlert(){
    alert(“服务类的方法”)
  }
}
```
- 使用provider 

正常的应用，注入，然后使用
```js
import { StorageService } from './../../service/storage.service';
export class LoginPage {

  userName: string = 'demo'
  password: string = '123456'

  constructor(
    public storageService: StorageService
    ) {
    
  }
  doLogin () {
    const para = {
      userName: this.userName,
      password:  this.password
    }
    console.log(para)
    if (para.userName === 'demo' && para.password === '123456') {
      this.storageService.setStorage('user', para)
    }
    setTimeout(() => {
      console.log(this.storageService.getStorage('user'))
    }, 3000)
  }
}
```
## 事件系统
> Events是一个`发布-订阅`样式事件系统，用于在您的应用程序中发送和响应应用程序级事件。

这个是不同页面之间交流的核心。主要用于组件的通信。你也可以用`events`传递数据到任何一个页面。

`Events`实例方法
- `publish(topic, eventData)`: 发布一个`event`
- `subscribe(topic, handler)`: 订阅一个`event`
- `unsubscribe(topic, handler)` 取消订阅一个`event`
```ts
// 发布event login.ts
// 发布event事件
submitEvent (data) {
  console.log(1)
  this.event.publish('user:login', data)
}
// 订阅页面  message.ts
constructor(public event: Events ) {
  // 订阅event事件
  event.subscribe('user:login', (data) => {
    console.log(data)
    let obj = {
      url: 'assets/imgs/logo.png',
      name: data.username
    }
    this.messages.push(obj)
  })
}
```
**注意点**: <font color="red">1： 订阅必须再发布之前，不然接收不到。打个比喻：比如微信公众号，你要先关注才能接收到它的推文，不然它再怎么发推文，你也收不到。2： `subscribe`中得`this`指向是有点问题的。</font>


## 用户操作事件
> Basic gestures can be accessed from HTML by binding to `tap`, `press`, `pan`, `swipe`, `rotate`, and `pinch` events.

Ionic对手势事件的解释基本是一笔带过。

## **组件间通信**
组件之间的通信：要把一个组件化的框架给玩6了。组件之前的通信搞明白了是个前提。在`Ionic`中，我们使用`Angular`中的方式来实现。
- `父 => 子`： `@Input()`
  - **通过`输入型绑定`把数据从父组件传到子组件**：这个用途最广泛和常见，和`recat`中的props非常相似
  ```ts
  // 父组件定义值（用来传递）
  export class NewsPage {
    father: number = 1 // 父组件数据
    /**
     * Ionic生命周期函数
    */
    ionViewDidLoad() {
      // 父组件数据更改
      setTimeout(() => {
        this.father ++ 
      }, 2000)
    }
  }
  // 子组件定义属性（用来接收）
  @Input() child: number // @Input装饰器标识child是一个输入性属性
  ```
  ```html
  <!-- 父组件使用 -->
  <backtop [child]="father"></backtop>
  <!-- 子组件定义 -->
  <div class="backtop">
    <p (click)="click()">back</p>
    father数据: {{child}}
  </div>
  ```
  - 通过`get, set`在子组件中对父组件得数据进行拦截来达到我们想要得结果
  ```ts
  // 拦截父组件得值
  private _showContent: string 
  @Input()
  // set value
  set showContent(name: string) {
    if (name !== '4') {
      this._showContent = 'no'
    } else {
      this._showContent = name
    }
  }
  // get value
  get showContent () :string {
    return this._showContent
  }
  ```
  - 通过`ngOnChanges`监听值得变化
  ```ts
  // 监听所有属性值得变化
  ngOnChanges(changes: SimpleChange): void {
    /**
     * 从旧值到新值得一次变更
     * class SimpleChange {
        constructor(previousValue: any, currentValue: any, firstChange: boolean)
        previousValue: any // 变化前得值
        currentValue: any // 当前值
        firstChange: boolean
        isFirstChange(): boolean // 检查该新值是否从首次赋值得来的。
      }
     */
    // changes props集合对象
    console.log(changes['child'].currentValue) // 
  }
  ```
  - 父组件与子组件通过`本地变量`互动
  > 父组件不能使用数据绑定来读取子组件的属性或调用子组件的方法。但可以在父组件模板里，`新建一个本地变量来代表子组件`，然后利用这个变量来读取子组件的属性和调用子组件的方法.

  通过`#childComponent`定义这个组件。然后直接使用`childComponent.XXX`去调用。这个的话就有点强大了，但是这个交流时页面级别的。仅限于在`html`定义本地变量然后在`html`中进行操作和通信。也就是`父组件-子组件的连接必须全部在父组件的模板中进行。父组件本身的代码对子组件没有访问权。`
  ```html
  <!-- 父组件 -->
  <button ion-button color="secondary" full  (click)="childComponent.fromFather()">测试本地变量</button>
  <backtop #childComponent [child]="father" [showContent] = "father" (changeChild)="childCome($event)"></backtop>
  ```
  ```ts
  // 子组件
  // 父子组件通过本地变量交互
  fromFather () {
    console.log(`I am from father`)
    this.show  = !this.show
  }
  ```
  - 父组件调用`@ViewChild()`互动
  > 如果父组件的类需要读取子组件的属性值或调用子组件的方法，可以把子组件作为 ViewChild，注入到父组件里面。

  也就是说`@ViewChild()`是为了解决上面的短板而出现的。
  ```ts
  // 父组件
  import { Component, ViewChild } from '@angular/core';
  export class NewsPage {
    //定义子组件数据
    @ViewChild(BacktopComponent)
    private childComponent: BacktopComponent
    ionViewDidLoad() {
      setTimeout(() => {
        // 通过child调用子组件方法
        this.childComponent.formChildView()
      }, 2000)
    }
  }
  ```
- `子 => 父`: `@Output()`： 最常用的方法
> 子组件暴露一个 `EventEmitter` 属性，当事件发生时，子组件利用该属性 `emits`(向上弹射)事件。父组件绑定到这个事件属性，并在事件发生时作出回应。
> 
```ts
// 父组件
// 接收儿子组件得来得值 并把儿子得值赋给父亲
childCome (data: number) {
  this.father =  data
}
// 字组件
// 子向父传递得事件对象
@Output() changeChild: EventEmitter<number> = new EventEmitter() // 定义事件传播器对象
// 执行子组件向父组件通信
click () {
  this.changeChild.emit(666)
}
```
```html
<!-- 父组件 -->
<backtop [child]="father" [showContent] = "father" (changeChild)="childCome($event)"></backtop>
```
**获取父组件实例**

有的时候我们也可以暴力一点获取父组件的实例去使用它（**未验证**）。
```ts
constructor(
    // 注册父组件
    @Host() @Inject(forwardRef(() => NewsPage)) father: NewsPage
  ) {
    this.text = 'Hello World';
    setTimeout(() => {
      // 直接通过对象来修改父组件
      father.father++
    }, 3000)
  }
```
- `父 <=> 子`：**父子组件通过服务来通信**
  
  如果我们`把一个服务实例的作用域被限制在父组件和其子组件内，这个组件子树之外的组件将无法访问该服务或者与它们通讯`。父子共享一个服务，那么我们可以利用该服务在家庭内部实现双向通讯。
  ```ts
  // service
  import { Injectable } from '@angular/core'; // 标记元数据
  // 使用service进行父子组件的双向交流
  @Injectable()
  export class MissionService {
    familyData: string = 'I am family data'
  }
  ```
  ```ts
  // father component
  import { MissionService } from './../../service/mission.service';
  export class NewsPage {
    constructor( public missionService: MissionService) {
    }
    ionViewDidLoad() {
      // 父组件数据更改
      setTimeout(() => {
        // 调用修改service中的数据 这个时候父子组件中的service都会改变
        this.missionService.familyData = 'change familyData'
      }, 2000)
    }
  }
  // child component
  import { Component} from '@angular/core';
  import { MissionService } from './../../service/mission.service';
  @Component({
    selector: 'backtop',
    templateUrl: 'backtop.html'
  })
  export class BacktopComponent {
    constructor(
      public missionService:MissionService
    ) {
      console.log(missionService)
      this.text = 'Hello World';
    }
    // 执行子组件向父组件通信
    click () {
      // 修改共享信息
      this.missionService.familyData = 'change data by child'
    }
  }
  ```
  ```html
  <!-- 父组件直接使用 -->
  {{missionService.familyData}}
  <!-- 子组件 -->
  <div>
    servicedata： {{missionService.familyData}}
  </div>
  ```
  
  在`service`中使用订阅也可以同样的实现数据的通信
  ```ts
  // mission.service.ts
  import { Subject } from 'rxjs/Subject';
  import { Injectable } from '@angular/core'; // 标记元数据
  // 使用service进行父子组件的双向交流
  @Injectable()
  export class MissionService {
    familyData: string = 'I am family data'
    // 订阅式的共享数据
    private Source = new Subject()
    Status$=this.Source.asObservable()
    statusMission (msg: string) {
      this.Source.next(msg)
    }
  }

  // 父组件
  // 通过service的订阅提交信息
  emitByService () {
    this.missionService.statusMission('emitByService')
  }
  // 子组件
  // 返回一个订阅器
  this.subscription = missionService.Status$.subscribe((msg:string) => {
    this.text = msg
  })
  ionViewWillLeave(){
    // 取消订阅
    this.subscription.unsubscribe()
  }
  ```
- `高级通信`
  
  1. 我们可以使用`ionic-angular中的Events`模块来进行 `父 <=> 子 , 兄 <=> 弟`的高级通信。`Events`模块在通信方面具有得天独厚的优势。具体可以看上面的示例
  2. 使用`EventEmitter`模块
  ```ts
  // service
  import { EventEmitter } from '@angular/core'; // 标记元数据
  // 使用service进行父子组件的双向交流
  @Injectable()
  export class MissionService {
    // Event通信 来自angular
    serviceEvent = new EventEmitter()
  }

  // 父组件
  // 通过Events 模块高级通信 接收信息
  this.missionService.serviceEvent.subscribe((msg: string) => {
    this.messgeByEvent = msg
  })

  // 子组件
  // 通过emit 进行高级通信 发送新
  emitByEvent () {
    this.missionService.serviceEvent.emit('emit by event')
  }

  ```
## Shared组件
公共组件设置
这个地方可以根据`Angular`提供的`CommonModule`共享模块，我们要知道他干了什么事儿
1. 它导入了 `CommonModule`，因为该模块需要一些常用指令。
2. 它声明并导出了一些工具性的管道、指令和组件类。
3. 它重新导出了 `CommonModule` 和 `FormsModule`
4. `CommonModule` 和 `FormsModule`可以代替`BrowserModule`去使用

- 定义
在`shared`文件夹下新建`shared.module.ts`
```ts
import { NgModule } from '@angular/core';
import { CommonModule } from '@angular/common';
import { FormsModule } from '@angular/forms'; 
// 通过重新导出 CommonModule 和 FormsModule，任何导入了这个 SharedModule 的其它模块，就都可以访问来自 CommonModule 的 NgIf 和 NgFor 等指令了，也可以绑定到来自 FormsModule 中的 [(ngModel)] 的属性了。
// 自定义的模块和指令
import { ComponentsModule } from './../components/components.module';
import { DirectivesModule } from './../directives/directives.module';

@NgModule({
  declarations: [],
  imports: [
    CommonModule,
    FormsModule
  ],
  exports:[
    // 导出模块
    CommonModule,
    FormsModule,
    ComponentsModule,
    DirectivesModule
  ],
  entryComponents: [

  ]
})
export class SharedModule {}
```
注意： **服务要通过单独的依赖注入系统进行处理，而不是模块系统**
## http部分
`Ionic`中的http模块是直接采用的`HttpClient`这个模块。这个没什么可说的，我们只需要根据我们的需求对`service`进行修改即可，例如我就把`http`改成了更加灵活的`Promise模式`
```ts
import { TokenServie } from './token.service';
import { StorageService } from './storage.service';
import { HttpClient, HttpHeaders, HttpParams } from '@angular/common/http'
import { Injectable, Inject } from '@angular/core'
import {ReturnObject, Config} from '../interface/index'
/*
Generated class for the HttpServiceProvider provider.
*/
@Injectable()
export class HttpService{
  /**
   * @param CONFIG 
   * @param http 
   * @param navCtrl 
   */
  constructor(
    @Inject("CONFIG") public CONFIG:Config, 
    public storage: StorageService,
    public tokenService: TokenServie,
    public http: HttpClient
    ) {
      console.log(this.CONFIG)
  }
  /**
   * key to 'name='qweq''
   * @param key 
   * @param value 
   */
  private toPairString (key, value): string {
    if (typeof value === 'undefined') {
      return key
    }
    return `${key}=${encodeURIComponent(value === null ? '' : value.toString())}`
  }
  /**
   * objetc to url params
   * @param param 
   */
  private toQueryString (param, type: string = 'get') {
    let temp = []
    for (const key in param) {
      if (param.hasOwnProperty(key)) {
        let encodeKey = encodeURIComponent(key)
        temp.push(this.toPairString(encodeKey, param[key]))
      }
    }
    return `${type === 'get' ? '?' : ''}${temp.join('&')}`
  }
  /**
   * set http header
   */
  private getHeaders () {
    let token = this.tokenService.getToken()
    return new HttpHeaders({
      'Content-Type':  'application/x-www-form-urlencoded',
      'tokenheader': token ? token : ''
    })
  }
  /**
   * http post请求 for promise
   * @param url
   * @param body
   */
  public post (url: string, body ? : any): Promise<ReturnObject> {
    const fullUrl = this.CONFIG.BASE_URL + url
    console.log(this.toQueryString(body, 'post'))
    return new Promise<ReturnObject>((reslove, reject) =>{
      this.http.post(fullUrl, body, {
        // params,
        headers: this.getHeaders()
      }).subscribe((res: any) => {
        reslove(res)
      }, err => {
        // this.handleError(err)
        reject(err)
      })
    })
  }
  /**
   * get 请求 return promise
   * @param url 
   * @param param 
   */
  public get(url: string, params: any = null): Promise<ReturnObject> {
    const fullUrl = this.CONFIG.BASE_URL + url
    let realParams = new HttpParams()
    for (const key in params) {
      if (params.hasOwnProperty(key)) {
        realParams.set(`${key}`, params[key])
      }
    }
    // add time map
    realParams.set(
      'timestamp', (new Date().getTime()).toString()
    )
    return new Promise<ReturnObject>((reslove, reject) =>{
      this.http.get(fullUrl, {
        params,
        headers: this.getHeaders()
      }).subscribe((res: any) => {
        console.log(res)
        reslove(res)
      }, err => {
        // this.handleError(err)
        reject(err)
      })
    })
  }
  // user has token or not
  public load () {
    if (this.tokenService.getToken()) {
      console.log('存在token')
    } else {
      console.log('用户还没登录')
    }
  }
}
```

## `Cordova插件使用`
下载`Cordova`插件
```bash
cordova add plugin plugin-name -D
```
使用(`Ionic Native`中的插件)示例
```ts
import { StatusBar } from '@ionic-native/status-bar';
constructor(private statusBar: StatusBar) {
    //沉浸式并且悬浮透明
    this.statusBar.overlaysWebView(true);
}
```
## App打包部分


## 优化部分
项目写完了，不优化一下 心里怪难受的。
### 组件懒加载

### 打包优化
- **新增--prod参数**
    ```bash
    "build:android": "ionic cordova build android --prod --release",
    ```
  - 预(AOT)编译：预编译 Angular 组件的模板。
  - 生产模式：启用生产模式部署到生产环境。
  - 打捆（Bundle）：把这些模块串接成一个单独的捆文件（bundle）。
  - 最小化：移除不必要的空格、注释和可选令牌（Token）。
  - 混淆：使用短的、无意义的变量名和函数名来重写代码。
  - 消除死代码：移除未引用过的模块和未使用过的代码.
### webpack配置修改
```js
// 拿到webpack 的默认配置 剩下的还不是为所欲为
const defaultConfig = require('@ionic/app-scripts/config/webpack.config.js');
// 像这样去修改配置
defaultConfig.prod.resolve.alias = {
  "@config": path.resolve(configPath('prod'))
}
defaultConfig.dev.resolve.alias = {
  "@config": path.resolve(configPath('dev'))
}
```
