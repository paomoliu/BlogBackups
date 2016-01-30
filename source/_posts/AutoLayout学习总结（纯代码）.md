title: AutoLayout学习总结（纯代码）
date: 2016-01-10 17:38:12
tags: [AutoLayout, VFL]

---

从15年3月正式准备做ios开发，到五月底，团队完成属于我们的第一个App，一直没有使用`AutoLayout`进行布局，当时觉得这个东西好复杂、好难，再加上项目赶得急，就把它丢在一边。在15年接下来的几个月里，没怎么撸码，一直停留在书本上。偶尔遇到一点`AutoLayout`，会去搜点博客，了解一丢丢`AutoLayout`。后来发现公司里好多都用`AutoLayout`，再加上最近在研究[开源中国](http://git.oschina.net/oschina/iphone-app)的源代码，它使用`AutoLayout`纯代码进行布局（主要是`VFL`），所以决定在出去找工作前，把欠的`AutoLayout`补起来。

<!--more-->

# AutoLayout是什么？
引用[喵神](http://onevcat.com/)的[WWDC 2012 Session笔记——202, 228, 232 AutoLayout（自动布局）入门](http://onevcat.com/2012/09/autoayout/)中的一段话：
> 使用一句Apple的官方定义的话
AutoLayout是一种基于约束的，描述性的布局系统。 Auto Layout Is a Constraint-Based, Descriptive Layout System.
关键词：

>  - 基于约束 － 和以往定义frame的位置和尺寸不同，AutoLayout的位置确定是以所谓相对位置的约束来定义的，比如x坐标为superView的中心，y坐标为屏幕底部上方10像素等
 - 描述性 － 约束的定义和各个view的关系使用接近自然语言或者可视化语言（稍后会提到）的方法来进行描述
 - 布局系统 － 即字面意思，用来负责界面的各个元素的位置。 
 
 在`AutoLayout`中，当描述完视图对象之间的约束后，`AutoLayout`会自动计算出视图对象的位置和大小，间接的设定视图的`Frame`。
 
# 创建约束

使用纯代码进行自动布局，首先必须关闭`View`的`AutoresizingMask`
```objectivec
view.translatesAutoresizingMaskIntoConstraints = NO;
```
该属性默认为`YES`，引用官方的一段话：
> If this property's value is YES, the system creates a set of constraints that duplicate the behavior specified by the view's autoresizing mask. This also lets you modify the view's size and location using the view's frame, bounds, or center properties, allowing you to create a static, frame-based layout within Auto Layout. 
Note that the autoresizing mask constraints fully specify the view’s size and position; therefore, you cannot add additional constraints to modify this size or position without introducing conflicts. If you want to use Auto Layout to dynamically calculate the size and position of your view, you must set this property to NO, and then provide a non ambiguous, nonconflicting set of constraints for the view. 

也就是说，当该属性值为`YES`时，运行时系统会自动将`AutoresizingMask`转换为`AutoLayout`的约束，这些约束很有可能会和我们自己添加的约束产生冲突，所以在使用`AutoLayout`进行纯代码布局时，必须设置该属性为`NO`。

## 创建约束的两个类方法：

`AutoLayout`中约束对应的类是`NSLayoutConstraint`，一个`NSLayoutConstraint`实例代表一条约束。`NSLayoutConstraint`有两个创建约束的类方法:
### 1. 第一个类方法：  
```objectivec
+ (instancetype)constraintWithItem:(id)view1                     
                         attribute:(NSLayoutAttribute)attr1
                         relatedBy:(NSLayoutRelation)relation
                            toItem:(id)view2
                         attribute:(NSLayoutAttribute)attr2
                        multiplier:(CGFloat)multiplier
                          constant:(CGFloat)c
```
该类方法对应的公式：
```objectivec
view1.attr1 = multiplsier × view2.attr2 + c
```
`view1`：约束左边的视图。
`attr1`：`view1`的属性。
`relation`：约束左边的视图与约束右边的视图的关系，上述公式中`＝`就是其中一种关系，还可以是`≥`或`≤`。
`view2`：约束右边的视图。
`attr2`：`view2`的属性。
`multiplier`：乘数。
`c`：常量。

**其中NSLayoutAttribute属性:**
```objectivec
typedef enum: NSInteger {
   NSLayoutAttributeLeft = 1,                                   //左边
   NSLayoutAttributeRight,                                      //右边
   NSLayoutAttributeTop,                                        //顶部
   NSLayoutAttributeBottom,                                     //底部
   NSLayoutAttributeLeading,                                    //首部
   NSLayoutAttributeTrailing,                                   //尾部
   NSLayoutAttributeWidth,                                      //宽
   NSLayoutAttributeHeight,                                     //高
   NSLayoutAttributeCenterX,                                    //x轴中心
   NSLayoutAttributeCenterY,                                    //y轴中心
   NSLayoutAttributeBaseline,                                   //基线
   NSLayoutAttributeLastBaseline = NSLayoutAttributeBaseline,   //基线，有的对象有超过一行的文本行，该基线是文本行最底的基线
   NSLayoutAttributeFirstBaseline,                              //基线，有的对象有超过一行的文本行，该基线是文本行最顶的基线
   
   NSLayoutAttributeLeftMargin,                                 //左边间距
   NSLayoutAttributeRightMargin,                                //右边间距
   NSLayoutAttributeTopMargin,                                  //顶部间距
   NSLayoutAttributeBottomMargin,                               //底部间距
   NSLayoutAttributeLeadingMargin,                              //首部间距
   NSLayoutAttributeTrailingMargin,                             //尾部间距
   NSLayoutAttributeCenterXWithinMargins,                       //The center along the x-axis between the object’s left and right margin.
   NSLayoutAttributeCenterYWithinMargins,                       //The center along the y-axis between the object’s top and bottom margin.
   
   NSLayoutAttributeNotAnAttribute = 0                          
} NSLayoutAttribute;
```
`NSLayoutAttributeNotAnAttribute`：官方文档的解释：
> A placeholder value that is used to indicate that the constraint’s second item and second attribute are not used in any calculations. Use this value when creating a constraint that assigns a constant to an attribute. For example, item1.height >= 40. If a constraint only has one item, set the second item to nil, and set the second attribute to NSLayoutAttributeNotAnAttribute. 

注意：`NSLayoutAttributeLeft/NSLayoutAttributeRight`和 `NSLayoutAttributeLeading/NSLayoutAttributeTrailing`的区别是`Left/Right`永远是指左右，而`Leading/Trailing`在某些从右至左习惯的地区会变成，`Leading`是右边，`Trailing`是左边。由于国际化的关系，Apple推荐使用`Leading`和`Trailing`代替`Left`和`Right`。

**NSLayoutRelation关系：**
```objectivec
enum {
   NSLayoutRelationLessThanOrEqual = -1,        //小于等于
   NSLayoutRelationEqual = 0,                   //等于
   NSLayoutRelationGreaterThanOrEqual = 1,      //大于等于
};
typedef NSInteger NSLayoutRelation;
```
### 2. 第二个类方法：

```objectivec
+ (NSArray<__kindof NSLayoutConstraint *> *)constraintsWithVisualFormat:(NSString *)format 
                                                                options:(NSLayoutFormatOptions)opts
                                                                metrics:(NSDictionary<NSSdtring *, id> *)metrics
                                                                  views:(NSDictionary<NSSdtring *, id> *)views
```

`format`：`VFL`字符串。
`opts`：描述在`VFL`中所有对象的属性和布局方向，默认是`0`。
`metrics`：在`VFL`中出现的常量字典，字典的`keys`必须是在`VFL`中使用的字符串值，他们的值必须是`NSNumber`对象，编译器在解析`VFL`字符串时，会自动将键替换为其对应的值。
`views`：在`VFL`中出现的视图字典，字典的`keys`必须是在`VFL`中使用的字符串值，他们的值必须是`UIView`对象。

**其中NSLayoutFormatOptions枚举：**
```objectivec
enum {
   /* choose only one of these */
   NSLayoutFormatAlignAllLeft = NSLayoutAttributeLeft,
   NSLayoutFormatAlignAllRight = NSLayoutAttributeRight,
   NSLayoutFormatAlignAllTop = NSLayoutAttributeTop,
   NSLayoutFormatAlignAllBottom = NSLayoutAttributeBottom,
   NSLayoutFormatAlignAllLeading = NSLayoutAttributeLeading,
   NSLayoutFormatAlignAllTrailing = NSLayoutAttributeTrailing,
   NSLayoutFormatAlignAllCenterX = NSLayoutAttributeCenterX,
   NSLayoutFormatAlignAllCenterY = NSLayoutAttributeCenterY,
   NSLayoutFormatAlignAllBaseline = NSLayoutAttributeBaseline,
   
   NSLayoutFormatAlignmentMask = 0xFF,
   
   /* choose only one of these three */
   NSLayoutFormatDirectionLeadingToTrailing = 0 << 8, // default 
   NSLayoutFormatDirectionLeftToRight = 1 << 8,
   NSLayoutFormatDirectionRightToLeft = 2 << 8,
   
   NSLayoutFormatDirectionMask = 0x3 << 8,
};
typedef NSUInteger NSLayoutFormatOptions;
```

#### **VFL（Visul Format Language）**
若按照第一种方法创建约束，多了就显得很繁琐，于是就有了`VFL`——可视化格式语言。
`V:`：表示纵向
`H:`：表示横向（默认）
`|` ：表示父视图
`[view]`：表示设置约束的视图
`-`：表示间距
`>=`：表示视图间距、宽度或高度必须大于或等于某个值
`==`：表示视图间距、宽度或高度必须等于某个值
`<=`：表示视图间距、宽度或高度必须小于或等于某个值
`@`：表示优先级

VFL示例：
> 1. `|-0-[view]-0-|`表示与父视图左、右间距为`0`，等价于`|[view]|`
2. `V:|-0-[view]-0-|`表示与父视图顶部、底部间距为`0`，等价于`|-[view]-|`、`|[view]|`
3. `|-[view]-|`表示水平方向，该字符串表示视图在父视图的左右边缘内，与父视图的左右间距不为`0`
4. `[view1]-[view2]`表示两视图间的间距为`8`（标准间距）
5. `[view(100)]`表示视图的宽度为`100`
6. `[view(100@750)]`表示视图宽度为`100`，该约束的优先级为`750`
7. `[view1][view2]`表示两视图水平方向紧邻（间距为`0`）
8. `[view(>=100,<=200)]`表示视图的宽度大于等于`100`且小于等于`200`
9. `|-[view1(view2)]-[view2]-|`表示两视图的宽度一样，并在父视图的左右边缘内
10. `V:|-30-[view(100)]`表示视图的高度为`100`，且与父视图顶部间距为`30`
11. `V:|-(==tpadding)-[view1]->=5-[view2]-(bpadding)|`表示两视图的间距必须大于等于`5`，`view1`距离父视图顶部`tpadding`，`view2`距离父视图底部`bpadding`
**[部分示例源代码](https://github.com/paomoliu/VFLDemo)**

`VFL`格式官方文档的介绍[Visul Format Language](https://developer.apple.com/library/watchos/documentation/UserExperience/Conceptual/AutolayoutPG/VisualFormatLanguage/VisualFormatLanguage.html)

## 优先级
各个约束的力量大小，是根据约束的`priority`属性的值决定，高优先级的约束会比低优先级约束优先得到满足。官方文档的描述：

> If a constraint's priority level is less than `UILayoutPriorityRequired`  in OS X or `UILayoutPriorityRequired` in iOS, then it is optional. Higher priority constraints are met before lower priority constraints.

该属性的取值必须**大于`0`小于等于`1000`**，它是`UILayoutPriority`枚举类型，系统内置了`4`个优先级：
```objectivec
 enum {
   UILayoutPriorityRequired = 1000,        //默认优先级
   UILayoutPriorityDefaultHigh = 750,
   UILayoutPriorityDefaultLow = 250,
   UILayoutPriorityFittingSizeLevel = 50,
};
typedef float UILayoutPriority;
```
`priority`属性的值越大，优先级越高，越会被优先满足。
每个约束的默认优先级是`UILayoutPriorityRequired`，这意味着我们给出的所有约束都必须得到满足，一旦约束间发生冲突，我们的应用就会**Crash**。

## 小结
一般开发中都是多种方式搭配使用，以最简洁的代码实现功能。在这段时间研究的[开源中国](http://git.oschina.net/oschina/iphone-app)的源代码中，一个源文件中对`UITableView`的布局就是使用`AutoresizingMask`，其他的`View`大部分是使用`VFL`，但由于**VFL并不能表达所有的约束（如：处理比例约束，处理相对居中效果）**，所以又使用了`AutoLayout`另一种创建约束的方式。

# 示例

# 参考
[学习AutoLayout(NSLayoutConstraint)](http://www.jianshu.com/p/ebb8570ad70f)
[学习AutoLayout(VFL)](http://www.jianshu.com/p/385070898e77)
[Autolayout的第一次亲密接触](http://www.jianshu.com/p/052e8c7e8e7f)
[AutoLayout深入浅出五[纯代码的偏执]](http://grayluo.github.io/WeiFocusIo/autolayout/2015/02/01/autolayout6/)
[iOS开发通过代码方式使用AutoLayout (NSLayoutConstraint + Masonry)](http://www.cocoachina.com/ios/20151029/13872.html)
[iOS 开发实践之 Auto Layout](http://xuexuefeng.com/autolayout/)


                                                    

 





