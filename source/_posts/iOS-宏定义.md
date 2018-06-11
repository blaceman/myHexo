---
title: iOS 宏定义那些事情
date: 2017-01-03 11:49:37
tags:
---

##### 一直以来用宏定义只用了些基本的东西,例如常量定义,最多也就是定义一个函数,很少关注宏定义高级的用法,今天就说一下摸索一下高级用法~

### 这篇文章主要说

- 宏定义的基本用法
- 高级用法
- 常用的宏定义


C中的宏分为两类，对象宏(object-like macro)和函数宏(function-like macro)。
### 宏定义的基本用法

##### 1.定义一个圆周率(对象宏)

```
//This defines PI
#define M_PI    3.14159265358979323846264338327950288
```
##### 2.定义一个平方函数(函数宏)

```
#define SQUARE(A) ((A) * (A))
```
其实这并不是正确的写法,正确的写法应该使用GUN C 扩展来避免参数带运算符所带来的错误等

```
#define SQUARE(A) ({ __typeof__(A) __a = (A); __a * __a;})
```


### 高级用法

##### 1.首先看看系统`MIN(A,B)`

```
#define __NSX_PASTE__(A,B) A##B
#if !defined(MIN)
    #define __NSMIN_IMPL__(A,B,L) ({ __typeof__(A) __NSX_PASTE__(__a,L) = (A); __typeof__(B) __NSX_PASTE__(__b,L) = (B); (__NSX_PASTE__(__a,L) < __NSX_PASTE__(__b,L)) ? __NSX_PASTE__(__a,L) : __NSX_PASTE__(__b,L); })
    #define MIN(A,B) __NSMIN_IMPL__(A,B,__COUNTER__)
#endif
```

这`MIN(A,B)`分为三部分,第一部分是`#define __NSX_PASTE__(A,B) A##B`,##的意思是将A和B连接起来的意思,而第二部分是带三个参数`#define __NSMIN_IMPL__(A,B,L)`参数A,B自然是输入参数,而L则是`__NSX_PASTE__(__a,L)`定义的连接`__a`和`L`的参数名,再看第三部分`#define MIN(A,B) __NSMIN_IMPL__(A,B,__COUNTER__)`,`__COUNTER__`是一个预定义的宏，这个值在编译过程中将从0开始计数，每次被调用时加1。明显这是保证构造独立的变量名称,大大避免了变量名相同而导致问题的可能性。<br>
我们把上诉自己定义的`SQUARE(A)`变成最接近系统那样的写法

```
#define __SQUARE_IMPL__(A,L) ({ __typeof__(A) __NSX_PASTE__(__a,L) = (A); __NSX_PASTE__(__a,L) * __NSX_PASTE__(__a,L);})
#define SQUARE(A) __square_IMPL__(A,__COUNTER__)
```

##### 2.使用 宏定义macros （#，##，...，__VA_ARGS_)

* 使用”#“预处理操作符来实现将宏中的参数转化为字符串

```
#define WARN_IF(EXP) \  
do { if (EXP) \  
        fprintf (stderr, "Warning: " #EXP "\n"); } \  
while (0)  
WARN_IF (x == 0);  
     ==> do { if (x == 0)  
           fprintf (stderr, "Warning: " "x == 0" "\n"); } while (0);
```

* 使用"##"操作符可以实现宏中token的连接。 刚才也用过

```
#define __NSX_PASTE__(A,B) A##B
```

* 多参数宏（Variadic Macros)

```
#define eprintf(...) fprintf (stderr, __VA_ARGS__)  
eprintf ("%s:%d: ", input_file, lineno)  
     ==>  fprintf (stderr, "%s:%d: ", input_file, lineno)
```
使用标识符__VA_ARGS_来表示多个参数，在宏的名称中则使用(...)

* "##"的特殊用法：

```
#define eprintf(format, ...) fprintf (stderr, format, ##__VA_ARGS__)  
eprintf ("success!\n")  
     ==> fprintf(stderr, "success!\n");
```
将"##"放在","和参数之间，那么如果参数留空的话，那么"##"前面的","就会删掉，从而防止编译错误。

* 例如:log 重定义

```
//Another wrong version of NSLog
#define NSLog(format, ...)   {
                               fprintf(stderr, "<%s : %d> %s\n",                                           \
                               [[[NSString stringWithUTF8String:__FILE__] lastPathComponent] UTF8String],  \
                               __LINE__, __func__);                                                        \
                               (NSLog)((format), ##__VA_ARGS__);                                           \
                               fprintf(stderr, "-------\n");                                               \
                             }
```

