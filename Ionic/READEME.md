# Ionic集锦

## 01 Ionic页面跳转以及传参
1. 跳转页面（在home.ts跳转为例）
   
  > 在home.ts中 先引入(NewsPage)
``` js
import { NewsPage } from '../news/news';
```
  > HomePage 默认引入NavController，直接使用this.navCtrl.push跳转
``` js
export class HomePage {
  constructor(public navCtrl: NavController) {

  }
  goNews(){
    // this.navCtrl.push('页面');
    this.navCtrl.push(NewsPage);
  }
}
<!--跳转到morePage上面去-->
  <button ion-button [navPush]="newsPage">界面方式跳转</button>
  <!--通过事件的方式-->
  <button ion-button (click)="goNews()">代码方式跳转</button>
```
2. 传递参数
  > home页面传递参数 
``` js
goNews(){
  // this.navCtrl.push('页面');
  this.navCtrl.push(NewsPage, {
    title : '测试传参'
  });
}
```
  > news页面接受参数
``` js
export class NewsPage {
  titleName: String = ''
  //NavController就是用来管理和导航页面的一个controller
  constructor(public navCtrl: NavController, public navParams: NavParams) {
    this.titleName = navParams.get('title')
  }

  ionViewDidLoad() {
    console.log(this.titleName)
    console.log('ionViewDidLoad NewsPage');
  }

}
```

