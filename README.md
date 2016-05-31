# 2016-5-27

Objective-C提供了三种内存管理方式：manual retain-release（MRR，手动管理），automatic reference counting（ARC，自动引用计数），garbage collection（垃圾回收）。iOS不支持垃圾回收；ARC作为苹果新提供的技术，苹果推荐开发者使用ARC技术来管理内存；这篇笔记主要讲的是手动管理。

内存管理的目的是：
1.不要释放或者覆盖还在使用的内存，这会引起程序崩溃；
2.释放不再使用的内存，防止内存泄露。iOS程序的内存资源是宝贵的。

MRR手动管理内存也是基于引用计数的，只是需要开发者发消息给某块内存（或者说是对象）来改变这块内存的引用计数以实现内存管理（ARC技术则是编译器代替开发者完成相应的工作）。一块内存如果计数是零，也就是没有使用者（owner），那么objective-C的运行环境会自动回收这块内存。

objective-C的内存管理遵守下面这个简单的策略：
注：文档中把引用计数加1的操作称为“拥有”（own，或者take ownership of）某块对象/内存；把引用计数减1的操作称为放弃（relinquish）这块对象/内存。拥有对象时，你可以放心地读写或者返回对象；当对象被所有人放弃时，objective-C的运行环境会回收这个对象。
1.你拥有你创建的对象
也就是说创建的对象（使用alloc，new，copy或者mutalbeCopy等方法）的初始引用计数是1。
2.给对象发送retain消息后，你拥有了这个对象
3.当你不需要使用该对象时，发送release或者autorelease消息放弃这个对象
4.不要对你不拥有的对象发送“放弃”的消息


注：简单的赋值不会拥有某个对象。比如：
...
NSString *name = person.fullName;
...
上面这个赋值操作不会拥有这个对象（这仅仅是个指针赋值操作）；这和C++语言里的某些基于引用计数的类的行为是有区别的。想拥有一个objective-C对象，必须发送“创建”或者retain消息给该对象。


dealloc方法
dealloc方法用来释放这个对象所占的内存(包括成员变量)和其它资源。
不要使用dealloc方法来管理稀缺资源，比如文件，网络链接等。因为由于bug或者程序意外退出，dealloc方法不能保证一定会被调用。


Accessor Methods和内存管理
Accessor Methods，也就是对象的property（属性）的getter和setter方法。显然，如果getter返回的对象已经被运行环境回收了，那么这个getter的返回值是毫无意义的。这就需要在setter方法里“拥有”相应的property。
比如：
@interface Counter : NSObject
@property (nonatomic, retain) NSNumber *count;
@end
getter方法仅仅返回成员变量就可以：
-(NSNumber *)count {
    return _count;
}
setter方法需要保证对这个成员变量的“拥有”：
-(void)setCount:(NSNumber *)newCount {
    [newCount retain]; //拥有新值
    [_count release]; //放弃老值
    _count = newCount; //简单赋值
}

使用Accessor Methods
以下是一种使用方式：
...
NSNumber *zero = ［NSNumber alloc] initWithInteger:0];
[self setCount:zero];
[zero release];
...
以下是一种可能引发错误的，偷懒的使用方式：
...
NSNumber *zero = ［NSNumber alloc] initWithInteger:0];
[_count release];
_count = zero; //这个代码做了不合理的假设－_count对象“拥有”了某个内存。当修饰count属性的
                         //attribute发声变化时，这个假设就不一定正确了。
...

不要在初始化方法（Initializer）和dealloc方法里使用Accessor Methods
不要在初始化方法里使用accessor methods的原因可能是（原文档中没有说明）：在初始化方法里，成员变量处于最初的状态，并没有任何值。考虑到一个成员变量的setter方法一般会对成员变量的旧值发送release消息。这种行为在初始化方法里没有意义。
如果需要在Initializer里给成员变量赋值，可参见一开始提到的原始文档里给出的示例代码。


