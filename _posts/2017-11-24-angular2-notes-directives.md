---
title: Angular2学习笔记-指令相关
date: 2017-11-24
categories:
- Angular2
tags: 
- Angular2
- Angular2学习笔记
---

### 一、指令相关

```
//指令的作用在于扩展特定DOM元素的功能。而组件可以当做指令一个类别
/**
 * 指令的三个类别：属性指令，结构指令，组件
 */

 //1、属性指令
 //属性指令一般用来改变元素的外观与行为。如NgStyle

 //2、结构指令
 //改变DOM结构  ,如果NgIf

 //3、组件被用来构造自定义行为的可重用视图。

//组件与其他两个类别的指令的不同点:
/**
 * a、组件用于自己的模板，以自定义标签的方式使用
 * b、一些生命周期的钩子函数为组件独有：例如：ng
 */
```

### 二、内置指令

```
//内置指令分为三个类别：通用指令，路由指令，表单指令

//1、通用指令
//包含在CommonModule模块
/**
 * NgClass
 * NgStyle
 * NgIf
 * NgSwitch,NgSwitchCase,NgSwitchDefault
 * NgFor
 * NgTemplateOutlet
 * NgPlural,NgPluralCase
 */


 //2、表单指令
 /**
  * 表单指令包含在三个模块中：
  FormsModule,ReactiveFormsModule,InternalFormsSharedModule
  */
//FormsModule
/**
 * NgModel,NgmodelGroup,NgForm,InternalFormsSharedModule
 */

 //ReactiveFormsModule
 /**
  * FormControlDirective,FormGroupDirective,FormControlName,FormGroupName,FormArrayName,InternalFormsModule
  FormControlDirective :将一个已有的FormControl实力绑定到DOM元素
  FormGroupDirective:将已有的表单组合绑定到一个Dom元素,通过formControl
  FormContrlName :将已有的表单控件与一个Dom元素绑定，可以为表单控件制定一个别名。利用这个别名可以做校验以及获取值等。
  其他几个不重要
  */
  //InternalFormsShareModule
  //FormsModule与ReactiveFormsModule均引自InternalFormsSharedModule
  /**
   * 包括表单元素访问器指令：如
   * DefaultValueAccessor,NumberValueAccessor等,这些访问器指令无需再应用中主动访问。他们负责DOM元素与表单输入控件的联系
   * 选择框选项指令：NgSelectOption,NgSelectMultipleOption。可以设置多选与单选框
   * 表单校验指令：RequiredValidator,MinLengthValidator,MaxLengthValidator,PatternValidator
   * 控件状态指令：NgControlStatus,NgControlStatusGroup。无需主动使用，会自动判别控件状态作如设置css类的处理。
   */


   //3、路由指令
   /**
    * routerLink,routerOutlet,routerLinkActive
     routerLink:占位符
      routerOutlet:组件插入的地方
      routerLinkActive:当前路由与css类的关系
    详细讲解见路由模块
    */

```

### 三、自定义属性指令

```
//1、如何创建一个指令
/**
 * a、首先需要有一个控制器类，该类使用@Directive装饰。该装饰制定选择器。控制类实现控制行为
 */
//例:
import {Directive,ElementRef} from '@angular/core';

@Directive({
    selector:'[yellowBg]' //表示使用了yellowBg属性的元素
})
export class YelloWBg{
    constructor(el:ElementRef){
            el.nativeElement.style.backgroundColor='yellow';
    }
}
//每一个匹配到的DOM元素都会创建一个实例，使用ElementRef的nativeElement属性可以直接访问DOM元素
//解析到属性-》创建指令类的实例->元素的引入传入到构造函数-》设置样式

//为指令绑定输入
//需要使用@Input装饰器
export class YellowBg{
    @Input()//不使用别名直接在元素中使用
    backgroundColor:string;
}
let test=`
    <div [yellowBg] [backgroundColor]="color"><>
`;

//响应用户操作
//通过@Hostlistener装饰器来修饰dom事件。是的dom事件与指令关联
@HostListener('click')
onClick{
    this.setStyle(this.backgroundColor||this._defaultColor);
}
```

### 四、自定义结构指令

```
//如何创建一个结构指令
/**
 * 引入@Directive装饰器
 * 添加css属性选择器，标识指令
 * 声明input绑定表达式
 * 添加TemplateRef(访问组件的模板)与ViewContainerRef(将模板内容插入到DOM中)可以用来渲染组件模板内容
 */

 import {Directive ,TemplateRef,与ViewContainerRef} from '@angular/core';
 @Directive({
     selector:['myUnless']
 })
 export class UnlessDirective{
     @Input('myUnless')
     set condition(newCondition:boolean){//set是属性拦截器
         if(!newCondition)this.ViewContainer.createEmbeddedView(this.templateRef);
         else this.viewContainer.clear()
     }

     constructor(){
         private templateRef:TmeplateRef<any>;
         private viewContainer:ViewContainerRef
     }
 }

 //扩展
 /**
  * 1、带星号前缀的指令会被替换成带有template标签的代码
  2、NgTemplateOutlet可以末班中创建内嵌视图
  3、NgPlural,NgPluralCase指令相当于NgSwitch，但是其支持范围匹配
  4、FormGroupName支持将已有表单组合到一个Dom元素
  5、FormArrayName当做FromGroupDirective指令使用。可以绑定表单数组
  */
```