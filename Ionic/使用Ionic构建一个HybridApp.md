# Ionic开发App中重要的部分

## 写在前面
`APP`赶在了春节之前上线了，所以这次我们分享一下使用`Ionic3 + Angular5`构建一个`Hybird App`过程中的经验。什么是`Hybird App`以及一些技术的选型这里就不讨论了。我每次完成一个部分就写一部分，所以有文章有点长。如果有错误的地方感谢大家指正~

### 为什么选了`Ionic` ?

有些朋友说`Angular/Ionic`不大行，但是我觉的技术没有好坏之分，只有适合不适合。首先在我看来`Ionic`已经在`Hybird App`开发领域立足多年，已经相当的成熟了，我觉的比大部分的解决方案都要好。其次因为我们的App是一个弱交互多展示类型的，`Ionic`
满足我们的需求。最后是因为**如果你想在没有`Android`团队和`IOS`团队支持的情况下独立完成一款APP，那么`Ionic`我觉的是不二之选**。因为`Ionic4`还在`beta`版本，并且是公司项目所以依然选用了稳定的`3.X`版本。

注意：**非基础入门教程**，所以在读这篇文章之前建议你最好先了解`[Angular](https://www.angular.cn/guide/quickstart), [TS](https://www.tslang.cn/docs/home.html), [Ionic](https://ionicframework.com/docs/)`的基础知识，这里主要是希望大家在使用`Ionic`的时候能少走一些弯路。

**由于我自己用的不是很熟练`Rxjs`这一块就没有写，等以后对`Rxjs`的理解更加深刻了再加上**

## Angular汇总部分

既然是基于`Angular`那我们首先来了解一下`Angular`，这个地方积累的是`Angular`中零散的部分。<small>如果内容多的话后期会拆分为单独的部分</small>

### Angular组件生命周期

`Angular`的生命周期

