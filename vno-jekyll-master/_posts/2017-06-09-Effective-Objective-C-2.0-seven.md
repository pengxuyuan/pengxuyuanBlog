---
layout: post
title: Effective Objective-C 2.0 总结（七）
date: 2017-06-09
---


## 系统框架

### 第 47 条：熟悉系统框架

1. 框架：将一系列代码封装成动态库，并在其中放入描述其接口的头文件。平时我们第三方框架用的是静态库，因为iOS 应用程序不允许其中包含动态库。
2. Foundation、CoreFoundation 框架平时用的比较多，“无缝桥接” 可以将这两种框架的对象平滑转换。
3. 还有很多系统库我们平时都应该尽量去用它们，而不是重新实现它们已经实现的功能：
   * CFNetwork
   * CoreAudio
   * AVFoundation
   * CoreData
   * CoreText



> * 许多系统框架都可以直接使用。其中最重要的是Foundation 与CoreFoundation，这两个框架提供了构建应用程序所需的许多核心功能。
> * 很多常见任务都能用框架来做，例如音频与视频处理、网络通信、数据管理等。
> * 请记住：用纯C 写成的框架与用Objective-C 写成的一样重要，若想成为优秀的Objective-C 开发者，应该掌握C 语音的核心概念。

### 第 48 条：多用块枚举，少用for 循环

**for 循环**

简单粗暴，遍历数组还可以，但是对于遍历字典或者set，就不太友好。  

**使用Objective-C 1.0 的NSEnumerator 来遍历**

```objective-c
	NSArray *array = @[@"A",@"B",@"C"];
	NSEnumerator *enumerator = [array objectEnumerator];
	NSString *string;
	while ((string = [enumerator nextObject]) != nil) {
		NSLog(@"%@",string);
	}
```

这种遍历使用相对比较统一，数组、字典和set 都可以这样子写，并且还有多种 “枚举器” 可供使用，例如反向遍历数组的枚举器。

**快速遍历**

```objective-c
    for (<#type *object#> in <#collection#>) {
        <#statements#>
    }
```

for in  这个更加简洁，如果某个类的对象支持快速遍历，那么就可以宣称自己遵从名为NSFastEnumeration 的协议，从而令开发者可以采用此语法来迭代改对象。此协议只定义了一个方法：

```objective-c
- (NSUInteger)countByEnumeratingWithState:(NSFastEnumerationState *)state objects:(id __unsafe_unretained _Nullable [_Nonnull])buffer count:(NSUInteger)len;
```

由于NSEnumerator 对象也实现了NSFastEnumeration 协议，所以能用来执行快速遍历。但是快速遍历拿不到当前操作对象的下标。

```objective-c
    NSArray *array = @[@"A",@"B",@"C"];
    for (NSString *string in [array reverseObjectEnumerator]) {
        NSLog(@"%@",string);
    }	
```

**基于块的遍历方式**

```objective-c
    NSArray *array = @[@"A",@"B",@"C"];
    [array enumerateObjectsUsingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        
    }];

    [array enumerateObjectsWithOptions:NSEnumerationReverse usingBlock:^(id  _Nonnull obj, NSUInteger idx, BOOL * _Nonnull stop) {
        
    }];
```

此方式对于其他相比，在遍历时候可以直接在块中获取更多信息，而且这种对于字典的遍历也是非常友好的，一次性可以返回键和值。并且还可以支持反向遍历。



> * 遍历collection 有四种方式。最基本的办法就是for 循环，其次是NSEnumerator 遍历法及快速遍历法，最新、最先进的方式则是 “块枚举法”。
> * “块枚举法” 本身就能通过GCD 来并发执行遍历操作，无须另行编写代码。而采用其他遍历则无法轻易实现这一点。
> * 若提前知道待遍历的collection 含有何种对象，则应修改块签名，指出对象的具体类型。



### 第 49 条：对自定义其内存管理语义的collection 使用无缝桥接

1. 使用 “无缝桥接” 计数，可以在定义于Foundation 框架中的Objective-C 类和定义于CoreFoundation 框架中的C 数据结构之间相互转换。

2. 三种转换方式

   * __bridge 只是声明类型转变，但是不做内存管理规则的转变
   * __bridge_retained 表示将指针类型转变的同时，将内存管理的责任由原来的Objective-C 交给Core Foundation 来处理，也就是ARC 转变成 MRC
   * __bridge_transfer 表示将管理的责任由Core Foundation 转交给Objective-C，即将MRC转变成ARC