使用weak reference（弱引用）来避免retain cycle
对一个对象发送retain消息会创建对这个对象的强引用（strong reference）。如果两个对象都有一个强引用指向对方，那么就形成了一个环（retain cycle）。这个环使得这两个对象都不可能被release。
弱引用（weak reference）指的是一种non-owning（非拥有）的关系，比如简单指针赋值关系。使用弱引用避免了retain cycle。但是需要注意的是，弱引用不能保证弱引用指向的对象是否存在，所以发消息给这个对象时一定要小心。如果弱引用指向的对象已经释放，那么发送消息给它会导致程序崩溃。所以，需要一点点额外的操作来使用弱引用所指的对象。比如，当向notification center注册一个对象时，notification center保存了一个指向这个对象的弱引用。当这个对象被回收时，需要通知下notification center。


当你使用对象时，要确保这个对象不会被回收。主要要注意以下两种情形：
1.当一个对象从collection对象（collection指的数组之类的集合）移除时，如果这个仅被collection对象拥有，那么移除操作了会被即可回收。所以如果要使用这个将要移除的对象，要先retain。
2.当“父”对象回收时。这和情形1类似。


Autorelease Pool
Autorelease Pool可以延后发送release消息给一个对象。发送一个autorelease消息给一个对象，相当于说这个对象在“一定时期”内都有效，“一定时期”后再release这个对象。
Autorelease Pool几个要点：
－autorelease pool是一个NSAutoreleasePool对象。
－程序里的所有autorelease pool是以桟（stack）的形式组织的。新创建的pool位于桟的最顶端。当发送autorelease消息给一个对象时，这个对象被加到栈顶的那个pool中。发送drain给一个pool时，这个pool里所有对象都会受到release消息，而且如果这个pool不是位于栈顶，那么位于这个pool“上端”的所有pool也会受到drain消息。
－一个对象被加到一个pool很多次，只要多次发送autorelease消息给这个对象就可以；同时，当这个pool被回收时，这个对象也会收到同样多次release消息。简单地可以认为接收autorelease消息等同于：接收一个retain消息，同时加入到一个pool里；这个pool用来存放这些暂缓回收的对象；一旦这个pool被回收（drain），那么pool里面的对象会收到同样次数的release消息。
－UIKit框架已经帮你自动创建一个autorelease pool。大部分时候，你可以直接使用这个pool，不必自己创建；所以你给一个对象发送autorelease消息，那么这个对象会加到这个UIKit自动创建的pool里。某些时候，可能需要创建一个pool：
1.没有使用UIKit框架或者其它内含autorelease pool的框架，那么要使用pool，就要自己创建。
2.如果一个循环体要创建大量的临时变量，那么创建自己的pool可以减少程序占用的内存峰值。（如果使用UIKit的pool，那么这些临时变量可能一直在这个pool里，只要这个pool受到drain消息；完全不使用autorelease pool应该也是可以的，可能只是要发一些release消息给这些临时变量，所以使用autorelease pool还是方便一些）
3.创建线程时必须创建这个线程自己的autorelease pool。
－使用alloc和init消息来创建pool，发送drain消息则表示这个pool不再使用。pool的创建和drain要在同一上下文中，比如循环体内。
一、计数器的基本操作
1> retain : +1
2> release :-1
3> retainCount : 获得计数器

二、set方法的内存管理
1> set方法的实现
- (void)setCar:(Car *)car
{
    if ( _car != car )
    {
        [_car release];
        _car = [car retain];
    }
}

2> dealloc方法的实现(不要直接调用dealloc)
- (void)dealloc
{
    [_car release];
    [super dealloc];
}

三、@property参数
1>　OC对象类型
@property (nonatomic, retain) 类名 *属性名;
@property (nonatomic, retain) Car *car;
@property (nonatomic, retain) id car;

// 被retain过的属性，必须在dealloc方法中release属性
- (void)dealloc
{
    [_car release];
    [super dealloc];
}

2>　非OC对象类型（int\float\enum\struct）
@property (nonatomic, assign) 类型名称 属性名;
@property (nonatomic, assign) int age;

四、autorelease
1.系统自带的方法中，如果不包含alloc、new、copy，那么这些方法返回的对象都是已经autorelease过的
[NSString stringWithFormat:....];
[NSDate date];

2.开发中经常写一些类方法快速创建一个autorelease的对象
* 创建对象的时候不要直接使用类名，用self

