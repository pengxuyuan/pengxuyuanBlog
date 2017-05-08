---
layout: post
title: Effective Objective-C 2.0 总结（二）
date: 2017-05-08
---


## 对象、消息、运行期

### 第 6 条：理解 “属性” 这一概念

1. “对象”（object）就是 “基本构造单元”（building block），开发者可以通过对象来存储并传递数据，在对象直接传递数据并执行任务的过程就叫做 “消息传递”（Messaging）。

2. 如果对象布局在编译器就固定了，访问变量时，编译器会使用 “偏移量”（offset）来计算，这个偏移量是 “硬编码”（hardcode），表示该变量距离存放对象的内存区域的起始地址有多远。 存在一个问题：如果代码使用了编译期计算出来的偏移量，那么修改类定义之后必须重新编译，否则就会出错。

   Objective-C 处理方式是：把实例变量当作一种存储偏移量所用的 “特殊变量”（speacial variable），交由 “类对象”（class object）保管。偏移量会在运行期查找，这样子总能找到正确的偏移量，这就是稳固的 “应用程序二进制接口”（Application Binary Interface，ABI）。

3. 使用 “点语法” 的效果与直接调用存取方法相同，没有丝毫差别。

4. 属性有很多优势：

   * 可以使用 “点语法”

   * 编译器会自动编写访问这些属性所需的方法，此过程就做 “自动合成”（autosynthesis）

   * 编译器还会自动向类添加适当类型的实例变量，并且在属性名前面加下划线，以此作为实例变量的名字

   * 在实现代码中可以通过@synthesize 语法来指定实例变量的名字

     ```objective-c
     @implementation EOCPerson
     @synthesize firstName = _myFirstName;
     @synthesize lastName = _myLastName;
     @end
     ```

   * @dynamic 关键字会告诉编译器：不要自动创建实现属性所用的实例变量，也不要为其创建存取方法

**属性特质**

1. 属性可以拥有的特质分类四类：原子性、读/写权限、内存管理语义、方法名

2. 原子性（atomicity）

   属性默认情况下编译器所合成的方法会通过锁定机制确保其原子性，用nonatomic 特质，就不使用同步锁。

   iOS 使用同步锁的开销较大，这会带来性能问题，一般情况下并不要求属性必须是 “原子的”，因为 “原子性” 并不能保证 “线程安全”（thread safety），若要实现 “线程安全” 的操作，还需采用更加深层的锁定机制才行。

3. 读／写权限

   * 具备readwrite（读写）特质的属性拥有 “获取方法”（getter）与 “设置方法”（setter）。若该属性由@synthesize 实现，则编译器会自动生成这两个方法。
   * 具备readonly（只读）特质的属性仅拥有获取方法，只有该属性由@synthesize 实现，编译器才会为其合成获取方法。

4. 内存管理语义

   * assign
   * strong
   * weak
   * unsafe_unretained
   * copy

5. 方法名

   可以指定存取的方法名。

   ```objective-c
   @property (nonatomic,getter = isOn) BOOL on;
   ```





>* 可以用@property 语法来定义对象中所封装的数据。
>* 通过 “特质” 来指定存储数据所需的正确语义
>* 在设置属性所对应的实例变量时，一定要遵从该属性所声明的语义。
>* 开发iOS 程序时应该使用nonatomic 属性，因为atomic 属性会严重影响性能。

### 第 7 条：在对象内部尽量直接访问实例变量

1. 强烈建议：读取实例变量的时候采用直接访问的形式，而在设置实例变量的时候通过属性来做。

2. 关于直接访问跟通过属性访问的区别：

   * 由于不经过Objective-C 的 “方法派发” 步骤，所以直接访问实例变量的速度当然比较快。在这种情况下，编译器所生成的代码会直接访问保存对象实例变量的那块内存。
   * 直接访问实例变量时，不会调用其 ”设置方法“，这就绕过了为相关属性所定义的 ”内存管理语义“。比方说，如果在ARC 下直接访问一个声明为copy 的属性，那么并不会拷贝改属性，只会保留新值并释放旧值。
   * 如果直接访问实例变量，那么不会触发 ”键值观测“（Key-Value Observing，KVO）通知。
   * 通过属性来访问有助于排查与之相关的错误，因为可以给 ”获取方法“ 或 ”设置方法“ 中新增断点，进行调试。

