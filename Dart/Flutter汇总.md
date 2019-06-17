# Flutter大杂烩

这篇文章会持续更新

`Flutter`用了两个月， 这里记录一下一些比较重要的部分。基础的部分就不写了，基础的估计各大论坛的博主们都已经写的七七八八了。



## 动画

由于移动端的体验相对而言比较重要。 所以动画在这里要详细的说一说。

> 在任何系统的UI框架中，动画实现的原理都是相同的，即：在一段时间内，快速地多次改变UI外观，由于人眼会产生视觉暂留，最终看到的就是一个“连续”的动画，这和电影的原理是一样的，而UI的一次改变称为一个动画帧，对应一次屏幕刷新，而决定动画流畅度的一个重要指标就是帧率FPS（Frame Per Second），指每秒的动画帧数。很明显，帧率越高则动画就会越流畅。一般情况下，对于人眼来说，动画帧率超过16FPS，就比较流畅了，超过32FPS就会非常的细腻平滑，而超过32FPS基本就感受不到差别了。由于动画的每一帧都是要改变UI输出，所以在一个时间段内连续的改变UI输出是比较耗资源的，对设备的软硬件系统要求都较高，所以在UI系统中，动画的平均帧率是重要的性能指标，而在Flutter中，理想情况下是可以实现60FPS的，这和原生应用动画基本是持平的。

### 基础用户

`Flutter`中实现动画非常的简单, 首先了解一下以下几个基本的**动画概念和类**
- `Animation`对象是动画库中的一个核心类，它生成指导动画的值:  `Animation`对象本身和`UI`渲染没有任何关系。`Animation`是一个抽象类，它拥有其当前值和状态（完成或停止）。其中一个比较常用的`Animation`类是`Animation<double>`。
- `AnimationController`管理`Animation`： `AnimationController`是一个特殊的`Animation`对象，在屏幕刷新的每一帧，就会生成一个新的值。默认情况下，`AnimationController`在给定的时间段内会线性的生成从`0.0`到`1.0`的数字。 可以说她就是用来控制动画的。

- `CurvedAnimation` 将过程抽象为一个非线性曲线.
- `Tween`在正在执行动画的对象所使用的数据范围之间生成值, 一般来讲就是动画状态的设置。
- 使用`Listeners`和`StatusListeners`监听动画状态改变。这个可以监听动画是否完成去做出不同的操作

在`Flutter`中我们用不同的`controller`去管理不同的**值**， 这个值就用在`UI`组件上，值的变化会处罚动画的执行。 ==> 这基本上就是`Animation`的一个流程， 下面来看一看例子
```dart
class AnimateHome extends StatefulWidget {
  AnimateHome({Key key}) : super(key: key);

  _AnimateHomeState createState() => _AnimateHomeState();
}

class _AnimateHomeState extends State<AnimateHome> with TickerProviderStateMixin {
  
  ///　AnimationController是一个特殊的Animation对象，在屏幕刷新的每一帧，就会生成一个新的值。默认情况下，
  /// AnimationController在给定的时间段内会线性的生成从0.0到1.0的数字
  AnimationController animationController;
  Animation animation;
  // 控制color的controller
  Animation animationColor;
  // 曲线Controller
  CurvedAnimation curve;

  @override
  void initState() {
    // TODO: implement initState
    super.initState();
    // 动画对象
    animationController = AnimationController(
      /* value: 30,
      lowerBound: 0,
      upperBound: 60, */
      duration: Duration(milliseconds: 1000),
      vsync: this // 防止屏幕外的动画消耗不必要的资源，  值是一个TickerProvider
    );
    
    // 监听动画
    /* animationController.addListener(() {
      print('${animationController.value}');
      // 这里的作用是手动的去重建当前widget 下面我们使用AnimatedWidget去让他自动的重建
      setState(() {
        
      });
    }); */
    // 曲线动画 Curves中内置了大量的不同的Curves动画， 你也可以自定义动画
    curve = CurvedAnimation(parent: animationController, curve: Curves.bounceInOut);

    // 使用Tween 来代替AnimationController中的范围值
    animation = Tween(
      begin: 20.0,
      end: 60.0,
    ).animate(curve);
    // 颜色动画
    animationColor = ColorTween(
      begin: Colors.red,
      end: Colors.red[900]
    ).animate(animationController);
    // 执行动画
    // animationController.forward();
    animationController.addStatusListener((AnimationStatus status) {
      print(status);
    });
  }

  @override
  void dispose() {
    // TODO: implement dispose
    super.dispose();
    animationController.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return Center( 
      child: AnimateHeart(
        animations: [
          animation, 
          animationColor
        ],
        controller: animationController,
      ),
    );
  }
}

class AnimateHeart extends AnimatedWidget {

  final List animations;
  final AnimationController controller;

  AnimateHeart({
    this.animations,
    this.controller
  }): super(listenable: controller);


  @override
  Widget build(BuildContext context) {
    // TODO: implement build
    return IconButton(
        icon: Icon(Icons.favorite),
        iconSize: animations[0].value,
        color: animations[1].value,
        onPressed: () {
          // 判断动画状态
          switch (controller.status) {
            case AnimationStatus.completed:
              // 动画如果完成了那么倒退
              controller.reverse();
              break;
            default:
              controller.forward();
          }
        },
      );
  }
}
```
### 内置的动画组件

`Flutter`本身也内置了大量的动画组件

- `ScaleTransition`：缩放过渡组件

  `ScaleTransition`(
    `Animation<double> scale` 过渡动画 
    `child`
  )

- `AnimatedOpacity`：透明度过渡组件

  `ScaleTransition`(
    `opacity`: 透明度
    `curve`: 过渡动画 
    `duration`: 动画持续时长
    `child`
  )