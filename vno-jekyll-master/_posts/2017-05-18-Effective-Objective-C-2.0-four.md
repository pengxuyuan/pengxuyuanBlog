---
layout: post
title: Effective Objective-C 2.0 总结（四）
date: 2017-05-17
---

## 协议与分类

[TOC]

### 第 23 条：通过委托与数据源协议进行对象间通信

1. Objective-C 可以使用 “委托模式”（Delegate pattern）的编程设计模式来实现对象间的通信：定义一套接口，某对象若想接受另一个对象的委托，则需遵从此接口，以便成为其 “委托对象”（delegate）。Objective-C 一般利用 “协议” 机制来实现此模式。

2. 定义协议：

   ```objective-c
   @protocol EOCNetworkingFetcherDelegate
   @optional
   - (void)newworkingFetcher:(EOCNetworkingFetcher *)fetcher
    		   didRecevieData:(NSData *)data;
   - (void)newworkingFetcher:(EOCNetworkingFetcher *)fetcher
            didFailWithError:(NSError *)error;
   @end
     
   @interface EOCNetworkingFetcher : NSObject
   @property (nonatomic,weak) id<EOCNetworkingFetcherDelegate> delegate;
   @end
   ```

   * 委托协议名通常时在相关的类名加上Delegate 一词，也是采用 “驼峰法” 来命名。
   * 类可以用一个属性存放其委托对象，属性要用weak 来修饰，避免产生 “保留环”（retain cycle）。
   * 某类若要遵从某委托协议，可以在其接口中声明，也可以在"class-continuation 分类" 中声明，如果要向外界公布此类实现了某协议，就在接口中声明，如果这个协议是个委托协议，通常只会在这个类的内部使用，这样子就在分类中声明就好了。

3. 如果要在委托对象上调用可选方法，那么必须提前使用类型信息查询方法，判断这个委托对象能否响应相关的选择子。

   ```objective-c
   NSData *data;
   if([_delegate respondsToSelector:@selector(networkFetcher:didRecevieData:)]){
   	[_delegate networkFetcher:self didRecevieData:data];
   }
   ```

   * 在调用delegate 对象中的方法时，总应该把发起委托的实例也一并传入方法中，这样子，delegate 对象在实现相关方法时，就能根据传入的实例分别执行不同的代码了。

4. delegate 里的方法也可以用于从委托对象中获取信息（数据源模式）。

5. 在实现委托模式和数据源模式的时，协议中的方法是可选的，我们就会写出大量这种判断代码：

   ```objective-c
   if([_delegate respondsToSelector:@selector(networkFetcher:didRecevieData:)]){
   	[_delegate networkFetcher:self didRecevieData:data];
   }
   ```

   * 每次调用方法都会判断一次，其实除了第一次检测的结构有用，后续的检测很有可能都是多余的，因为委托对象本身没变，不太可能会一下子不响应，一下子响应的，所以我们这里可以把这个委托对象能否响应某个协议方法记录下来，以优化程序效率。

   * 将方法响应能力缓存起来的最佳途径是使用 “位段”（bitfield）数据类型。我们可以把结构体中某个字段所占用的二进制位个数设为特定的值。

     > 位段，C语言允许在一个结构体中以位为单位来指定其成员所占内存长度，这种以位为单位的成员称为“位段”或称“位域”( bit field) 。
     >
     > struct data {
     >
     > ​	unsigned int filedA : 8;
     >
     > ​	unsigned int filedB : 4;
     >
     > ​	unsigned int filedC : 2;
     >
     > ​	unsigned int filedD : 1;
     >
     > }
     >
     > filedA 位段占用8个二进制位，filedB 位段占用4个二进制位，filedC 位段占用2个二进制位，filedD位段占用1个二进制位。filedA 就可以表示0至255之间的值，而filedD 则可以表示0或1这两个值。
     >
     > 我们可以像filedD 这样子，创建大小只有1的位段，这样子就可以把Boolean 值塞入这一小块数据里面，这里很适合这样子做。

   * 利用位段就可以清楚的表示delegate 对象是否能响应协议中的方法。

     ```objective-c
     @interface EOCNetworkingFetcher ()
     struct {
     	unsigned int didReceiveData : 1;
     	unsigned int didFailWithError : 1;
     	unsigned int didUpdateProgressTo : 1;
     } _delegateFlags
     @end
       
     //使用
     //set flag
     _delageteFlags.didReceiveData = 1;

     //check flag
     if(_delageteFlags.didReceiveData){
     	//YES
     }
     ```

   * 可以在delegate 属性的设置方法里面写实现缓存功能所用的代码。 

   * 这样子，每次调用delegate 的相关方法之前，就不用检测委托对象是否能响应给定的选择子了，而是直接查询结构体里面的标志。

   * 在相关方法需要调用很多次时，就要思考是否有必要进行优化，分析代码性能，找出瓶颈，使用这个位段这个技术可以提供执行速度。





> * 委托模式为对象提供了一套接口，使其可由此将相关事件告知其他对象。
> * 将委托对象应该支持的接口定义成协议，在协议中把可能需要处理的事件定义成方法。
> * 当某对象需要从另外一个对象中获取数据时，可以使用委托模式。这种情境下，该模式亦称 “数据源协议”（data source protocal）。
> * 若有必要，可实现含有位段的结构体，将委托对象是否能响应相关协议方法这一信息缓存至其中。

### 第 24 条：将类的实现代码分散到便于管理的数个分类之中