3. 在初始化方法中总是应该直接访问实例变量，避免子类重写了设置方法（处理异常情况抛出异常）但是：如果待初始化的实例声明在超类中，而我们又无法在子类直接访问此实例变量的话，那么就需要调用 “设置方法” 了。

4. 在 “惰性初始化”（lazy initialization），必须通过 “获取方法” 来访问属性，不然实例变量永远不会初始化。

   ​

>* 在对象内部读取数据时，应该直接通过实例变量来读，而写入数据时，则应通过属性来写。
>* 在初始化方法及dealloc 方法中，总是应该直接通过实例变量来读写数据。
>* 有时会使用惰性初始化技术配置某份数据，在这种情况下，需要通过属性来读取数据。

### 第 8 条：理解 “对象等同性” 这一概念

1. 使用 == 操作符比较的两个指针的本身，而不是其所指的对象；所以这里有可能会出轨，得不到我们想要的结果。NSObject 提供 “isEqual” 方法，某些对象也提供了特殊的 “等同性判定方法”

   ```objective-c
   NSString *foo =@"Badger 123";
   NSString *bar = [NSString stringWithFormat:@"Badger %i",123];
   BOOL equalA = (foo == bar);	//equalA = NO
   BOOL equalB = [foo isEqual:bar];	//equalB = YES
   BOOL equalC = [foo isEqualToString:bar];	//equalC = YES
   ```

2. NSObject 协议中有两个用于判断等同性的关键方法：

   ```objective-c
   - (BOOL)isEqual:(id)object;
   - (NSUInteger)hash;
   ```

   如果 “isEqual” 方法判定两个对象相等，那么其hash 方法也必须返回同一个值。但是，如果两个对象的hash 方法返回同一个值，那么 “isEqual” 方法未必会认为两者相等。

   对于实现hash 方法需要一些技巧：

   ```objective-c
   - (NSUInteger)hash {
   	return 1337;
   }
   //这样子是可以的，但是会对collection 使用这个对象产生性能问题。因为在collection 在检索哈希表的时，会用对象的哈希码来做索引，在set 集合中，会根据哈希码把对象分装到不同的数组里面，在添加新对象的时候，要根据其哈希码找对与之对应的数组，依次检查其中各个元素，看数组已有的对象是否和将要添加的新对象相等，如果相等，就说明添加的对象已经在set 集合中了，是添加失败的。（如果所有对象的hash 值对一样，这样子set 集合只会有一个数组，所有数据都在一起了，每次插入数据都会遍历这个数组，这样子就会出现性能问题）

   - (NSUInteger)hash {
   	NSString *stringToHash = [NSString stringWithFormat@"%@:%@",_firstName,_lastNmae];
   	return [stringToHash hash];
   }
   //这样子能保证返回不同的哈希码，但是这里会存在创建字符串的开销，会比返回单一值要慢

   - (NSUInteger)hash {
       return [self.firstName hash] ^ [self.lastNmae hash];
   }
   //这样子可以保存较高的效率，又不会过于频繁的重复
   ```

3. 特定类所具有的等同性判定方法

   1. isEqualToString、isEqualToArray、isEqualToDictionary

   2. 如果需要经常判断等同性，可以自己创建等同性判断方法，这样子可以避免检测参数的类型，提升检测效率。

      ```objective-c
      - (BOOL)isEqualToPerson:(EOCPerson *)otherPerson {
      	if (self == object) return YES;
      	
      	if (![_firstName isEqualToString:otherPerson.face]) return NO;
      	if (![_lastName isEqualToString:otherPerson.head]) return NO;
      	return YES;
      }

      -(BOOL)isEqual:(id)object {
      	if ([self class] == [object class]){
      		return [self isEqualToPerson:(EOCPerson *)object];
      	}else{
      		return [super isEqual:object];
      	}
      }
      ```

4. 等同性判定的执行深度

   在我们只需要通过判断一个标识符就可以判断对象相等的时候，我们重写方法可以很方便的达到目的，比如判断一个idectifier 就能确定这两个对象相等，就不用判断那么多属性了。

5. 容器中可变类的等同性 

   把某个对象放入colloection 之后，不应该再去改变其哈希码了，不然会出现问题，在set 集合会导致改变之后对象存在在一个在原则上 “错误” 的位置。

   ​