`Hooks`[官方介绍](https://www.angular.cn/guide/lifecycle-hooks)
- `constructor() `： 在任何其它生命周期钩子之前调用。可以用它来注入依赖项，**但不要在这里做正事**。
- `ngOnChanges(changes: SimpleChanges) => void`：    当被绑定的输入属性的值发生变化时调用，**首次调用一定会发生在 `ngOnInit()` 之前**
- `ngOnInit() => void`：  在第一轮 `ngOnChanges()` 完成之后调用。**只调用一次**
- `ngDoCheck() => void`：   在每个变更检测周期中调用，`ngOnChanges()` 和 `ngOnInit()` 之后
- `ngAfterContentInit() => void`： `Angular` 把外部内容投影进组件/指令的视图之后调用。可以认为是外部内容初始化
- `ngAfterContentChecked() => void`：  `Angular` 完成被投影组件内容的变更检测之后调用。可以认为是外部内容更新
- `ngAfterViewInit() => void`： 每当 `Angular` 初始化完**组件视图及其子视图**之后调用。**只调用一次。**
- `ngAfterViewChecked() => void`：每当 `Angular` 做完组件视图和子视图的变更检测之后调用, `ngAfterViewInit()` 和每次 `ngAfterContentChecked()` 之后都会调用。
- `ngOnDestroy() => void`：在 Angular 销毁指令/组件之前调用。

### Angular中内容映射(插槽)的实现

- `<ng-content></ng-content>`默认映射
  这个内容映射方向是`由父组件映射到子组件中`这个就相当于`vue`中的slot，用法也都是一样的：
  ```html
  <!-- 父组件 -->
  <child-component>
    我是父组件中的内容默认映射过来的
  </child-component>
  <!-- 子组件 -->
  <!-- 插槽 -->
    <ng-content>
      
    </ng-content>
  ```
  上面是最简单的默认映射使用方式
- 针对性映射（具名插槽）
  我们也可以通过<ng-content>的`select`属性实现我们的具名插槽。这个是可以根据条件进行填充。`select`属性支持根据`CSS`选择器(ELement, Class, [attribute]...)来匹配你的元素，如果不设置就全部接受，就像下面这样：
  ```html
  <!-- 父组件 -->
  <child-component>
    我是父组件中的内容默认映射过来的
    <header>
      我是根据header来映射的
    </header>
    <div class="class">
      我是根据class来映射的
    </div>
    <div name="attr">
      我是根据attr来映射的
    </div>
  </child-component>

  <!-- 子组件 -->
  <!-- 具名插槽 -->
  <ng-content select="header"></ng-content>
  <ng-content select=".class"></ng-content>
  <ng-content select="[name=attr]"></ng-content>
  ```
- `ngProjectAs`
  上面那些都是映射都是作为直接子元素进行的映射，那要不是呢？ 我想在外面再套一层呢？
  ```html
  <!-- 父组件 -->
  <child-component>
    <!-- 这个时不是直接子节点了 这肯定是不行的 那我们就用到ngProjectAs了-->
    <div>
      <header>
        我是根据header来映射的
      </header>
    </div>
  </child-component>
  ```
  使用`ngProjectAs`,它可以作用于任何元素上。
  ```html
  <!-- 父组件 -->
  <child-component>
    <div ngProjectAs="header">
      <header>
        我是根据ngProjectAs header来映射的
      </header>
    </div>
  </child-component>
  ```

- `ng-content`有一个`@ContentChild`装饰器，可以用来调用和投影内容。但是要注意：只有在`ngAfterContentInit`声明周期中才能成功获取到通过`ContentChild`查询的元素。

既然提到了`ng-content`那我们就来聊一聊`ng-template`和`ng-container`
- `ng-template`
  
  <ng-template> 元素是动态加载组件的最佳选择，因为它**不会渲染任何额外的输出**
  ```html
  <div class="ad-banner-example">
    <h3>Advertisements</h3>
    <ng-template ad-host></ng-template>
  </div>
  ```

- `ng-container`
  <ng-container> 是一个由 `Angular` 解析器负责识别处理的语法元素。 它不是一个指令、组件、类或接口，更像是  `JavaScript` 中 `if` 块中的花括号。一般用来把一些兄弟元素归为一组,它不会污染样式或元素布局，因为 Angular 压根不会把它放进 DOM 中。
  ```html
  <p>
    I turned the corner
    <ng-container *ngIf="hero"><!-- ng-container不会被渲染 -->
      and saw {{hero.name}}. I waved
    </ng-container>
    and continued on my way.
  </p>
  ```
  
### Angular指令
`Angular`中的指令分为`组件`,`属性指令`和`结构形指令`。`属性型指令`用于改变一个 `DOM` 元素的外观或行为，例如`NgStyle`。`结构型指令`的职责是 `HTML` 布局。 它们塑造或重塑 `DOM` 的结构，比如添加、移除或维护这些元素，例如`NgFor`和`NgIf`。

1. 属性型指令
  - 通过`Directive`装饰符把一个类标记为 `Angular` 指令, 该选项提供配置元数据，用于决定该指令在运行期间要如何处理、实例化和使用。[@Directive](https://angular.cn/api/core/Directive)
  - 通过`ElementRef`获取绑定元素的DOM对象，[ElementRef](https://angular.cn/api/core/ElementRef)。
  - 通过`HostListener`响应用户引发的事件，把一个事件绑定到一个宿主监听器，并提供配置元数据。 当宿主元素发出特定的事件时，Angular 就会执行所提供的处理器方法，并使用其结果更新所绑定到的元素。 如果该事件处理器返回 `false`，则在所绑定的元素上执行 `preventDefault`。[HostListener](https://angular.cn/api/core/HostListener)
  - 通过`Input`装饰符把某个类字段标记为输入属性，并且提供配置元数据。 声明一个可供数据绑定的输入属性，在变更检测期间，`Angular` 会自动更新它，[@Input](https://angular.cn/api/core/Input)。
  ```ts
  @Input('appHighlight') highlightColor: string;
  ```
  下面是一个完整的属性形指令的例子
  ```ts
  import {Directive, ElementRef, HostListener, Input} from '@angular/core';

  @Directive({
    selector: '[sxylight]'
  })
  export class SxylightDirective {
    constructor(private el: ElementRef) {
      el.nativeElement.style.backgroundColor = 'yellow';
    }
    // 指令绑定的值
    @Input('sxylight') highlightColor: string;
    // 在指令内部，该属性叫 highlightColor，在外部，你绑定到它地方，它叫 sxylight 这个是绑定的别名

    // 指令宿主绑定的值
    @Input() defaultColor: string;
    // 监听宿主事件
    @HostListener('mouseenter') onMouseEnter() {
      this.highlight(this.highlightColor || this.defaultColor || 'red');
    }
    @HostListener('mouseleave') onMouseLeave() {
      this.highlight(null);
    }
    private highlight(color: string) {
      this.el.nativeElement.style.backgroundColor = color;
    }
  }
  ```
2. 结构型指令
  - 星号（*）前缀：这个东西其实是语法糖，`Angular` 把 `*ngIf` 属性 翻译成一个 `<ng-template>` 元素 并用它来包裹宿主元素。
  - `<ng-template>`: 它是一个 Angular 元素，用来渲染 HTML。 它永远不会直接显示出来。 事实上，在渲染视图之前，Angular 会把 <ng-template> 及其内容替换为一个注释。
  - `<ng-container>`： 它是一个分组元素，但它不会污染样式或元素布局，因为 `Angular` 压根不会把它放进 `DOM` 中。
  - `TemplateRef`： 可以使用`TemplateRef`取得 `<ng-template>` 的内容，[TemplateRef<any>](https://angular.cn/api/core/TemplateRef)
  - `ViewContainerRef`： 可以通过`ViewContainerRef`来访问这个视图容器，[ViewContainerRef](https://angular.cn/api/core/ViewContainerRef)。
  
  完整示例
  ```ts
  import { Directive, Input, TemplateRef, ViewContainerRef } from '@angular/core';
  /**
  * Input, TemplateRef, ViewContainerRef 这三个模块是构建一个结构型指令必须的模块
  * Input： 传值
  * TemplateRef: 表示一个内嵌模板，它可用于实例化内嵌的视图。 要想根据模板实例化内嵌的视图，请使用 ViewContainerRef 的 createEmbeddedView() 方法。
  * ViewContainerRef: 表示可以将一个或多个视图附着到组件中的容器。
  */
  @Directive({
    selector: '[structure]' // Attribute selector
  })
  export class StructureDirective {
    private hasView = false
    @Input()
    set structure(contion: boolean) {
      console.log(contion)
      if (!contion && !this.hasView) {
        this.viewCon.createEmbeddedView(this.template) // 实例化内嵌视图并插入到容器中
        this.hasView = true
      } else if (contion && this.hasView) {
        this.viewCon.clear() // 销毁容器中的所有试图
        this.hasView = false
      }
    }

    constructor(
      private template: TemplateRef<any>,
      private viewCon: ViewContainerRef
    ) {
      console.log('Hello StructureDirective Directive');
    }

  }

  ```

## **Angular中的Module**
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
```js
app.module.ts
└───@NgModule
    └───declarations                // 告诉Angular哪些模块属于NgModule
    │───imports                     // 导入需要使用的模块
    │───bootstrap                   // 启动模块
    │───entryComponents             // 定义组建时应该被编译的组件
    └───providers                   // 服务配置
```

`entryComponents`:`Angular`使用`entryComponents`来启用`tree-shaking`，即只编译项目中实际使用的组件，而不是编译所有在`ngModule`中声明但从未使用的组件。离线模板编译器`（OTC）`只生成实际使用的组件。如果组件不直接用于模板，`OTC`不知道是否需要编译。有了`entryComponents`，你可以告诉`OTC`也编译这些组件，以便在运行时可用。
## Ionic工程目录结构
首先来看项目目录
```js
Ionic-frame
│   build                   // 打包扩展
│   platforms               // Android/IOS 平台代码
│   plugins                 // cordova插件
│   resources
└───src                     // 业务逻辑代码
│   │   app                 // 启动组件
│   │   assets              // 资源
│   │   components          // 公共组件
│   │   config              // 配置文件
│   │   directive           // 公共指令
│   │   interface           // interface配置中心
│   │   pages               // 页面
│   │   providers           // 公共service
│   │   service             // 业务逻辑service
│   │   shared              // 共享模块
│   │   theme               // 样式模块
│   │   index.d.ts          // 声明文件
└───www                     // 打包后静态资源
```

## Ionic视图生命周期

生命周期的重要性不用多说，这是`Ionic`官网的[介绍](https://ionicframework.com/docs/api/navigation/NavController/)
- `constrctor => void`:         构造函数启动，构造函数在ionViewDidLoad之前被触发
- `ionViewDidLoad => void`：    资源加载完毕时触发。ionViewDidLoad只在第一次进入页面时触发**只触发一次**
- `ionViewWillEnter => void`：  页面即将给进入时触发**每次都会触发**
- `ionViewDidEnter => void`：   进入视图之后出发**每次都会触发**
- `ionViewWillLeave => void`：  即将离开（仅仅是触发要离开的动作）时触发**每次都会触发**
- `ionViewDidLeave => void`：   已经离开页面时触发**每次都会触发**
- `ionViewWillUnload => void`： 在页面即将被销毁并删除其元素时触发
- `ionViewCanEnter => boolean`：在视图可以进入之前运行。 这可以在经过身份验证的视图中用作一种“保护”，您需要在视图可以进入之前检查权限
- `ionViewCanLeave => boolean`：在视图可以离开之前运行。 这可以在经过身份验证的视图中用作一种“防护”，您需要在视图离开之前检查权限

**注意： 当你想使用`ionViewCanEnter/ionViewCanLeave`进行对路由的拦截时，你需要返回一个`Boolen`。返回`true`进入下一个视图，返回`fasle`留在当前视图。**

可以按照下面的代码感受一下生命周期的顺序
```js
constructor(public navCtrl: NavController) {
  console.log('触发构造函数')
}

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

## 项目配置文件设置

`Ionic3.X`中并没有提供相应的的配置文件，所以我们需要自己按照下面步骤手动去添加配置文件来对项目进行配置。
1. 新增`config`目录
```js
src
  |__config
      |__config.dev.ts
      |__config.prod.ts
```
`config.dev.ts` / `config.prod.ts`
```js
export const CONFIG = {
  BASE_URL            : 'http://XXXXX/api', // API地址
  VERSION             : '1.0.0'
}
```
2. 在根目录下新增`build`文件夹，在文件夹中新增`webpack.config.js` config文件
```js
const fs = require('fs')
const chalk =require('chalk')
const webpack = require('webpack')
const path = require('path')
const defaultConfig = require('@ionic/app-scripts/config/webpack.config.js')

const env = process.env.IONIC_ENV
/**
 * 获取配置文件
 * @param {*} env 
 */
function configPath(env) {
  const filePath = `./src/config/config.${env}.ts`
  if (!fs.existsSync(filePath)) {
    console.log(chalk.red('\n' + filePath + ' does not exist!'));
  } else {
    return filePath;
  }
}
// 定位当前文件
const resolveDir = filename => path.join(__dirname, '..', filename)
// 其他文件夹别名
let alias ={
  "@": resolveDir('src'),
  "@components": resolveDir('src/components'),
  "@directives": resolveDir('src/directives'),
  "@interface": resolveDir('src/interface'),
  "@pages": resolveDir('src/pages'),
  "@service": resolveDir('src/service'),
  "@providers": resolveDir('src/providers'),
  "@theme": resolveDir('src/theme')
}
console.log("当前APP环境为："+process.env.APP_ENV)
let definePlugin =  new webpack.DefinePlugin({
  'process.env': {
    APP_ENV: '"'+process.env.APP_ENV+'"'
  }
})
// 设置别名
defaultConfig.prod.resolve.alias = {
  "@config": path.resolve(configPath('prod')), // 配置文件
  ...alias
}
defaultConfig.dev.resolve.alias = {
  "@config": path.resolve(configPath('dev')),
  ...alias
}

// 其他环境
if (env !== 'prod' && env !== 'dev') {
  defaultConfig[env] = defaultConfig.dev
  defaultConfig[env].resolve.alias = {
    "@config": path.resolve(configPath(env))
  }
}
// 删除sourceMaps

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

如果过我们想修改`Ionic`中其他的`webpack`配置， 那么可以像上面那种形式来进行修改。
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

## Ionic路由
- 首页设置
  有时候我们需要设置我们第一次显示得页面。那这样我们就需要使用`NavController`来设置
  ```ts
  // app.component.ts
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
- 相关常用API
  1. `navCtrl.push(OtherPage, param)`： 跳转页面
  2. `navCtrl.pop()`: `Removing a view` 移除当前View，相当于返回上一个页面
  3. **路由中参参数相关**
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

## provider(service)使用
> 当重复的需要一个类中的方法时，可封装它为服务类，以便重复使用，如http。

`provider`，也叫`service`。前者是`ionic`的叫法，后者是`ng`的叫法。建议仔细得学一下`Angular`
- 创建`Provider`
`Ionic`提供了创建指令
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
    alert("服务类的方法")
  }
}
```
- 使用`provider` 

如果是顶级的服务(全局通用服务)，需要在`app.module.ts`的`providers`中注册后然后使用
```ts
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

## Ionic事件系统
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
**注意点**: <font color="red">1： 订阅必须再发布之前，不然接收不到。打个比喻：比如微信公众号，你要先关注才能接收到它的推文，不然它再怎么发推文，你也收不到。2： `subscribe`中得`this`指向是有点问题的，这里需要注意一下。</font>


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
  
  如果我们`把一个服务实例的作用域被限制在父组件和其子组件内，这个组件子树之外的组件将无法访问该服务或者与它们通讯`。父子共享一个服务，那么我们可以利用该服务在**家庭内部**实现**双向通讯**。
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
公共组件设置，`Angular`倡导的是模块化开发，所以公共组件的注册可能稍有不同。

在这里我们根据`Angular`提供的`CommonModule`共享模块，我们要知道他干了什么事儿：

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

使用了`shared`模块仅仅需要在`xxx.module.ts`中引用即可,然后又就可以使用`shared`中所有引入的公共模块。
```ts
import { NgModule } from '@angular/core';
import { IonicPageModule } from 'ionic-angular';
import { XXXPage } from './findings';
import { SharedModule } from '@shared/shared.module';

@NgModule({
  declarations: [
    XXXPage,
  ],
  imports: [
    SharedModule,
    IonicPageModule.forChild(FindingsPage),
  ]
})
export class XXXPageModule {}
```

## http部分
`Ionic`中的http模块是直接采用的`HttpClient`这个模块。这个没什么可说的，我们只需要根据我们的需求对`service`进行修改即可，例如可以把`http`改成了更加灵活的`Promise模式`。你也可以用`Rxjs`的模式来实现。**下面这个是个简单版本的实现**：
```ts
import { TokenServie } from './token.service';
import { StorageService } from './storage.service';
import { HttpClient, HttpHeaders, HttpParams } from '@angular/common/http'
import { Injectable, Inject } from '@angular/core'
import {ReturnObject, Config} from '../interface/index' // 返回数据类型和配置文件
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
}
```

## `Cordova插件使用`

`Ionic`提供了丰富的基于`cordova`的插件,[官网介绍](https://ionicframework.com/docs/native)，使用起来也很简单。

下载`Cordova`插件
```bash
cordova add plugin plugin-name -D
npm install @ionic-native/plugin-name
```
使用插件(**从`@ionic-native/plugin-name`中导入**)
```ts
import { StatusBar } from '@ionic-native/status-bar';
constructor(private statusBar: StatusBar) {
    //沉浸式并且悬浮透明
    statusBar.overlaysWebView(true);
    // 设置状态栏颜色为默认得黑色 适合浅色背景
    statusBar.styleDefault() 
    // 浅色状态栏 适合深色背景
    // statusBar.styleLightContent() 
}
```

## 优化部分
项目写完了，不优化一下 心里怪难受的。
- `App`启动页体验优化

`Ionic App`毕竟是个混合App,毕竟还没有达到秒开级别。所以这个时候我们需要启动页来帮助我们提升用户体验,首先在`config.xml`种配子我们的启动页相关配置
```xml
<preference name="ShowSplashScreenSpinner" value="false" /> <!-- 隐藏加载时的loader -->
<preference name="ScrollEnabled" value="false" /> <!-- 禁用启动屏滚动 -->
<preference name="SplashMaintainAspectRatio" value="true" /> <!-- 如果值设置为 true，则图像将不会伸展到适合屏幕。如果设置为 false ，它将被拉伸 -->
<preference name="FadeSplashScreenDuration" value="1000" /><!-- fade持续时长 -->
<preference name="FadeSplashScreen" value="true" /><!-- fade动画 -->
<preference name="SplashShowOnlyFirstTime" value="false" /><!-- 是否只第一次显示 -->
<preference name="AutoHideSplashScreen" value="false" /><!-- 自动隐藏SplashScreen -->
<preference name="SplashScreen" value="screen" />
<platform name="android">
    <allow-intent href="market:*" />
    <icon src="resources/android/icon/icon.png" />
    <splash src="resources/android/splash/screen.png" /><!-- 启动页路径 -->
    <!-- 下面是各个分辨率的兼容 -->
    <splash height="800" src="resources/android/splash/screenh.png" width="480" />
    <splash height="1280" src="resources/android/splash/screenm.png" width="720" />
    <splash height="1600" src="resources/android/splash/screenxh.png" width="960" />
    <splash height="1920" src="resources/android/splash/screenxxh.png" width="1280" />
    <splash height="2048" src="resources/android/splash/screenxxxh.png" width="1536" />
</platform>
```
我在这里关闭了自动隐藏`SplashScreen`，因为她的判定条件是一旦`App`出事还完毕就隐藏，这显然不符合我们的要求。我们需要的是我们的`Ionic WebView`程序启动之后再隐藏。所以我们在`app.component.ts`中借助`@ionic-native/splash-screen`来进行这个操作.
```ts
platform.ready().then(() => {
      // 延迟1s隐藏启动屏幕
      setTimeout(() => { 
        splashScreen.hide()
      }, 1000)
    })
```
这样一来我们就可以完美的欺骗用户，体验能好点。

<!-- ### 组件懒加载 -->

### 打包优化
- **新增--prod参数**
    ```bash
    "build:android": "ionic cordova build android --prod --release",
    ```
  - 预(AOT)编译：预编译 Angular 组件的模板。
  - 生产模式：启用生产模式部署到生产环境。
  - 打捆（`Bundle`）：把这些模块串接成一个单独的捆文件（`bundle`）。
  - 最小化：移除不必要的空格、注释和可选令牌（`Token`）。
  - 混淆：使用短的、无意义的变量名和函数名来重写代码。
  - 消除死代码：移除未引用过的模块和未使用过的代码.

## App打包

我认为打包`APK`对于一些不了解服务端和`Android`的前端工程师来说还是比较费劲的。下面我们来仔细的说一说这个部分。

### 环境配置
第一步进行各个环境的配置

1. `Node`安装/配置环境变量（我相信这个你已经弄完了）
2. `jdk`安装 (无需配置环境变量)
   
   jdk是`java`的开发环境支持，你可以在[这里](https://pan.baidu.com/s/1MtZDLklTr7nn_kyPseJU3Q)下载， 提取码：`9p74`。

   下载完成后，解压，直接按照提示安装，全局点确定，不出意外，最后的安装路径为：`C:\Program Files\Java`jdk安装完成，在cmd中，输入`java -version`验证是否安装成功。我这边是修改了安装路径，如果你不熟悉的话还是不要修改安装路径。出现了下面的`log`表示安装成功
   
   ![jdk](./jdk.png 'jdk')

3. **`SDK`安装/配置环境变量**：这一部分是重点，稍微麻烦一些。
  
  先[下载](https://pan.baidu.com/s/1MtZDLklTr7nn_kyPseJU3Q)。
  
  解压后将重命名的文件夹，跟`jdk`放在一个父目录，便于查找：`C:\Program Files\SDK`

  接着配置环境变量，**我的电脑——右键属性——-高级系统设置——-环境变量**。

  在下面的**系统变量(s)**中，新建，键值对如下：
  ```bash
  name: ANDROID_HOME
  key: C:\Program Files\SDK
  ```
  ![sdk](./sdk.png 'sdk')
  
  新建完系统变量之后在`path`中加入全局变量。

  ![sdk_path](./sdk_path.png 'sdk_path')

  在控制台中输入`android -h`，出现下面的日志，表示`sdk`安装成功

  ![sdk_ok](./sdk_ok.png 'sdk_ok')

  接下来我们使用`Android Studio进行SDK下载`，`Adnroid Studio`[下载地址](http://www.android-studio.org/)，`studio`安装完之后就要安装`Android SDK Tools,Android SDK platform-tools,Android SDK Build-tools`这些工具包和`SDK platform`


  ![studio](./studio.png 'studio')

  ![studio_sdk](./studio_sdk.png 'studio_sdk')

4. **`gradle`安装/配置环境变量**
  
  在`SDK`都安装完了之后我们再进行`gradle`的安装和配置。

  先在[官网](http://services.gradle.org/distributions/)或者在[这里](https://pan.baidu.com/s/1MtZDLklTr7nn_kyPseJU3Q)下载

  然后同样安装在`JDK,SDK`的目录下，便于查找。
  和`SDK`同样的配置环境变量：
  ```
  GRADLE_HOME=C:\Program Files\SDK\gradle-4.1
  ;%GRADLE_HOME%\bin
  ```
  测试命令（查看版本）：`gradle -v` 出现下面的日志，表示安装成功

  ![gradle](./gradle.png 'gradle')

### 进行打包
打包之前的环境准备工作都已经做完了，接下来我们进行打包`apk。
1. 安装`cordova`
```bash
npm i cordova -g
```
2. 在项目中创建`Android`工程，在`Ionic`项目中执行下面命令
```bash
ionic cordova platform add android
```
![platfrom](./platfrom.png 'platfrom')

这可能是一个很漫长的过程，你要耐心等待，毕竟曙光就在眼前了。

3. 创建完`Android`项目之后项目的`platform`文件夹下会多出来一个`android`文件夹。这下接着执行打包命令。
```
ionic cordova build android
```
然后你会看到控制台疯狂输出，最后出现下图表明你已经打包出来一个**未签名**的安装包

4. `APK`签名
`APK`不签名是没法发布的。这个有两种方法

 - 使用`jdk`签名，这里不多说，想了解的可以看[这篇文章](https://blog.csdn.net/u013254166/article/details/79290351)
 - 使用`Android Studio`打签名包。
  
  在`AS`上方工具栏`build`中选取`Generate Signed APK`首先创建一个签名文件
  
  ![key](./key.png 'key')
  
  生成完之后可以直接用`AS`打签名包

  ![apk](./apk.png 'apk')

  点击`locate`就能看到我们的`apk`包了~ 至此我们的`Android`就ok了，IOS的之后再补上。

## 简单APP服务器更新（简单示例）

由于`Android`的要求不如苹果那么严，我们也可以通过自己的服务器进行程序的更新。下面就是实现一个比较简单的更新`Service`

更新我们主要是使用到下面几个`Cordova`插件
- `cordova-plugin-file-transfer / @ionic-native/file-transfer`: 线上文件的下载和存储（官方推荐使用`XHR2`，有兴趣的可以看一看）
- `cordova-plugin-file-opener2 / @ionic-native/file-opener`: 用于打开APK文件
- `cordova-plugin-app-version / @ionic-native/app-version`： 用于获取app的版本号
- `cordova-plugin-file / @ionic-native/file`：操作app上的文件系统
- `cordova-plugin-device / @ionic-native/device`：获取当前设备信息，主要用于平台的区分

在下载完插件之后我们来实现一个**比较简陋**的版本更新`service`,具体解释我会写在代码注释中,主要分成两部分，一部分是具体的更新操作`update.service.ts`, 另一部分是用于存放数据的`data.service.ts`
**`data.service.ts`**
```ts
/*
 * @Author: etongfu
 * @Description: 设备信息
 * @youWant: add you want info here
 */
import { Injectable } from '@angular/core';
import { Device } from '@ionic-native/device';
import { File } from '@ionic-native/file';
import { TokenServie } from './token.service';
import { AppVersion } from '@ionic-native/app-version';

@Injectable()
export class DataService {
  /******************************APP数据模块******************************/
  // app 包名
  private packageName: string = '' 
  // app 版本号
  private appCurrentVersion: string =  '---'
  // app 版本code
  private appCurrentVersionCode:number = 0
  // 当前程序运行平台
  private currentSystem: string
  // 当前userId
  // app 下载资源存储路径
  private savePath: string
  //  当前app uuid
  private uuid: string

  /******************************通用数据模块******************************/
  constructor (
    public device: Device,
    public file: File,
    public app: AppVersion,
    public token: TokenServie,
    public http: HttpService
  ) {
    // 必须在设备准备完之后才能进行获取
    document.addEventListener("deviceready", () => {
      // 当前运行平台
      this.currentSystem = this.device.platform
      // console.log(this.device.platform)
      // app版本相关信息
      this.app.getVersionNumber().then(data => {
        //当前app版本号  data，存储该版本号
        if (this.currentSystem) {
          // console.log(data)
          this.appCurrentVersion = data
        }
      }, error => console.error(error))
      this.app.getVersionCode().then((data) => {
        //当前app版本号数字代码 
        if (this.currentSystem) {
          this.appCurrentVersionCode = Number(data)
        }
      }, error => console.error(error))
      // app 包名
      this.app.getPackageName().then(data => {
          //当前应用的packageName：data，存储该包名
          if (this.currentSystem) {
            this.packageName = data;
          }
      }, error => console.error(error))
      // console.log(this.currentSystem)
      // file中的save path 根据平台进行修改地址
      this.savePath = this.currentSystem === 'iOS' ? this.file.documentsDirectory : this.file.externalDataDirectory;

    }, false);
  }
  /**
   * 获取app 包名
   */
  public getPackageName () {
    return this.packageName
  }
  /**
   * 获取当前app版本号
   * @param hasV 是否加上V标识
   */
  public getAppVersion (hasV: boolean = true): string {
    return hasV ? `V${this.appCurrentVersion}` : this.appCurrentVersion
  }
  /**
   * 获取version 对应的nuamber 1.0.0 => 100
   */
  public getVersionNumber ():number {
    const temp = this.appCurrentVersion.split('.').join('')
    return Number(temp)
  }
  /**
   * 获取app version code 用于比较更新使用
   */
  public getAppCurrentVersionCode (): number{
    return this.appCurrentVersionCode
  }
  /**
   * 获取当前运行平台
   */
  public getCurrentSystem (): string {
    return this.currentSystem
  }
  /**
   * 获取uuid
   */
  public getUuid ():string {
    return this.uuid
  }
  /**
   * 获取存储地址
   */
  public getSavePath ():string {
    return this.savePath
  }
}
```
**`update.service.ts`**
```ts
/*
 * @Author: etongfu
 * @Email: 13583254085@163.com
 * @Description: APP简单更新服务
 * @youWant: add you want info here
 */
import { HttpService } from './../providers/http.service';
import { Injectable, Inject } from '@angular/core'
import { AppVersion } from '@ionic-native/app-version';
import { PopSerProvider } from './pop.service';
import { DataService } from './data.service';
import {Config} from '@interface/index'
import { FileTransfer, FileTransferObject } from '@ionic-native/file-transfer';
import { FileOpener } from '@ionic-native/file-opener';
import { LoadingController } from 'ionic-angular';

@Injectable()
export class AppUpdateService {

  constructor (
    @Inject("CONFIG") public CONFIG:Config, 
    public httpService: HttpService,
    public appVersion: AppVersion,
    private fileOpener: FileOpener,
    private transfer: FileTransfer,
    private popService: PopSerProvider, // 这就是个弹窗的service
    private dataService: DataService,
    private loading:LoadingController
  ) {

  }
  /**
   * 通过当前的字符串code去进行判断是否有更新
   * @param currentVersion 当前app version
   * @param serverVersion 服务器上版本
   */
  private hasUpdateByCode (currentVersion: number, serverVersion:number):Boolean {
    return serverVersion > currentVersion
  }
  /**
   * 查询是否有可更新程序
   * @param noUpdateShow  没有更新时显示提醒
   */
  public checkForUpdate (noUpdateShow: boolean = true) {
    // 拦截平台
    return new Promise((reslove, reject) => {
      // http://appupdate.ymhy.net.cn/appupdate/app/findAppInfo?appName=xcz&regionCode=370000
      // 查询app更新
      this.httpService.get(this.CONFIG.CHECK_URL, {}, true).then((result: any) => {
        reslove(result)
        if (result.succeed) {
          const data = result.appUpload
          const popObj = {
            title: '版本更新',
            content: ``
          }
          console.log(`当前APP版本：${this.dataService.getVersionNumber()}`)
          // 存在更新的情况下
          if (this.hasUpdateByCode(this.dataService.getVersionNumber(), data.versionCode)) {
          // if (this.hasUpdateByCode(101, data.versionCode)) {
            let title = `新版本<b>V${data.appVersion}</b>可用，是否立即下载?<h5 class="text-left">更新日志</h5>`
            // 更新日志部分
            let content = data.releaseNotes
            popObj.content = title + content
            // 生成弹窗
            this.popService.confirmDIY(popObj, data.isMust === '1' ? true: false, ()=> {
              this.downLoadAppPackage(data.downloadPath)
            }, ()=> {
              console.log('取消');
            })
          } else {
            popObj.content = '已是最新版本！'
            if(!noUpdateShow) {
              this.popService.confirmDIY(popObj, data.isMust === '1' ? true: false)
            }
          }
        } else {
          // 接口响应出现问题 直接提醒默认最新版本
          if(!noUpdateShow) {
            this.popService.alert('版本更新', '已是最新版本！')
          }
        }
        }).catch((err) => {
          console.error(err)
          reject(err)
        })
      })
  }
  /**
   * 下载新版本App
   * @param url: string 下载地址
   */
  public downloadAndInstall (url: string) {
    let loading = this.loading.create({
      spinner: 'crescent',
      content: '下载中'
    })
    loading.present()
    try {
      if (this.dataService.getCurrentSystem() === 'iOS') {
        // IOS跳转相应的下载页面
        // window.location.href = 'itms-services://?action=download-manifest&url=' + url;
      } else {
        const fileTransfer: FileTransferObject = this.transfer.create();
        fileTransfer.onProgress(progress =>{
          // 展示下载进度
          const present = new Number((progress.loaded / progress.total) * 100);
          const presentInt = present.toFixed(0);
          if (present.toFixed(0) === '100') {
            loading.dismiss()
          } else {
            loading.data.content = `已下载 ${presentInt}%`
          }
        })
        const savePath = this.dataService.getSavePath() + 'xcz.apk';
        // console.log(savePath)
        // 下载并且保存
        fileTransfer.download(url,savePath).then((entry) => {
          //
          this.fileOpener.open(entry.toURL(), "application/vnd.android.package-archive")
          .then(() => console.log('打开apk包成功！'))
          .catch(e => console.log('打开apk包失败！', e))
        }).catch((err) => {
          console.error(err)
          console.log("下载失败");
          loading.dismiss()
          this.popService.alert('下载失败', '下载异常')
        })
      }
    } catch (error) {
      this.popService.alert('下载失败', '下载异常')
      // 有异常直接取消dismiss
      loading.dismiss()
    }
  }
}
```
以上我们就可以根据直接调用`service`去进行更新
**`app.component.ts`**
```ts
// 调用更新
this.appUpdate.checkForUpdate()
```

## App真机调试

说实在的，`Hybird`真机调试是真的痛苦。目前比较流行的方式是以下两种调试方式
- `Chrome Inspect`调试
依靠`chrome`的强大能力，我们可以把`App`中的`WebView`中的内容完全的显示在chrome端。可以在web端控制我们的app中的网页，还是先当的炫酷的。以下是操作步骤
  1. 在chrome中打开`chrome://inspect/#devices`
  ![Ionic](./step1.png 'ionic')
  2. 连接设备，注意**第一次连接的话，是需要fan墙的，否则会出现`404`等等的问题**
  ![Ionic](./step2.png 'ionic')
  3. 在连接的设备中安装需要调试的App，接着`Chrome`会自动找到需要调试的`WebView`
  4. 愉快的开始调试
  ![Ionic](./step3.png 'ionic')

- 使用`VConsole`进行调试

  这个就更简单了，直接`npm install vconsole`这个库， 然后在`app.component.ts`进行引用
  ```ts
  import VConsole from 'vconsole'
  export class MyApp {
  constructor() {
      platform.ready().then(() => {
        console.log(APP_ENV)
        // 调试程序
        APP_ENV === 'debug' && new VConsole()
      })
    }
  }
  ```

  效果如下

  ![Ionic](./vconsole.png 'ionic')
  
## Ionic中的特殊部分(坑)
- 静态资源路径问题

如果在打完包之后静态路径出来问题，没有加载出来的话要注意以下情况
```html
<!-- html中的img标签直接引用图片处理   -->
<img src="./assets/xxx.jpg"/>
<!-- 或者这样 -->
<img src="assets/imgs/timeicon.png" style="width: 1rem;">
```
```css
/*scss文件中要使用绝对路径*/
.bg{
  background-image: url("../assets/xxx.jpg")
}
```
- `Android API版本修改`
Ionic中现在默认的SDK版本太高了，有些低版本的机器没发安装需要修改的有以下这么几个部分
```xml
<!-- platforms/android/project.properties  -->
target=android-26
<!-- 和platforms/android/CordovaLib/project.properties  -->
target=android-26
```

- 关于`SDK`和`cordova`插件中的坑(暂时不写)
  
这个东西真的是坑的一塌糊涂，以`cordova-plugin-file-opener2`为例

- `AS3.0`打包之后`Android7.0`以下的手机无法安装
  
这个不能算是`Ionic`的坑，要算也得是`Android Studio3.0`的坑，之前因为不了解在打包的时候下面的选项并没有勾选上

![打包1](./pack1.png '打包1')

不加上的时候一直在`Android7.0`以下都没法安装，一直以为是项目代码的问题，没想到是设置的问题，加上了`V1`选项之后打也就可以了，查了一下原因如下。

上图中提供的选项其实是`签名版本选择`，在`AS3.0`的时候新增的选项。

`Android 7.0`中引入了`APK Signature Scheme v2`，`v1`呢是`jar Signature`来自`JDK`
V1：应该是通过ZIP条目进行验证，这样APK 签署后可进行许多修改 - 可以移动甚至重新压缩文件。
V2：验证压缩文件的所有字节，而不是单个 ZIP 条目，因此，在签名后无法再更改(包括 `zipalign`)。正因如此，现在在编译过程中，我们将压缩、调整和签署合并成一步完成。好处显而易见，更安全而且新的签名可缩短在设备上进行验证的时间（不需要费时地解压缩然后验证），从而加快应用安装速度。

如果不勾选V1,**那么在7.0以下会直接安装完显示未安装，7.0以上则使用了V2的方式验证**。如果勾选了V1，**那么7.0以上就不会使用更加安全的快速的验证方式。**

也可以在app目录下的`build.gradle`中进行配置
```gradle
signingConfigs {
    debug {
        v1SigningEnabled true
        v2SigningEnabled true
    }
    release {
        v1SigningEnabled true
        v2SigningEnabled true
    }
}
```

# 总结 

这么一番折腾下来，越到了不少坑。但是也都一一解决了。使用`Ionic`最大的感触就是`TS`+`Angular`的模块化开发模式很舒服。而且开发速度上也不至于太慢，对`Angular`感兴趣的朋友我认为还是可以一试的。

示例代码请稍后

[原文地址](https://github.com/QDMarkMan/CodeBlog/blob/master/Ionic/%E4%BD%BF%E7%94%A8Ionic%E6%9E%84%E5%BB%BA%E4%B8%80%E4%B8%AAHybridApp.md)  如果觉得有用得话给个⭐吧