3. 在Foundation 中字典对象：对其键的内存管理语义为 “拷贝”，而值的语义是 “保留”。只能通过强大的无缝桥接技术，否则无法改变其语义。

   CoreFoundation 框架的字典类型是CFDictionary，可变版本是CFMutableDictionary。

   ```objective-c
   //CFMutableDictionary 用CFDictionaryCreateMutable 来创建
   //用CFDictionaryCreateMutable 定义
   CFMutableDictionaryRef CFDictionaryCreateMutable (
     CFAllocatorRef allocator, 
     CFIndex capacity, 
     const CFDictionaryKeyCallBacks *keyCallBacks, 
     const CFDictionaryValueCallBacks *valueCallBacks
   );

   /*CFAllocatorRef 表示将要使用的内存分配器，CoreFoundation 对象里的数据结构需要占用内存，而分配器负责分配及回收这些内存，一般传NULL，表示采用默认的分配器。

   CFIndex 表示字典的初始大小，跟我们Foundation 字典的创建一样，并不限制最大容量 就是预先分配内存

   最后两个参数都是指向结构体的指针，定义了很多回调函数，用于指示字典中的键和值遇到各种事件时应该执行何种操作。

   CFDictionaryKeyCallBacks 的结构体定义
   typedef struct {
       CFIndex				version;
       CFDictionaryRetainCallBack		retain;
       CFDictionaryReleaseCallBack		release;
       CFDictionaryCopyDescriptionCallBack	copyDescription;
       CFDictionaryEqualCallBack		equal;
       CFDictionaryHashCallBack		hash;
   } CFDictionaryKeyCallBacks;

   CFDictionaryValueCallBacks 的结构体定义
   typedef struct {
       CFIndex				version;
       CFDictionaryRetainCallBack		retain;
       CFDictionaryReleaseCallBack		release;
       CFDictionaryCopyDescriptionCallBack	copyDescription;
       CFDictionaryEqualCallBack		equal;
   } CFDictionaryValueCallBacks;

   version 参数目前应设置为0，表示版本号；
   其他参数都是函数指针，例如，字典加入了新的键与值，那么就会调用retain 函数，定义如下：
   typedef const void *(*CFDictionaryRetainCallBack)(
   	CFAllocatorRef allocator, 
   	const void *value
   );
   retain 是个函数指针，其所指向的函数接受两个参数，其类型分别是CFAllocatorRef、const void *。传给此函数的value 参数表示即将加入字典中的键或值。而返回的void * 则表示加到字典里的最终值。我们可以这样子实现：
   const void *CustomCallback（CFAllocatorRef allocator，const void *value）{
   	return value;
   }
   如果用它充当retain 回调函数来创建字典，那么该字典就不会 “保留” 键和值。然后再利用无缝桥接搭配起来，就可以创建特殊的NSDictionary 对象，跟我们普通的字典不一样。

   开发者可以直接在CoreFoundation 层创建字典，于是就能修改内存管理语义，对键执行 “保留” 而非 “拷贝” 操作了。
   ```



> * 通过无缝桥接技术，可以在Foundation 框架中的Objective-C 对象与CoreFoundation 框架中的C 语言数据结构之间来回转换。
> * 在CoreFoundation 层面创建collection 时，可以指定许多回调函数，这些函数表示此collection 应如何处理其元素。然后，可运用无缝桥接技术，将其转换成具备特殊内存管理语义的Objective-C collection。

### 第 50 条：构建缓存时选用 NSCache 而非 NSDictionay

1. NSCache 是专门来处理缓存的，在系统资源将要耗尽时，它可以自动删减缓存。
2. NScahe 并不会 “拷贝” 键，而是会 “保留” 它。NScahe 键很多时候都是由不支持拷贝操作对象充当的。NSCache 是线程安全的，不用编写加锁代码，多个线程便可以同时访问NSCache。
3. NSPurgeableData 类是NSMutableData 的子类，而且实现了NSDiscardableContent 协议。如果对象所占的内存能够根据需要随时丢弃，那么久可以实现该协议所定义的接口。当系统资源紧张时，可以把保存NSPurgeableData 对象那块内存释放掉。



> * 实现缓存时应选用NSCache 而非NSDictionary 对象。因为NSCache 可以提供优雅的自动删减功能，而且是线程安全的，此外，它与字典不同，并不会拷贝健。
> * 可以给NSCache 对象设置上限，用以限制缓存中的对象总个数及总成本，而这些尺度则定义了缓存删减其中对象的时机。但是绝对不要把这些尺度当成可靠的 “硬限制”，它们仅对NSCache 其指导作用。
> * 将NSPurgeableData 与 NSCache 搭配使用，可实现自动清除数据的功能，也就是说，当NSPurgeableData 对象所占内存为系统丢弃时，该对象也会从缓存中移除。
> * 如果缓存使用得当，那么应用程序的响应速度就能提高。只有那种 “ 重新计算起来很费事的” 数据，才值得放入缓存，比如那些需要从网络获取或从磁盘读取的数据。