>* 若想检测对象的等同性，请提供 “isEqual：” 与 hash 方法。
>* 相同的对象必须具有相同的哈希码，但是两个哈希码相同的对象却未必相同。
>* 不要盲目地逐个检测每条属性，而是应该依照具体需求来指定检测方案。
>* 编写hash 方法时，应该使用计算速度快而且哈希码碰撞几率低的算法。

### 第 9 条：以 ”类族模式“ 隐藏实现细节

1. “类族” 是一种很有用的模式，可以隐藏 “抽象基类” （abstract base class）背后的实现细节。
2. 用户无须自己创建子类实例，只需要调用基类方法来创建即可。

**创建类族**

1. 每个 “实体子类” 都从基类继承而来，“工厂模式” 是创建类族的办法之一，调用基类方法返回子类实例。

2. 如果对象所属的类位于某个类族中，那么查询其类型信息要注意，你可能觉得自己创建了某个类的实例，然后实际上创建的却是其子类的实例。

   ```objective-c
   -(BOOL) isKindOfClass: classObj; 判断是否是这个类或者这个类的子类的实例
   -(BOOL) isMemberOfClass: classObj; 判断是否是这个类的实例
   ```

**Cocoa 里的类族**

1. 系统框架中有许多类族，大部分collection 类都是类族。

   * 在使用NSArray 的alloc 方法来获取实例时，该方法首先会分配一个属于某类的实例，此实例充当 “占位数组”（placeholder array）。该数组稍后会转换另一个类的实例，而那个类是NSArray 的实体子类。

2. 对于Cocoa 中NSArray 这样子的类族，新增子类需要遵循几条规则：

   * 子类应该继承自类族中的抽象基类。

     若要编写NSArray 类族的子类，则需要其继承自不可变数组的基类或可变数组的基类。

   * 子类应该定义自己的数据存储方式。

     编写NSArray 子类时，必须用一个实例变量来存放数组中的对象；NSArray 本身只是包在其他隐藏对象外面的壳，它仅仅定义了所有数组都需要的一些接口。

   * 子类应当覆写超类文档中指明需要覆写的方法

     在每个抽象基类中，都有一些子类必须覆写的方法，编码前需要看下文档。

     ​

>* 类族模式可以把实现细节隐藏在一套简单的公共接口后面。
>* 系统框架经常使用类族。
>* 从类族的公共抽象基类中继承子类时要当心，若有开发文档，则应首先阅读。

### 第 10 条：在既有类中使用关联对象存放自定义数据

1. 可以给类关联许多其他的对象，这些对象通过 “键” 来区分。

2. 储存对象值的时候，可以指明 ”存储策略“（storage policy），用以维护相应的 ”内存管理语义“，`objc_AssociationPolicy`  的枚举定义存储策略。

   | 关联类型                              | 等效的@property 属性  |
   | :-------------------------------- | :--------------- |
   | OBJC_ASSOCIATION_ASSIGN           | assign           |
   | OBJC_ASSOCIATION_RETAIN_NONATOMIC | nonatomic，retain |
   | OBJC_ASSOCIATION_COPY_NONATOMIC   | nonatomic，copy   |
   | OBJC_ASSOCIATION_RETAIN           | retain           |
   | OBJC_ASSOCIATION_COPY             | copy             |

3. 下列方法可以管理关联对象：

   ```objective-c
   void objc_setAssociatedObject(id object, const void *key, id value, objc_AssociationPolicy policy)
   此方法以给定的键和策略为某对象关联对象值
     
   id objc_getAssociatedObject(id object, const void *key)
   此方法根据给定的键从某对象中获取相应的对象值
    
   void objc_removeAssociatedObjects(id object)
   此方法移除指定对象的全部关联对象
     
   设置关联对象用的键是不透明指针（opaque pointer），其指向的数据结构不局限于某种特定类型的指针。
   设置关联对象值时，若想令两个键匹配到同一个值，则两者必须是完全相同的指针，所以在设置关联对象值时，通常使用静态全局变量做键。
   ```





>* 可以通过 “关联对象” 机制来把两个对象连起来。
>* 定义关联对象时可指定内存管理语义，用以模仿定义属性时所采用的 “拥有关系” 与 “非拥有关系”。
>* 只有在其他做法不可行时才应选用关联对象，因为这种做法通常会引入难于查找的bug。

