title: NSAttributedString与NSMutableAttributedString学习总结
date: 2016-02-1 13:05:28
tags: [richtext]
---

在研究[开源中国](http://git.oschina.net/oschina/iphone-app)的源代码过程中，发现它里面的评论、点赞等按钮既有图又有文字，即所谓的**图文混排**，并不是我所想的`UIImageView＋UIButton`。里面还有一些属性字符串，支持链接、@等。这些都是利用[DTCoreText
](https://github.com/Cocoanetics/DTCoreText)和[TTTAttributedLabel
](https://github.com/TTTAttributedLabel/TTTAttributedLabel)两个第三方库实现。于是想自己了解一下富文本相关的知识，开始学习该部分内容，做个小总结。这篇总结主要是参考官方文档。


`NSAttributedString`可以对字符串附加格式信息，比如字体和段落对齐。可以在整个字符串上使用这些元数据，也可以对字符串的不同部分使用不同的属性。其字符串是**只读**的。我们将字符与属性的关联称为**属性字符串（attributed string）**。`NSMutableAttributedString`是`NSAttributedString`的子类，它是可修改的属性字符串。

<!--more-->

引用官方文档的定义：

> An `NSAttributedString` object manages character strings and associated sets of attributes (for example, font and kerning) that apply to individual characters or ranges of characters in the string. An association of characters and their attributes is called an attributed string. The cluster’s two public classes, `NSAttributedString` and `NSMutableAttributedString`, declare the programmatic interface for read-only attributed strings and modifiable attributed strings, respectively. 
> 
> `NSMutableAttributedString` declares the programmatic interface to objects that manage mutable attributed strings. You can add and remove characters (raw strings) and attributes separately or together as attributed strings. See the class description for `NSAttributedString` for more information about attributed strings. 

# 创建属性字符串
我们可以使用好几种方法创建属性字符串，主要记录三种：
## **第一种方法：**
```objectivec
- (instancetype)initWithString:(NSString *)aString;
```
该方法返回一个由`aString`初始化且无属性信息的`NSAttributedString`对象。

**示例：**
```objectivec
NSString *labelString = @"2016，Happy New Year!";
NSMutableAttributedString *attributedString = [[NSMutableAttributedString alloc] initWithString:labelString];
    
//设置富文本样式
[attributedString addAttribute:NSFontAttributeName                  //文本字体
                         value:[UIFont boldSystemFontOfSize:25.f]
                         range:NSMakeRange(0, 4)];
    
[attributedString addAttribute:NSForegroundColorAttributeName       //文本前景色
                         value:[UIColor redColor]
                         range:NSMakeRange(0, 4)];
    
UILabel *label = [[UILabel alloc] initWithFrame:CGRectMake(0, 100, 320, 50)];
label.attributedText = attributedString;
[self.view addSubview:label];
```
## **第二种方法：**
```objectivec
- (instancetype)initWithAttributedString:(NSAttributedString *)attributedString;
```
该方法返回一个由属性字符串`attributedString`初始化的`NSAttributedString`对象。

**示例：**
```objectivec
NSString *labelString = @"2016，Happy New Year!";
NSMutableAttributedString *attributedString = [[NSMutableAttributedString alloc] initWithString:labelString];

//设置富文本样式
NSDictionary *attrubutes = @{NSFontAttributeName            : [UIFont boldSystemFontOfSize:25.f],
                             NSForegroundColorAttributeName : [UIColor redColor]
                             };
    
NSRange range = NSMakeRange(0, 4);                          //富文本设置的范围
    
[attributedString setAttributes:attrubutes range:range];
    
//第一个label的富文本设置
UILabel *label = [[UILabel alloc] initWithFrame:CGRectMake(0, 100, 320, 50)];
label.attributedText = attributedString;
[self.view addSubview:label];
    
NSMutableAttributedString *attributedString2 = [[NSMutableAttributedString alloc] initWithAttributedString:attributedString];
    
//第二个label的富文本设置
UILabel *label2 = [[UILabel alloc] initWithFrame:CGRectMake(0, 200, 320, 50)];
label2.attributedText = attributedString2;
[self.view addSubview:label2];
```
## **第三种方法：**
```objectivec
- (instancetype)initWithString:(NSString *)aString
                    attributes:(NSDictionary<NSString *,
                                        id> *)attributes
```
该方法返回一个由字符串`aString`和属性`attributes`初始化的`NSAttributedString`对象。

**示例：**
```objectivec
NSString *labelString = @"2016，Happy New Year!";
//设置富文本样式
NSDictionary *attrubutes = @{NSFontAttributeName            : [UIFont boldSystemFontOfSize:25.f],
                             NSForegroundColorAttributeName : [UIColor redColor]
                             };
    
NSMutableAttributedString *attributedString = [[NSMutableAttributedString alloc] initWithString:labelString attributes:attrubutes];
    
UILabel *label = [[UILabel alloc] initWithFrame:CGRectMake(0, 100, 320, 50)];
label.attributedText = attributedString;
[self.view addSubview:label];
```
其余几种参看官方文档[Creating Attributed Strings in Cocoa](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/AttributedStrings/Tasks/CreatingAttributedStrings.html#//apple_ref/doc/uid/20000714-BBCCGGCC)。

# 字符属性（Character Attributes）
字符属性可以应用于属性字符串的text中。
以下是官方文档给出的字符属性内容：
```objectivec
NSString *const NSFontAttributeName;                    //设置文本字体
NSString *const NSParagraphStyleAttributeName;          //设置段落样式
NSString *const NSForegroundColorAttributeName;         //设置文本前景色（文本填充色）
NSString *const NSBackgroundColorAttributeName;         //设置文本背景色
NSString *const NSLigatureAttributeName;                //设置连字符
NSString *const NSKernAttributeName;                    //设置字符间距
NSString *const NSStrikethroughStyleAttributeName;      //设置删除线
NSString *const NSUnderlineStyleAttributeName;          //设置下划线
NSString *const NSStrokeColorAttributeName;             //设置画笔颜色
NSString *const NSStrokeWidthAttributeName;             //设置画笔宽度
NSString *const NSShadowAttributeName;                  //设置阴影
NSString *const NSTextEffectAttributeName;              //设置文本特效
NSString *const NSAttachmentAttributeName;              //设置文本附件，常用于图文混排
NSString *const NSLinkAttributeName;                    //设置链接，点击后打开指定URL地址
NSString *const NSBaselineOffsetAttributeName;          //设置基线偏移
NSString *const NSUnderlineColorAttributeName;          //设置下划线颜色
NSString *const NSStrikethroughColorAttributeName;      //设置删除线颜色
NSString *const NSObliquenessAttributeName;             //设置文本字形倾斜度，正值右倾，负值左倾
NSString *const NSExpansionAttributeName;               //设置文本横向拉伸属性，正值横向拉伸文本，负值横向压缩文本
NSString *const NSWritingDirectionAttributeName;        //设置文本书写方向，从左至右或从右至左
NSString *const NSVerticalGlyphFormAttributeName;       //设置文本排版方向，0横排文本，1竖排文本，在iOS中总是0，0以外的未定义
```
具体的说明查官方文档[Character Attributes](https://developer.apple.com/library/ios/documentation/UIKit/Reference/NSAttributedString_UIKit_Additions/index.html#//apple_ref/doc/constant_group/Character_Attributes)

注：`NSStrokeColorAttributeName`和`NSStrokeWidthAttributeName`结合使用，可使文本空心（此时前景色设置无效）。

对字符属性的部分属性做了编码，其代码与创建属性字符串在同一个项目文件，源代码地址：[AttributedStringDemo](https://github.com/paomoliu/AttributedStringDemo)

# 参考
[NSAttributedString Class Reference](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSAttributedString_Class/index.html#//apple_ref/occ/cl/NSAttributedString)
[NSMutableAttributedString Class Reference](https://developer.apple.com/library/ios/documentation/Cocoa/Reference/Foundation/Classes/NSMutableAttributedString_Class/index.html#//apple_ref/occ/cl/NSMutableAttributedString)
[NSAttributedString UIKit Additions Reference](https://developer.apple.com/library/ios/documentation/UIKit/Reference/NSAttributedString_UIKit_Additions/index.html#//apple_ref/doc/uid/TP40011688)
[Attributed String Programming Guide](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/AttributedStrings/AttributedStrings.html#//apple_ref/doc/uid/10000036-BBCCGDBG)