这里用到了三个预定义宏，和刚才的`__COUNTER__`类似，预定义宏的行为是由编译器指定的。`__FILE__`返回当前文件的绝对路径，`__LINE__`返回展开该宏时在文件中的行数，`__func__`是改宏所在scope的函数名称。我们在做Log输出时如果带上这这三个参数，便可以加快解读Log，迅速定位


### 常用的宏定义

##### 输出rect，size和point的宏
```
#define NSLogRect(rect) NSLog(@"%s x:%.4f, y:%.4f, w:%.4f, h:%.4f", #rect, rect.origin.x, rect.origin.y, rect.size.width, rect.size.height)
#define NSLogSize(size) NSLog(@"%s w:%.4f, h:%.4f", #size, size.width, size.height)
#define NSLogPoint(point) NSLog(@"%s x:%.4f, y:%.4f", #point, point.x, point.y)
```


##### 获取屏幕 宽度、高度

```
#define SCREEN_WIDTH ([UIScreen mainScreen].bounds.size.width)
#define SCREENH_HEIGHT ([UIScreen mainScreen].bounds.size.height)
```

##### 系统版本
```
#define IOS10_OR_LATER		( [[[UIDevice currentDevice] systemVersion] floatValue] >= 10.0 )
#define IOS9_OR_LATER		( [[[UIDevice currentDevice] systemVersion] floatValue] >= 9.0  )
#define IOS8_OR_LATER		( [[[UIDevice currentDevice] systemVersion] floatValue] >= 8.0 )
```
##### 尺寸

```
#define IS_SCREEN_55_INCH	([UIScreen instancesRespondToSelector:@selector(currentMode)] ? CGSizeEqualToSize(CGSizeMake(1242, 2208), [[UIScreen mainScreen] currentMode].size) : NO)
#define IS_SCREEN_47_INCH	([UIScreen instancesRespondToSelector:@selector(currentMode)] ? CGSizeEqualToSize(CGSizeMake(750, 1334), [[UIScreen mainScreen] currentMode].size) : NO)
#define IS_SCREEN_4_INCH	([UIScreen instancesRespondToSelector:@selector(currentMode)] ? CGSizeEqualToSize(CGSizeMake(640, 1136), [[UIScreen mainScreen] currentMode].size) : NO)
#define IS_SCREEN_35_INCH	([UIScreen instancesRespondToSelector:@selector(currentMode)] ? CGSizeEqualToSize(CGSizeMake(640, 960), [[UIScreen mainScreen] currentMode].size) : NO)
```


##### 颜色

```
//带有RGBA的颜色设置
#define COLOR(R, G, B, A) [UIColor colorWithRed:R/255.0 green:G/255.0 blue:B/255.0 alpha:A]
#define RandomColor [UIColor colorWithRed:arc4random_uniform(256)/255.0 green:arc4random_uniform(256)/255.0 blue:arc4random_uniform(256)/255.0 alpha:1.0]

```

##### 强弱引用

```
#define LRWeakSelf(type)  __weak typeof(type) weak##type = type;
#define LRStrongSelf(type)  __strong typeof(type) type = weak##type;
```

##### 圆角边框

```
#define LRViewBorderRadius(View, Radius, Width, Color)\
\
[View.layer setCornerRadius:(Radius)];\
[View.layer setMasksToBounds:YES];\
[View.layer setBorderWidth:(Width)];\
[View.layer setBorderColor:[Color CGColor]]
```

##### 模拟器真机

```
// 判断是不是iOS系统，如果是iOS系统在真机和模拟器输出都是YES
#if TARGET_OS_IPHONE  
#endif  
   
#if (TARGET_IPHONE_SIMULATOR)    
          // 在模拟器的情况下
#else
         // 在真机情况下
#endif
```
##### 沙河目录

```
//获取temp
#define kPathTemp NSTemporaryDirectory() 
   
//获取沙盒 Document
#define kPathDocument [NSSearchPathForDirectoriesInDomains(NSDocumentDirectory, NSUserDomainMask, YES) firstObject] 
   
//获取沙盒 Cache
#define kPathCache [NSSearchPathForDirectoriesInDomains(NSCachesDirectory, NSUserDomainMask, YES) firstObject]
```


主要参考链接:[宏定义的黑魔法](https://onevcat.com/2014/01/black-magic-in-macro/), [iOS开发 高级：使用 宏定义macros （#，##，...，__VA_ARGS_)](http://blog.csdn.net/songrotek/article/details/8929963)