### 第 11 条：理解 objc_msgSend 的作用

1. 调用对象方法，在Objective-C 中叫做 “传递消息”（pass a message），消息有 “名称”（name）或“选择子”（selector），可以接受参数，而且可能还有返回值。

2. objc_megSend 的原型： ``` void objc_msgSend(id self,SEL cmd,...)```  ，是一个 “参数个数可变的函数”，能够接受两个或两个以上的参数，第一个参数代表接收者，第二个参数代表选择子，后续参数就是参数。

3. objc_megSend 函数会依据接收者和选择子来调用适当的方法：
   * 在接收者所属的类搜寻其 “方法列表”
   * 找不到的话，就沿着继承体系继续向上查找
   * 最终还是找不到相符的方法就执行 “消息转发”

4. objc_msgSend 会将匹配结果缓存在 “快速映射表”（fast map）里面，每个类都有这样子的一块缓存，接下来还向该类发送一样的消息，那么执行起来就很快了。

5. 这里有些特殊情况，需要由Objective-C 运行环境的另外一些函数来处理：
   * objc_msgSend_stret ：如果待发送的消息要返回结构体，那么可以交由此函数处理。只有当CPU 寄存器能够容纳得下消息返回类型时，这个函数才能处理此消息。若是返回值无法容纳于CPU 寄存器（比如说返回的结构体太大了），那么就由另外一个函数执行派发。此时，那个函数会通过分配在栈上的某个变量来处理消息所返回的结构体。
   * objc_msgSend_fpret：如果消息返回的是浮点数，可以交由此函数处理。这个函数是为了处理x86 等架构CPU 中某些令人惊讶的奇怪状况。
   * objc_msgSendSuper：如果要给超类发消息，那么就交由此函数处理。

6. 每个类里都有一张函数表，选择子的名称则是表的 “键”，对应的值都是指向函数的指针。objc_msgSend 等函数就是通过这个函数表来寻找应该执行的方法并执行跳转的。

7. 如果某函数的最后一项操作是调用另外一个函数，那么就可以运用 “尾调用优化” 技术。编译器会生成跳转至另外一个函数所需的指令码，而且不会向调用栈推入新的 “栈帧”。

   ​



> * 消息由接收者、选择子及参数构成。给某对象 “发送消息” 也就是相当于在该对象上 “调用方法”。
> * 发给某对象的全部消息要由 “动态消息派发系统” 来处理，该系统会查出对应的方法，并执行其代码。

### 第 12 条：理解消息转发机制

1. 当对象接收到无法解读的消息后，就会启动 “消息转发”（message forwarding）机制，程序员可经由此过程告诉对象应该如何处理未知消息。

2. 消息转发分为两大阶段：

   * 第一阶段选征询接收者，所属的类，看其是否能动态添加方法，以处理当前这个 “未知的选择子“（unknown seletor），这叫做 ”动态方法解析“（dynamic method resolution）。

   * 第二阶段涉及 ”完整的消息转发机制“（full forwarding mechanism）。如果运行期系统已经把第一阶段执行完了，那么接收者自己就无法再以动态新增方法的手段来响应包含该选择子的消息了。这里的第二阶段又分为下面两小步：

     * 首先，请接收者看看有没其他对象能处理这条消息；若有，则运行期系统会把消息转给那个对象，于是消息转发过程结束，一切正常。


     * 若没有 ”备援的接收者“（replacement receiver），则启动完整的消息转发机制，运行期系统会把与消息有关的全部细节都封装到NSInvocation 对象中，再给接受者最后一次机会，令其设法解决当前还未处理的这条消息。

**动态方法解析**

对象在收到无法解读的消息后，首先将调用其所属类的下列类方法：

```objective-c
+ (BOOL)resolveClassMethod:(SEL)sel
+ (BOOL)resolveInstanceMethod:(SEL)sel

//表示这个类是否能新增一个方法来处理此选择子
```

**备援接收者**

当前接收者还有第二次机会处理未知的选择子，运行期系统会它：能不能把这条消息转发给其他接收者来处理：

```objective-c
- (id)forwardingTargetForSelector:(SEL)aSelector
```

