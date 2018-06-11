---
title: Angular2学习笔记-组件02
date: 2017-11-23
categories:
- Angular2
tags: 
- Angular2
- Angular2学习笔记
---

### 三、组件与模块的交互
```
//模块包含组件，指令，路由，服务

//模块的构成
//app.module.ts
import { NgModule} from '@angular/core';
import {BrowserModule} from '@angular/platform-browser';
import {ContactComponent} from './contact.component'

@NgModule({
    imports:[BrowserModule],  //引入依赖模块或路由
    declarations:[ContactComponent],  //指定这个模块的视图类
    bootstrap:[ContactComponent],
    //providers://依赖的服务
    //exports://导出视图类  ,当该模块被引入到外部模块时，可以指定可以使用的视图类
})

export class AppModule{}


//视图类的引入
//使用declarations可以将指令，组件，管道等视图类引入模块，从而当前模块或其子模块的组件等都可以使用这个视图类的相关东西
NgModule({
    //如果ListComponent组件使用HeaderComponent等组件的内容，那么就要在模块中引入，这样划分清晰，耦合性低
   declarations:[
       HeaderComponent,
       FooterComponent,
       ListComponent
   ]
})


//模块间的视图类（如组件）如何引用
//a模块声明导出的视图类，b模块导入a模块，并声明导入a模块，则在b模块的组件就可以使用a模块的组件了
//a.module.ts
@NgModule({
    declarations:[aComponent],
    exports:[aComponent]
})

//b.module.ts
import {aModule} from './a.module';
@NgModule({
    declarations:[bComponent],
    imports:[aModule]
})

//引入服务的方式
/**
 * 1、@NgModule的provides
 * 2、@Component 的providers
 */


```

### 四、父子组件的交互

```
//组件数据的流入流出是根据输入与输出语法来处理的
/**
 * 1、类中使用@Input ,@Output
 * 2、修饰符中使用inputs与outputs属性,其值为字符串数组
 * 相应的属性需要时组件类的成员变量
 */

//item.component.ts
export class ItemComponent implements OnInit{
    @Input() contact:any={};
    @Output() routerNavigate=new EventEmitter<number>();
}
//在父级组件中使用
//list.component.html
/*
* <li *ngFor="let contact of contacts">
*     <list-item [contact]="contact" (routeNavigate)="routerNavigator($event)"></list-item>
*</li>
* */

//第二种写法
@Component({
    //..
    inputs:['contact'],
    outputs:['routerNavigate']
})

//父组件通过子组件的输入属性流入数据，子组件可以接受或拦截


//可以通过setter与getter处理与返回输入值
//listItem.component.ts
@Component({
  selector:'list-item',
    template:`
        <div>
            <span>{{contactObj.name}}</span>
            <span>{{contactObj.telNum}}</span>
        </div>
    `
})

export class ListItemComponent implements OnInit{
    _contact:object={};
    @input()
    //set与get相当于在组件类的原型对象上设置了一个contactObj属性，在组件使用的是contactObj
    set contactObj(contact:object){  //处理输入的数据
        this._contact.name=(contact.name&&contact.name.trim())||'no name set';
        this._contact.telNum=contact.telNum||'000-000';
    }
    get contactObj(){return this._contact;} //返回输入的数据
}



//ngOnChanges监听数据变化
//父组件的数据可以在子组件中记录变化并处理，当然其在自身也是可以用的
//ngOnChanges用于及时响应angular在属性绑定中发生的数据变化。这个方法接受一个对象参数，包含当前值与变化值
//detail.component.ts
import {Component} from '@angular/core';

@Component({
    selector:'detail',
    //change-log组件打印出数据的变化
    template:`
        <a class="edit" (click)="editContact()">编辑</a>
        <change-log [contact]="detail"></change-log>
    `
})
export class DetailComponenet implements OnInit{
    detail:any={};
    editContact(){
        //...
        this.detail=data;//修改后的数据
    }
}

//changelog.component.ts
import { Component,Input,OnChanges,SimpleChanges} from '@angular/core';

@Component({
    selector:'change-log',
    template:`
        <h4></h4>
        <ul>
            <li *ngFor="let change of changes">{{change}}</li>
        </ul>
    `
})
export class ChangeLogComponent implements OnChanges{
    @Input() contact :any={};
    changes:string[]=[];
    ngOnchanges(changes:{[propKey:string]:SimpleChanges}){
        let log:string[]=[];
        for(let propName in changes){
            let changedProp=changes[propName],
                from=JSON.stringify(changedProp.previousValue),
                to=JSON.stringify(changedProp.currentValue);
            log.push(`${propName} changed from ${from} to ${to}`);
        }
        this.changes.push(log.join(','));
    }
}


//子组件如何向父组件传递数据

//使用事件传递的方式，子组件通过EventEmitter自定义事件，父组件然后订阅(以普通的事件订阅方式),中间的数据媒介是组件@output属性
//例:子组件点击收藏，父组件完成收藏的操作
//父组件CollectionComponent.ts

import {Component} from "@angular/core";

@Component({
    selector:'collection',
    //使用组件contact-collect
    template:`
        <contact-collect [contact]="detail" (onCollect)="collectTheContact($event)"></contact-collect>
    `
})

export class CollectionComponent implements Onint{
    detail:any={};
    //收藏操作处理逻辑
    collectTheContact(){
        this.detail.collection==0?this.detail.collection=1:this.detail.collection=0;
    }
}

//子组件contactCollection.component.ts
import { Component ,EventEmitter,Input,Output } from '@angular/core';

@Component({
    selector:'contact-collect',
    template:`
       <i [ngClass]="{collectd:contact.collection}" (cilck)="collectTheContact()">收藏</i>
    `
})
export class ContactCollecComponent{
    @Input() contact:any={};
    @Output() onCollect=new EventEmitter<boolean>();//声明事件绑定的输出特性，相当于注册一个事件
    collectTheContact(){
        this.onCollect.emit();
    }
}

//使用#号声明局部变量来获取子组件实例，从而访问它的公共成员
//注意这种方式只能在组件模板里使用
//父组件
//...
@Component({
    selector:'collection',
    //collect是当前其所在组件的实例的引用，可以直接在父组件中使用子组件的方法，子组件不需要设置
    template:`
        <contact-collect (click)="collect.collectTheContact()" #collect></contact-collect>
    `
})
//...
//子组件
export class ContactCollectComponent{
    detail:any={};
    collectTheContact(){
        this.detail.collection==0?this.detail.collection=1:this.detail.collection=0;
    }
}

//通过@ViewChild在父组件类中直接使用组件类的数据
/**
 * @ViewChild是装饰器，需要引入
 * 1、参数为类，那么父组件将可以绑定一个指令或者子组件实例
 * 2、参数为字符串，那么相当于在父组件中绑定一个模板局部变量
 */

import {Component,AfterViewInit,ViewChild} from '@anuglar/core';
import {ContactCollecComponent} from './ContactCollecComponent.ts'
//...
export class CollectionComponent{
    @ViewChild(ContactCollecComponent) contactCollect:ContactCollecComponent;
    ngAfterViewInit(){
        //...
    }
    //便可以直接使用子组件的方法
    collectTheContact(){
        this.contactCollect.collectTheContact();
    }
}

/**
 * 综上，父组件向子组件交互数据的基本方式有5种：使用@input,@output属性修饰，set与get设置拦截与读取,ngOnChanges监听数据变化，模板局部变量,@ViewChild修饰器
 */

```