1. 一个类经常有很多方法，尽管代码写的比较规范，这个文件还是会越来越大，定位问题以及阅读上都会造成不便。我们可以通过 “分类” 机制来把代码按逻辑划分到几个分区中。
2. 通过分类机制，可以把类代码分成很多个易于管理的小块，以便单独检视。
3. 可以考虑创建Private 分类，将一些不是公共API 的方法，隐藏起来。写程序库的时候，加上不暴露头文件，使用者就不知道库里还有这些私有方法。 





> *  使用分类机制把类的实现代码划分成易于管理的小块。
> *  将应该视为 “私有” 的方法归入为叫Private 的分类中，以隐藏实现细节。

### 第 25 条：总是为第三方类的分类名称加前缀

1. 分类机制常用于向无源码的既有类中新增新功能，但是在使用的时候要十分小心，不然很容易产生Bug。因为这个机制时在运行期系统加载分类时，将其方法直接加到原类中，这里要注意方法重名的问题，不然会覆盖原类中的同名方法。
2. 一般用前缀来区分各个分类的名称与其中所定义的方法。
3. 不要轻易去利用分类来覆盖方法，这里需要慎重考虑。





> * 向第三方类中添加分类时，总应该给其名称加上你专用的前缀。
> * 向第三方类中添加分类时，总应给其中的方法名加上你专用的前缀

### 第 26 条：勿在分类中声明属性

1. 可以利用运行期的关联对象机制，为分类声明属性，但是这种做法要尽量避免，因为除了 "class-continuation 分类" 之外，其他分类都无法向类中新增实例变量，因此，他们无法把实现属性所需的实例变量合成出来。

2. 在分类定义属性的时候，会报警告，表明此分类无法合成该属性相关的实例变量，所以开发者需要在分类中为该属性实现存取方法。

3. 利用关联对象机制可以解决分类中不能合成实例变量的问题。自己实现存取方法，但是要注意该属性的内存管理语义（属性特质）。

   ```objective-c
   @property (nonatomic,copy) NSString *name;

   static const void *kViewControllerName = &kViewControllerName;

   - (void)setName:(NSString *)name {
       objc_setAssociatedObject(self, kViewControllerName, name, OBJC_ASSOCIATION_COPY_NONATOMIC);
   }

   - (NSString *)name {
       NSString *myName = objc_getAssociatedObject(self, kViewControllerName);
       return myName;
   }
   ```

4. 在可以修改源代码的情况下，尽量把属性定义在主接口中，这里是唯一能够定义实例变量的地方，属性只是定义实例变量及相关存取方法所用的 “语法糖”。

5. 由于实现属性所需的全部方法都已实现，所以不会再为该属性自动合成实例变量了。





> * 尽量把封装数据所用的全部属性都定义在主接口里。
> * 在 “class-continuation 分类” 之外的其他分类中，可以定义存取方法，但尽量不要定义属性。

### 第 27 条：使用 ”class-continuation 分类“ 隐藏实现细节

1. ”class-continuation 分类“ 必须定义在本身类的实现文件中，而且这里是唯一可以声明实例变量的分类，而且此分类没有特定的实现文件，这个分类也没有名字。这里可以定义实例变量的原因是 “ 稳固的ABI” 机制，我们无须知道对象的大小就可以直接使用它。

   ```	objective-c
   @interface EOCPerson ()

   @end
   ```

2. 可以将不需要要暴露给外界知道的实例变量及方法写在 “class-continuation 分类” 中。

3. 编写Objective-C++ 代码时候，使用 “class-continuation 分类” 会十分方便。因为对于引用了C++的文件的实现文件需要用.mm 为扩展名，表示编译器应该将此文件按照Objective-C++ 来编译。C++ 类必须完全引入，编译器要完整地解析其定义才能得知这个C++ 对象的实例变量大小。如果把对C++ 类的引用写在头文件的话，其他引用到这个类也会引用到这个C++ 类，就也需要编译成Objective-C++ 才行，这样子很容易失控。

   这里可以利用 “class-continuation 分类” 把引用C++ 类的细节写到实现文件中，这样子别的类引用这个类就不会受到影响，甚至都不知道这个类底层实现混有C++ 代码。

4. 使用 “class-continuation 分类” 还可以将头文件声明 “只读” 的属性扩展成 “可读写”，以便在类的内部可以设置其值。

5. 我们通常不直接访问实例变量，而是通过设置方法来做，因为这样子可以触发 “键值观测” （Key-Value Observing，KVO）通知。

6. 若对象所遵循的协议只应视为私有，也可以同过“class-continuation 分类” 来隐藏。





> * 通过 “class-continuation 分类” 向类中新增实例变量。
> * 如果某属性在主接口中声明为 “只读”，而类的内部又要用设置方法修改此属性，那么就在 “class-continuation 分类” 中将其扩展为 “可读写”。
> * 把私有方法的原型声明在 “class-contiunation 分类” 里面。
> * 若想使类所遵循的协议不为人所知，则可于 “class-contiunation 分类” 中声明。

### 第 28 条：通过协议提供匿名对象

1. ```objective-c
   @property (nonatomic,weak) id<EOCDelegate> delegate;
   ```

   该属性类型是id\<EOCDelegate> 的，所以实际上任何类的都能充当这一属性，即便该类不继承NSObject 也可以，只要遵循EOCDelegae 协议就可以了，对于具备此属性的类来说，delegate 就是 “匿名的”。



> * 协议可在某种程度上提供匿名类型。具体的对象类型可以淡化成遵从某协议的id 类型，协议里规定了对象所应实现的方法。
> * 使用匿名对象来隐藏类型名称（或类名）。
> * 如果具体类型不重要，重要的是对象能够响应（定义在协议里的）特定方法，那么可使用匿名对象来表示。