我们无法操作经由这一步所转发的消息，若是想在发送给备援接收者之前先修改消息内容，那就得通过完整的消息转发机制。

**完整的消息转发**

将消息有关的信息全部丢到NSInvacation 对象中，把消息指派给目标对象

```objective-c
- (void)forwardInvocation:(NSInvocation *)anInvocation
```



>* 若对象无法响应某个选择子，则进入消息转发流程。
>* 通过运行期的动态方法解析功能，我们可以在需要用到的某个方法时再将其加入类中。
>* 对象可以把其无法解读的某些选择子转交给其他对象来处理。
>* 经过上述两步之后，如果还是没办法处理选择子，那就启动完整的消息转发机制。

### 第 13 条：用 “方法调配技术” 调试 “黑盒方法”

1. 不需要源代码，也不需要通过继承子类来覆写方法就能改变这个类本身的功能，新功能在本类的所有实例都生效，此方案称为 “方法调配”（method swizzling）。

2. 每个类有个方法列表（函数指针 IMP），各自映射到自己的方法实现，只要我们能操作这个函数指针的指向，我们就可以动态的增加替换原有的方法。

3. 互换两个已经写好的方法实现：

   ```objective-c
   void method_exchangeImplementations(Method m1, Method m2) 
     
   方法实现获取：
   Method class_getInstanceMethod(Class cls, SEL name)  
   ```

4. 为已有方法增加新功能：

   ```objective-c
   Method originalMethod = class_getInstanceMethod(class, originalSelector);
       Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);
       BOOL didAddMethod =
       class_addMethod(class,
                       originalSelector,
                       method_getImplementation(swizzledMethod),
                       method_getTypeEncoding(swizzledMethod));
       
       if (didAddMethod) {
           class_replaceMethod(class,
                               swizzledSelector,                            			   	    method_getImplementation(originalMethod),
   				   method_getTypeEncoding(originalMethod));
       } else {
          method_exchangeImplementations(originalMethod, swizzledMethod);
       }
   ```



> * 在运行期，可以向类中新增或替换选择子所对应的方法实现。
> * 使用另一份实现来替换原有的方法实现，这道工序叫做 “方法调配”，开发者常用此技术向原有实现中添加新功能。
> * 一般来说，只有调试程序的时候才需要在运行期修改方法实现，这种做法不宜滥用。

### 第 14 条：理解 “类对象” 的用意

1. 每个Objective-C 对象实例都是指向某块内存数据的指针。

2. Objective-C 对象所用的数据结构

   ```objective-c
   struct objc_object {
       Class isa;
   };

   /// A pointer to an instance of a class.
   typedef struct objc_object *id;
   ```

   每个对象结构体首个成员是Class 类的变量，定义了对象所属的类，通常称为 “is a” 指针。

3. Class 对象的数据结构定义：

   ```objective-c
   struct objc_class {
       Class isa  OBJC_ISA_AVAILABILITY;

   #if !__OBJC2__
       Class super_class                                        OBJC2_UNAVAILABLE;
       const char *name                                         OBJC2_UNAVAILABLE;
       long version                                             OBJC2_UNAVAILABLE;
       long info                                                OBJC2_UNAVAILABLE;
       long instance_size                                       OBJC2_UNAVAILABLE;
       struct objc_ivar_list *ivars                             OBJC2_UNAVAILABLE;
       struct objc_method_list **methodLists                    OBJC2_UNAVAILABLE;
       struct objc_cache *cache                                 OBJC2_UNAVAILABLE;
       struct objc_protocol_list *protocols                     OBJC2_UNAVAILABLE;
   #endif

   } OBJC2_UNAVAILABLE;
   ```

   Class 首个变量也是isa 指针，说明Class 本身也是Objective-C 对象，指向 “元类”（meta class）。

**在类继承体系中查询类型信息**

1. “isMemberOfClass” 判断对象是否为某个特定类的实例
2. “isKindOfClass” 判断出对象是否为某类或其派生类的实例



> * 每个实例都有一个指向Class 对象的指针，用以表明其类型，而这些Class 对象则构成了类的继承体系。
> * 如果对象类型无法在编译器确定，那么就应该使用类型信息查询方法来探知。
> * 尽量使用类型信息查询方法来确定对象类型，而不要直接比较类对象，因为某些对象可能实现了消息转发功能。