### 五、组件内容嵌入

```
//组件内容嵌入能方便代码的复用
//内容嵌入用来创建可复用的组件，如模态对话框与导航栏
//需要使用ng-content

//产生一个动态组件
import {Component} from '@angular/core';

@Component({
    selector:'example-content',
    //ng-content中的select属性能够指定哪些dom元素被加进来,这里是指第一个header元素
    template:`
        <div>
            <h4>示例</h4>
            <div style="border:1px solid red;">
                <ng-content select="header"></ng-content>
            </div>
        </div>
    `
})

export class ExampleContentComponent{}

//在一个组件中使用这个动态组件
//被example-content包裹的dom元素会被结合放到ExampleContentComponent组件的dom结构，最终形成最终的dom树
@Component({
    selector:'container',
    template:`
        <example-content>
            <header >header content</header>
        </example-content>
    `
})
export class ContainerComponent{}

//ng-content的select属性的使用类似css选择器
//如,select=".class-select",select="[name=footer]"

```

### 六、组件的生命周期

```
//组件生命的各个阶段都有相应的钩子函数可调用，从创建，渲染，数据变动事件触发，再到组件被移除

//生命周期钩子的使用：
/**
 * 钩子的接口定义在@angular/core中，然后再组件类中使用。使用时一般命名:ng+接口名
 */
/**
 * 组件常用的生命周期钩子方法，按调用顺序排列
 * ngOnChanges                                 //组件输入值变化触发
 * ngOnInit                                    //第一次ngOnChanges之后调用，可用于数据绑定输入属性之后初始化组件，经常在组件中使用ngOnInit获取数据
 * ngDoCheck                                   //每次变化监测时都会调用（每一个变化监测周期内）,监测的粒度比ngOnChanges小，用于特殊情况
 * ngAfterContentInit                          //ng-content将外部内容嵌入到组件视图后调用，只执行一次
 * ngAfterContentChecked                       //组件ng-content自定义内容，外部内容嵌入到组件视图后，或者每次变化检测的时候调用
 * ngAfterViewInit                             //angular创建了组件的视图或者在其子视图之后被调用
 * ngAfterViewChecked                           //创建了组件的视图或者在其子视图之后被调用一次，然后子组件变化监测时会被调用
 * ngOnDestroy                                  //销毁组件/指令之前触发。那些不会被自动回收的资源如:已订阅的观察者事件，绑定过的DOM事件，设置的定时器。都需要在ngOnDestroy手动销毁掉
 * 有些组件还提供自己特有的钩子，如路由自己的钩子
 */
```