### 第 51 条：精简 initalize 与 load 的实现代码

1. 对于加入运行期系统中的每个类及分类，必定会调用load 这个方法，而且仅调用一次。意思就是程序启动的时候需要加载load 方法，这个时候运行期系统也是出于 “脆弱状态”，在执行子类的load 方法之前，必定会先执行所有超类的load 方法。
2. 如果load 代码还依赖了其他类，那类的load 也必然会先执行，我们无法判断每个类的载入顺序，所以load 方法使用其他类是不安全的。
3. load 方法不遵从继承规则，如果某个类没实现load 方法，那么不管其各级超类是否实现此方法，系统都不会调用。
4. load 方法要实现的精简点，因为应用程序在执行load 方法会阻塞。load 一般作为调试用，很少用来做初始化操作。
5. 想执行与类相关的初始化操作，可以使用 `+(void)initialize`  这个方法，它跟load 有以下几个区别：
   * 这个方法是在首次使用这个类的时候调用，类似 “惰性调用” ，只有用到这个类才会调用。
   * 运行期在执行该方法的时候，是出于正常状态的，此时是可以安全调用任意类的任意方法，而且运行期系统会确保initialize 方法一定在 “线程安全的环境” 中执行。其他线程都要先阻塞，等initialize 执行完。
   * initialize 方法跟其他方法一样，某个类没有实现它，而超类方法实现了，那么就会运行超类的实现代码。
6. 也就是说initalize 与 load 的实现代码要精简些。
7. 若某个全局状态无法在编译期初始化，则可以放在initalize 里来做。（例如Objectice-C 对象，创建实例之前必须先激活运行期系统）





> * 在加载阶段，如果类实现了load 方法，那么系统就会调用它。分类里也可以定义此方法，类的load 方法要比分类中的先调用。与其他方法不同，load 方法不参与覆写机制。
> * 首次使用某个类之前，系统会向其发送initialize 消息。由于此方法遵从普通的覆写规则，所以通常应该在里面判断当前要初始化的哪个类。
> * load 与initialize 方法都应该实现得精简一些，这有助于保持应用程序的响应能力，也能减少引入 “依赖环” 的几率。
> * 无法在编译器设定的全局变量，可以放在initialize 方法里初始化。

### 第 52 条：别忘了NSTimer 会保留其目标对象

1. 计时器放在运行循环里，它才能正常触发任务。

2. ```objective-c
   + scheduledTimerWithTimeInterval:target:selector:userInfo:repeats:
   ```

   计时器会保留其目标对象，等到自身 “失效” 时再释放此对象，调用invalidate 方法可令计时器失效；设置成重复执行模式的计时器，要注意 “保留环” 问题。

3. 如何解决外界不调用invalidate 方法也不产生 “保留环” 的问题。

   可以用块来解决这个问题，其实就是将timer 的target 对象不要指向持有timer 的对象，这里用的方法是让timer 的taerget 指向自己。

   ```objective-c
   //定义
   + (NSTimer *)my_scheduledTimerWithTimeInterval:(NSTimeInterval)ti
                                            block:(void(^)())block
                                          repeats:(BOOL)yesOrNo;

   //实现
   + (NSTimer *)my_scheduledTimerWithTimeInterval:(NSTimeInterval)ti
                                            block:(void(^)())block
                                          repeats:(BOOL)yesOrNo {
       return [self scheduledTimerWithTimeInterval:ti
                                            target:self
                                          selector:@selector(my_blockInvoke:)
                                          userInfo:[block copy]
                                           repeats:yesOrNo];
   }

   - (void)my_blockInvoke:(NSTimer *)timer {
       void (^block) () = timer.userInfo;
       if (block) {
           block();
       }
   }
   ```





> * NSTimer 对象会保留其目标，直到计时器本身失效为止，调用invalidate 方法可令计时器实效，另外，一次性的计时器在触发完任务之后也会失效。
> * 反复执行任务的计时器，很容易引入保留环，如果这种计时器的目标对象又保留了计时器本身，那肯定会导致保留环。这种环状保留关系，可能是直接发生的，也可能是通过对象图里的其他对象间接发生的。
> * 可以扩充NSTimer 的功能，用 “块” 来打破保留环。不过，除非NSTimer 将来在公共接口里提供此功能，否则必须创建分类，将相关实现代码加入其中。