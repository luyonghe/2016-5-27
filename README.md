# 2017-9-12



1.ReactiveCocoa简介

ReactiveCocoa（简称为`RAC`）,是由Github开源的一个应用于iOS和OS开发的新框架,Cocoa是苹果整套框架的简称，因此很多苹果框架喜欢以Cocoa结尾。

2.ReactiveCocoa作用

在我们iOS开发过程中，经常会响应某些事件来处理某些业务逻辑，例如按钮的点击，上下拉刷新，网络请求，属性的变化（通过KVO）或者用户位置的变化（通过CoreLocation）。但是这些事件都用不同的方式来处理，比如action、delegate、KVO、callback等。

其实这些事件，都可以通过RAC处理，ReactiveCocoa为事件提供了很多处理方法，而且利用RAC处理事件很方便，可以把要处理的事情，和监听的事情的代码放在一起，这样非常方便我们管理，就不需要跳到对应的方法里。非常符合我们开发中`高聚合，低耦合`的思想。

 3.编程思想

在开发中我们也不能太依赖于某个框架，否则这个框架不更新了，导致项目后期没办法维护，比如之前Facebook提供的`Three20框架`，在当时也是神器，但是后来不更新了，也就没什么人用了。因此我感觉学习一个框架，还是有必要了解它的`编程思想`。

先简单介绍下目前咱们已知的`编程思想`。

3.1 `面向过程`：处理事情以过程为核心，一步一步的实现。

3.2 `面向对象`：万物皆对象

3.3 `链式编程思想`：是将多个操作（多行代码）通过点号(.)链接在一起成为一句代码,使代码可读性好。a(1).b(2).c(3)

*	`链式编程特点`：方法的返回值是block,block必须有返回值（本身对象），block参数（需要操作的值）

*	`代表`：masonry框架。

3.4 `响应式编程思想`：不需要考虑调用顺序，只需要知道考虑结果，类似于蝴蝶效应，产生一个事件，会影响很多东西，这些事件像流一样的传播出去，然后影响结果，借用面向对象的一句话，万物皆是流。

*	`代表`：KVO运用。



3.5 `函数式编程思想`：是把操作尽量写成一系列嵌套的函数或者方法调用。

*	`函数式编程本质`:就是往方法中传入Block,方法中嵌套Block调用，把代码聚合起来管理
*	`函数式编程特点`：每个方法必须有返回值（本身对象）,把函数或者Block当做参数,block参数（需要操作的值）block返回值（操作结果）

*	`代表`：ReactiveCocoa。


 4.ReactiveCocoa编程思想
ReactiveCocoa结合了几种编程风格：

`函数式编程（Functional Programming）`

`响应式编程（Reactive Programming）`

所以，你可能听说过ReactiveCocoa被描述为函数响应式编程（FRP）框架。

以后使用RAC解决问题，就不需要考虑调用顺序，直接考虑结果，把每一次操作都写成一系列嵌套的方法中，使代码高聚合，方便管理。


5.如何导入ReactiveCocoa框架
通常都会使用CocoaPods（用于管理第三方框架的插件）帮助我们导入。

PS:CocoaPods教程（http://code4app.com/article/cocoapods-install-usage）



`注意`：
*   podfile如果只描述pod 'ReactiveCocoa', '~> 4.0.2-alpha-1'，会导入不成功
![](Snip20150926_1.png)
*   报错信息
![](Snip20150926_2.png)
*   需要在podfile加上use_frameworks，重新pod install 才能导入成功。
![](Snip20150926_3.png)


 6.ReactiveCocoa常见类。

学习框架首要之处:个人认为先要搞清楚框架中常用的类，在RAC中最核心的类RACSiganl,搞定这个类就能用ReactiveCocoa开发了。



`RACSiganl`:信号类,一般表示将来有数据传递，只要有数据改变，信号内部接收到数据，就会马上发出数据。

*	信号类(RACSiganl)，只是表示当数据改变时，信号内部会发出数据，它本身不具备发送信号的能力，而是交给内部一个订阅者去发出。

*	默认一个信号都是冷信号，也就是值改变了，也不会触发，只有订阅了这个信号，这个信号才会变为热信号，值改变了才会触发。

*	如何订阅信号：调用信号RACSignal的subscribeNext就能订阅。

*	`RACSiganl简单使用:`

```
	// RACSignal使用步骤：
    // 1.创建信号 + (RACSignal *)createSignal:(RACDisposable * (^)(id<RACSubscriber> subscriber))didSubscribe
    // 2.订阅信号,才会激活信号. - (RACDisposable *)subscribeNext:(void (^)(id x))nextBlock
    // 3.发送信号 - (void)sendNext:(id)value


    // RACSignal底层实现：
    // 1.创建信号，首先把didSubscribe保存到信号中，还不会触发。
    // 2.当信号被订阅，也就是调用signal的subscribeNext:nextBlock
    // 2.2 subscribeNext内部会创建订阅者subscriber，并且把nextBlock保存到subscriber中。
    // 2.1 subscribeNext内部会调用siganl的didSubscribe
    // 3.siganl的didSubscribe中调用[subscriber sendNext:@1];
    // 3.1 sendNext底层其实就是执行subscriber的nextBlock

    // 1.创建信号
    RACSignal *siganl = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {

        // block调用时刻：每当有订阅者订阅信号，就会调用block。

        // 2.发送信号
        [subscriber sendNext:@1];

        // 如果不在发送数据，最好发送信号完成，内部会自动调用[RACDisposable disposable]取消订阅信号。
        [subscriber sendCompleted];

        return [RACDisposable disposableWithBlock:^{

            // block调用时刻：当信号发送完成或者发送错误，就会自动执行这个block,取消订阅信号。

            // 执行完Block后，当前信号就不在被订阅了。

            NSLog(@"信号被销毁");

        }];
    }];

    // 3.订阅信号,才会激活信号.
    [siganl subscribeNext:^(id x) {
        // block调用时刻：每当有信号发出数据，就会调用block.
        NSLog(@"接收到数据:%@",x);
    }];

```

`RACSubscriber`:表示订阅者的意思，用于发送信号，这是一个协议，不是一个类，只要遵守这个协议，并且实现方法才能成为订阅者。通过create创建的信号，都有一个订阅者，帮助他发送数据。

`RACDisposable`:用于取消订阅或者清理资源，当信号发送完成或者发送错误的时候，就会自动触发它。

*	`使用场景`:不想监听某个信号时，可以通过它主动取消订阅信号。

`RACSubject`:RACSubject:信号提供者，自己可以充当信号，又能发送信号。

*	`使用场景`:通常用来代替代理，有了它，就不必要定义代理了。


`RACReplaySubject`:重复提供信号类，RACSubject的子类。
*   `RACReplaySubject`与`RACSubject`区别:
    *   RACReplaySubject可以先发送信号，在订阅信号，RACSubject就不可以。
*	`使用场景一`:如果一个信号每被订阅一次，就需要把之前的值重复发送一遍，使用重复提供信号类。
*	`使用场景二`:可以设置capacity数量来限制缓存的value的数量,即只缓充最新的几个值。

*	`RACSubject和RACReplaySubject简单使用:`

```
    // RACSubject使用步骤
    // 1.创建信号 [RACSubject subject]，跟RACSiganl不一样，创建信号时没有block。
    // 2.订阅信号 - (RACDisposable *)subscribeNext:(void (^)(id x))nextBlock
    // 3.发送信号 sendNext:(id)value

    // RACSubject:底层实现和RACSignal不一样。
    // 1.调用subscribeNext订阅信号，只是把订阅者保存起来，并且订阅者的nextBlock已经赋值了。
    // 2.调用sendNext发送信号，遍历刚刚保存的所有订阅者，一个一个调用订阅者的nextBlock。

    // 1.创建信号
    RACSubject *subject = [RACSubject subject];

    // 2.订阅信号
    [subject subscribeNext:^(id x) {
        // block调用时刻：当信号发出新值，就会调用.
        NSLog(@"第一个订阅者%@",x);
    }];
    [subject subscribeNext:^(id x) {
        // block调用时刻：当信号发出新值，就会调用.
        NSLog(@"第二个订阅者%@",x);
    }];

    // 3.发送信号
    [subject sendNext:@"1"];


    // RACReplaySubject使用步骤:
    // 1.创建信号 [RACSubject subject]，跟RACSiganl不一样，创建信号时没有block。
    // 2.可以先订阅信号，也可以先发送信号。
    // 2.1 订阅信号 - (RACDisposable *)subscribeNext:(void (^)(id x))nextBlock
    // 2.2 发送信号 sendNext:(id)value

    // RACReplaySubject:底层实现和RACSubject不一样。
    // 1.调用sendNext发送信号，把值保存起来，然后遍历刚刚保存的所有订阅者，一个一个调用订阅者的nextBlock。
    // 2.调用subscribeNext订阅信号，遍历保存的所有值，一个一个调用订阅者的nextBlock

    // 如果想当一个信号被订阅，就重复播放之前所有值，需要先发送信号，在订阅信号。
    // 也就是先保存值，在订阅值。

    // 1.创建信号
    RACReplaySubject *replaySubject = [RACReplaySubject subject];

    // 2.发送信号
    [replaySubject sendNext:@1];
    [replaySubject sendNext:@2];

    // 3.订阅信号
    [replaySubject subscribeNext:^(id x) {

        NSLog(@"第一个订阅者接收到的数据%@",x);
    }];

    // 订阅信号
    [replaySubject subscribeNext:^(id x) {

        NSLog(@"第二个订阅者接收到的数据%@",x);
    }];

```

*	`RACSubject替换代理`

```
	// 需求:
	// 1.给当前控制器添加一个按钮，modal到另一个控制器界面
	// 2.另一个控制器view中有个按钮，点击按钮，通知当前控制器

步骤一：在第二个控制器.h，添加一个RACSubject代替代理。
@interface TwoViewController : UIViewController

@property (nonatomic, strong) RACSubject *delegateSignal;

@end

步骤二：监听第二个控制器按钮点击
@implementation TwoViewController
- (IBAction)notice:(id)sender {
    // 通知第一个控制器，告诉它，按钮被点了

     // 通知代理
     // 判断代理信号是否有值
    if (self.delegateSignal) {
        // 有值，才需要通知
        [self.delegateSignal sendNext:nil];
    }
}
@end

步骤三：在第一个控制器中，监听跳转按钮，给第二个控制器的代理信号赋值，并且监听.
@implementation OneViewController
- (IBAction)btnClick:(id)sender {

    // 创建第二个控制器
    TwoViewController *twoVc = [[TwoViewController alloc] init];

    // 设置代理信号
    twoVc.delegateSignal = [RACSubject subject];

    // 订阅代理信号
    [twoVc.delegateSignal subscribeNext:^(id x) {

        NSLog(@"点击了通知按钮");
    }];

    // 跳转到第二个控制器
    [self presentViewController:twoVc animated:YES completion:nil];

}
@end

```

`RACTuple`:元组类,类似NSArray,用来包装值.

`RACSequence`:RAC中的集合类，用于代替NSArray,NSDictionary,可以使用它来快速遍历数组和字典。

`使用场景`：1.字典转模型

`RACSequence和RACTuple简单使用`

```
    // 1.遍历数组
    NSArray *numbers = @[@1,@2,@3,@4];

    // 这里其实是三步
    // 第一步: 把数组转换成集合RACSequence numbers.rac_sequence
    // 第二步: 把集合RACSequence转换RACSignal信号类,numbers.rac_sequence.signal
    // 第三步: 订阅信号，激活信号，会自动把集合中的所有值，遍历出来。
    [numbers.rac_sequence.signal subscribeNext:^(id x) {

        NSLog(@"%@",x);
    }];


    // 2.遍历字典,遍历出来的键值对会包装成RACTuple(元组对象)
    NSDictionary *dict = @{@"name":@"xmg",@"age":@18};
    [dict.rac_sequence.signal subscribeNext:^(RACTuple *x) {

        // 解包元组，会把元组的值，按顺序给参数里面的变量赋值
        RACTupleUnpack(NSString *key,NSString *value) = x;

        // 相当于以下写法
//        NSString *key = x[0];
//        NSString *value = x[1];

        NSLog(@"%@ %@",key,value);

    }];


    // 3.字典转模型
    // 3.1 OC写法
    NSString *filePath = [[NSBundle mainBundle] pathForResource:@"flags.plist" ofType:nil];

    NSArray *dictArr = [NSArray arrayWithContentsOfFile:filePath];

    NSMutableArray *items = [NSMutableArray array];

    for (NSDictionary *dict in dictArr) {
        FlagItem *item = [FlagItem flagWithDict:dict];
        [items addObject:item];
    }

    // 3.2 RAC写法
    NSString *filePath = [[NSBundle mainBundle] pathForResource:@"flags.plist" ofType:nil];

    NSArray *dictArr = [NSArray arrayWithContentsOfFile:filePath];

    NSMutableArray *flags = [NSMutableArray array];

    _flags = flags;

    // rac_sequence注意点：调用subscribeNext，并不会马上执行nextBlock，而是会等一会。
    [dictArr.rac_sequence.signal subscribeNext:^(id x) {
        // 运用RAC遍历字典，x：字典

        FlagItem *item = [FlagItem flagWithDict:x];

        [flags addObject:item];

    }];

    NSLog(@"%@",  NSStringFromCGRect([UIScreen mainScreen].bounds));


    // 3.3 RAC高级写法:
    NSString *filePath = [[NSBundle mainBundle] pathForResource:@"flags.plist" ofType:nil];

    NSArray *dictArr = [NSArray arrayWithContentsOfFile:filePath];
    // map:映射的意思，目的：把原始值value映射成一个新值
    // array: 把集合转换成数组
    // 底层实现：当信号被订阅，会遍历集合中的原始值，映射成新值，并且保存到新的数组里。
    NSArray *flags = [[dictArr.rac_sequence map:^id(id value) {

        return [FlagItem flagWithDict:value];

    }] array];

```

`RACMulticastConnection`:用于当一个信号，被多次订阅时，为了保证创建信号时，避免多次调用创建信号中的block，造成副作用，可以使用这个类处理。

`使用注意`:RACMulticastConnection通过RACSignal的-publish或者-muticast:方法创建.

`RACMulticastConnection简单使用`:

```
    // RACMulticastConnection使用步骤:
    // 1.创建信号 + (RACSignal *)createSignal:(RACDisposable * (^)(id<RACSubscriber> subscriber))didSubscribe
    // 2.创建连接 RACMulticastConnection *connect = [signal publish];
    // 3.订阅信号,注意：订阅的不在是之前的信号，而是连接的信号。 [connect.signal subscribeNext:nextBlock]
    // 4.连接 [connect connect]

    // RACMulticastConnection底层原理:
    // 1.创建connect，connect.sourceSignal -> RACSignal(原始信号)  connect.signal -> RACSubject
    // 2.订阅connect.signal，会调用RACSubject的subscribeNext，创建订阅者，而且把订阅者保存起来，不会执行block。
    // 3.[connect connect]内部会订阅RACSignal(原始信号)，并且订阅者是RACSubject
    // 3.1.订阅原始信号，就会调用原始信号中的didSubscribe
    // 3.2 didSubscribe，拿到订阅者调用sendNext，其实是调用RACSubject的sendNext
    // 4.RACSubject的sendNext,会遍历RACSubject所有订阅者发送信号。
    // 4.1 因为刚刚第二步，都是在订阅RACSubject，因此会拿到第二步所有的订阅者，调用他们的nextBlock


    // 需求：假设在一个信号中发送请求，每次订阅一次都会发送请求，这样就会导致多次请求。
    // 解决：使用RACMulticastConnection就能解决.

    // 1.创建请求信号
   RACSignal *signal = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {


        NSLog(@"发送请求");

        return nil;
    }];
    // 2.订阅信号
    [signal subscribeNext:^(id x) {

        NSLog(@"接收数据");

    }];
    // 2.订阅信号
    [signal subscribeNext:^(id x) {

        NSLog(@"接收数据");

    }];

    // 3.运行结果，会执行两遍发送请求，也就是每次订阅都会发送一次请求


    // RACMulticastConnection:解决重复请求问题
    // 1.创建信号
    RACSignal *signal = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {


        NSLog(@"发送请求");
        [subscriber sendNext:@1];

        return nil;
    }];

    // 2.创建连接
    RACMulticastConnection *connect = [signal publish];

    // 3.订阅信号，
    // 注意：订阅信号，也不能激活信号，只是保存订阅者到数组，必须通过连接,当调用连接，就会一次性调用所有订阅者的sendNext:
    [connect.signal subscribeNext:^(id x) {

        NSLog(@"订阅者一信号");

    }];

    [connect.signal subscribeNext:^(id x) {

        NSLog(@"订阅者二信号");

    }];

    // 4.连接,激活信号
    [connect connect];

```

`RACCommand`:RAC中用于处理事件的类，可以把事件如何处理,事件中的数据如何传递，包装到这个类中，他可以很方便的监控事件的执行过程。

`使用场景`:监听按钮点击，网络请求

`RACCommand简单使用`

```

 	// 一、RACCommand使用步骤:
    // 1.创建命令 initWithSignalBlock:(RACSignal * (^)(id input))signalBlock
    // 2.在signalBlock中，创建RACSignal，并且作为signalBlock的返回值
    // 3.执行命令 - (RACSignal *)execute:(id)input

    // 二、RACCommand使用注意:
    // 1.signalBlock必须要返回一个信号，不能传nil.
    // 2.如果不想要传递信号，直接创建空的信号[RACSignal empty];
    // 3.RACCommand中信号如果数据传递完，必须调用[subscriber sendCompleted]，这时命令才会执行完毕，否则永远处于执行中。

    // 三、RACCommand设计思想：内部signalBlock为什么要返回一个信号，这个信号有什么用。
    // 1.在RAC开发中，通常会把网络请求封装到RACCommand，直接执行某个RACCommand就能发送请求。
    // 2.当RACCommand内部请求到数据的时候，需要把请求的数据传递给外界，这时候就需要通过signalBlock返回的信号传递了。

    // 四、如何拿到RACCommand中返回信号发出的数据。
    // 1.RACCommand有个执行信号源executionSignals，这个是signal of signals(信号的信号),意思是信号发出的数据是信号，不是普通的类型。
    // 2.订阅executionSignals就能拿到RACCommand中返回的信号，然后订阅signalBlock返回的信号，就能获取发出的值。

    // 五、监听当前命令是否正在执行executing

    // 六、使用场景,监听按钮点击，网络请求


	// 1.创建命令
    RACCommand *command = [[RACCommand alloc] initWithSignalBlock:^RACSignal *(id input) {


        NSLog(@"执行命令");

        // 创建空信号,必须返回信号
        //        return [RACSignal empty];

        // 2.创建信号,用来传递数据
        return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {

            [subscriber sendNext:@"请求数据"];

            // 注意：数据传递完，最好调用sendCompleted，这时命令才执行完毕。
            [subscriber sendCompleted];

            return nil;
        }];

    }];

    // 强引用命令，不要被销毁，否则接收不到数据
    _conmmand = command;


    // 3.执行命令
    [self.conmmand execute:@1];

    // 4.订阅RACCommand中的信号
    [command.executionSignals subscribeNext:^(id x) {

        [x subscribeNext:^(id x) {

            NSLog(@"%@",x);
        }];

    }];

    // RAC高级用法
    // switchToLatest:用于signal of signals，获取signal of signals发出的最新信号,也就是可以直接拿到RACCommand中的信号
    [command.executionSignals.switchToLatest subscribeNext:^(id x) {

        NSLog(@"%@",x);
    }];

    // 5.监听命令是否执行完毕,默认会来一次，可以直接跳过，skip表示跳过第一次信号。
    [[command.executing skip:1] subscribeNext:^(id x) {

        if ([x boolValue] == YES) {
            // 正在执行
            NSLog(@"正在执行");

        }else{
            // 执行完成
            NSLog(@"执行完成");
        }

    }];

```

`RACScheduler`:RAC中的队列，用GCD封装的。

`RACUnit` :表⽰stream不包含有意义的值,也就是看到这个，可以直接理解为nil.

`RACEvent`: 把数据包装成信号事件(signal event)。它主要通过RACSignal的-materialize来使用，然并卵。


 7.ReactiveCocoa开发中常见用法。

7.1 代替代理:

*	`rac_signalForSelector`：用于替代代理。

7.2 代替KVO :

*	`rac_valuesAndChangesForKeyPath`：用于监听某个对象的属性改变。

7.3 监听事件:

*	`rac_signalForControlEvents`：用于监听某个事件。

7.4 代替通知:

*	`rac_addObserverForName`:用于监听某个通知。

7.5 监听文本框文字改变:

*	`rac_textSignal`:只要文本框发出改变就会发出这个信号。

7.6 处理当界面有多次请求时，需要都获取到数据时，才能展示界面

*	`rac_liftSelector:withSignalsFromArray:Signals`:当传入的Signals(信号数组)，每一个signal都至少sendNext过一次，就会去触发第一个selector参数的方法。
*	使用注意：几个信号，参数一的方法就几个参数，每个参数对应信号发出的数据。

7.7 代码演示

```

  	// 1.代替代理
    // 需求：自定义redView,监听红色view中按钮点击
    // 之前都是需要通过代理监听，给红色View添加一个代理属性，点击按钮的时候，通知代理做事情
    // rac_signalForSelector:把调用某个对象的方法的信息转换成信号，就要调用这个方法，就会发送信号。
    // 这里表示只要redV调用btnClick:,就会发出信号，订阅就好了。
    [[redV rac_signalForSelector:@selector(btnClick:)] subscribeNext:^(id x) {
        NSLog(@"点击红色按钮");
    }];

    // 2.KVO
    // 把监听redV的center属性改变转换成信号，只要值改变就会发送信号
    // observer:可以传入nil
    [[redV rac_valuesAndChangesForKeyPath:@"center" options:NSKeyValueObservingOptionNew observer:nil] subscribeNext:^(id x) {

        NSLog(@"%@",x);

    }];

    // 3.监听事件
    // 把按钮点击事件转换为信号，点击按钮，就会发送信号
    [[self.btn rac_signalForControlEvents:UIControlEventTouchUpInside] subscribeNext:^(id x) {

        NSLog(@"按钮被点击了");
    }];

    // 4.代替通知
    // 把监听到的通知转换信号
    [[[NSNotificationCenter defaultCenter] rac_addObserverForName:UIKeyboardWillShowNotification object:nil] subscribeNext:^(id x) {
        NSLog(@"键盘弹出");
    }];

    // 5.监听文本框的文字改变
   [_textField.rac_textSignal subscribeNext:^(id x) {

       NSLog(@"文字改变了%@",x);
   }];

   // 6.处理多个请求，都返回结果的时候，统一做处理.
    RACSignal *request1 = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {

        // 发送请求1
        [subscriber sendNext:@"发送请求1"];
        return nil;
    }];

    RACSignal *request2 = [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
        // 发送请求2
        [subscriber sendNext:@"发送请求2"];
        return nil;
    }];

    // 使用注意：几个信号，参数一的方法就几个参数，每个参数对应信号发出的数据。
    [self rac_liftSelector:@selector(updateUIWithR1:r2:) withSignalsFromArray:@[request1,request2]];


}
// 更新UI
- (void)updateUIWithR1:(id)data r2:(id)data1
{
    NSLog(@"更新UI%@  %@",data,data1);
}

```

### 8.ReactiveCocoa常见宏。

8.1 `RAC(TARGET, [KEYPATH, [NIL_VALUE]])`:用于给某个对象的某个属性绑定。

`基本用法`

```
	// 只要文本框文字改变，就会修改label的文字
    RAC(self.labelView,text) = _textField.rac_textSignal;

```

8.2 `RACObserve(self, name) `:监听某个对象的某个属性,返回的是信号。

`基本用法`

```
[RACObserve(self.view, center) subscribeNext:^(id x) {

        NSLog(@"%@",x);
    }];

```

8.3  `@weakify(Obj)和@strongify(Obj)`,一般两个都是配套使用,解决循环引用问题.

8.4 `RACTuplePack`：把数据包装成RACTuple（元组类）

`基本用法`
```
	// 把参数中的数据包装成元组
    RACTuple *tuple = RACTuplePack(@10,@20);
```

 

8.5 `RACTupleUnpack`：把RACTuple（元组类）解包成对应的数据。

`基本用法`
```
	// 把参数中的数据包装成元组
    RACTuple *tuple = RACTuplePack(@"xmg",@20);

    // 解包元组，会把元组的值，按顺序给参数里面的变量赋值
    // name = @"xmg" age = @20
    RACTupleUnpack(NSString *name,NSNumber *age) = tuple;
```











# 2017-7-27
一些小知识点的纪录

一、 iPhone Size

手机型号	屏幕尺寸
iPhone 4 4s	320 * 480
iPhone 5 5s	320 * 568
iPhone 6 6s	375 * 667
iphone 6 plus 6s plus	414 * 736

二、 给navigation Bar 设置 title 颜色

UIColor *whiteColor = [UIColor whiteColor];
NSDictionary *dic = [NSDictionary dictionaryWithObject:whiteColor forKey:NSForegroundColorAttributeName];
[self.navigationController.navigationBar setTitleTextAttributes:dic];

三、 如何把一个CGPoint存入数组里

CGPoint  itemSprite1position = CGPointMake(100, 200);
NSMutableArray * array  = [[NSMutableArray alloc] initWithObjects:NSStringFromCGPoint(itemSprite1position),nil];
//    从数组中取值的过程是这样的：
CGPoint point = CGPointFromString([array objectAtIndex:0]);
NSLog(@"point is %@.", NSStringFromCGPoint(point));

谢谢@bigParis的建议，可以用NSValue进行基础数据的保存，用这个方法更加清晰明确。

CGPoint  itemSprite1position = CGPointMake(100, 200);
NSValue *originValue = [NSValue valueWithCGPoint:itemSprite1position];
NSMutableArray * array  = [[NSMutableArray alloc] initWithObjects:originValue, nil];
//    从数组中取值的过程是这样的：
NSValue *currentValue = [array objectAtIndex:0];
CGPoint point = [currentValue CGPointValue];
NSLog(@"point is %@.", NSStringFromCGPoint(point));

现在Xcode7后OC支持泛型了，可以用NSMutableArray<NSString *> *array来保存。

四、 UIColor 获取 RGB 值

UIColor *color = [UIColor colorWithRed:0.0 green:0.0 blue:1.0 alpha:1.0];
const CGFloat *components = CGColorGetComponents(color.CGColor);
NSLog(@"Red: %f", components[0]);
NSLog(@"Green: %f", components[1]);
NSLog(@"Blue: %f", components[2]);
NSLog(@"Alpha: %f", components[3]);

五、 修改textField的placeholder的字体颜色、大小

self.textField.placeholder = @"username is in here!";
[self.textField setValue:[UIColor redColor] forKeyPath:@"_placeholderLabel.textColor"];
[self.textField setValue:[UIFont boldSystemFontOfSize:16] forKeyPath:@"_placeholderLabel.font"];

六、两点之间的距离

static __inline__ CGFloat CGPointDistanceBetweenTwoPoints(CGPoint point1, CGPoint point2) { CGFloat dx = point2.x - point1.x; CGFloat dy = point2.y - point1.y; return sqrt(dx*dx + dy*dy);}

七、IOS开发－关闭/收起键盘方法总结

1、点击Return按扭时收起键盘

- (BOOL)textFieldShouldReturn:(UITextField *)textField
{
return [textField resignFirstResponder];
}

2、点击背景View收起键盘（你的View必须是继承于UIControl）

[self.view endEditing:YES];

3、你可以在任何地方加上这句话，可以用来统一收起键盘

[[[UIApplication sharedApplication] keyWindow] endEditing:YES];

八、在使用 ImagesQA.xcassets 时需要注意

将图片直接拖入image到ImagesQA.xcassets中时，图片的名字会保留。
这个时候如果图片的名字过长，那么这个名字会存入到ImagesQA.xcassets中，名字过长会引起SourceTree判断异常。

九、UIPickerView 判断开始选择到选择结束

开始选择的，需要在继承UiPickerView，创建一个子类，在子类中重载

- (UIView*)hitTest:(CGPoint)point withEvent:(UIEvent*)event

当[super hitTest:point withEvent:event]返回不是nil的时候，说明是点击中UIPickerView中了。结束选择的， 实现UIPickerView的delegate方法

- (void)pickerView:(UIPickerView*)pickerView didSelectRow:(NSInteger)row inComponent:(NSInteger)component

当调用这个方法的时候，说明选择已经结束了。

十、iOS模拟器 键盘事件

当iOS模拟器 选择了Keybaord->Connect Hardware keyboard 后，不弹出键盘。当代码中添加了

[[NSNotificationCenter defaultCenter] addObserver:self
selector:@selector(keyboardWillHide)
name:UIKeyboardWillHideNotification
object:nil];

进行键盘事件的获取。那么在此情景下将不会调用- (void)keyboardWillHide.
因为没有键盘的隐藏和显示。

十一、在ios7上使用size classes后上面下面黑色

使用了size classes后，在ios7的模拟器上出现了上面和下面部分的黑色
可以在General->App Icons and Launch Images->Launch Images Source中设置Images.xcassets来解决。

￼

十二、设置不同size在size classes

Font中设置不同的size classes。

￼

十三、线程中更新 UILabel的text

[self.label1 performSelectorOnMainThread:@selector(setText:)                                      withObject:textDisplay
waitUntilDone:YES];

label1 为UILabel，当在子线程中，需要进行text的更新的时候，可以使用这个方法来更新。其他的UIView 也都是一样的。

十四、使用UIScrollViewKeyboardDismissMode实现了Message app的行为

像Messages app一样在滚动的时候可以让键盘消失是一种非常好的体验。然而，将这种行为整合到你的app很难。幸运的是，苹果给UIScrollView添加了一个很好用的属性keyboardDismissMode，这样可以方便很多。

现在仅仅只需要在Storyboard中改变一个简单的属性，或者增加一行代码，你的app可以和办到和Messages app一样的事情了。

这个属性使用了新的UIScrollViewKeyboardDismissMode enum枚举类型。这个enum枚举类型可能的值如下：

typedef NS_ENUM(NSInteger, UIScrollViewKeyboardDismissMode) {
UIScrollViewKeyboardDismissModeNone,
UIScrollViewKeyboardDismissModeOnDrag,      // dismisses the keyboard when a drag begins
UIScrollViewKeyboardDismissModeInteractive, // the keyboard follows the dragging touch off screen, and may be pulled upward again to cancel the dismiss
} NS_ENUM_AVAILABLE_IOS(7_0);

以下是让键盘可以在滚动的时候消失需要设置的属性：

￼

十五、报错 "_sqlite3_bind_blob", referenced from:

将 sqlite3.dylib加载到framework

十六、ios7 statusbar 文字颜色

iOS7上，默认status bar字体颜色是黑色的，要修改为白色的需要在infoPlist里设置UIViewControllerBasedStatusBarAppearance为NO，然后在代码里添加：

[application setStatusBarStyle:UIStatusBarStyleLightContent];

十七、获得当前硬盘空间

NSFileManager *fm = [NSFileManager defaultManager];
NSDictionary *fattributes = [fm attributesOfFileSystemForPath:NSHomeDirectory() error:nil];
NSLog(@"容量%lldG",[[fattributes objectForKey:NSFileSystemSize] longLongValue]/1000000000);
NSLog(@"可用%lldG",[[fattributes objectForKey:NSFileSystemFreeSize] longLongValue]/1000000000);

十八、给UIView 设置透明度，不影响其他sub views

UIView设置了alpha值，但其中的内容也跟着变透明。有没有解决办法？设置background color的颜色中的透明度，比如：

[self.testView setBackgroundColor:[UIColor colorWithRed:0.0 green:1.0 blue:1.0 alpha:0.5]];

设置了color的alpha， 就可以实现背景色有透明度，当其他sub views不受影响给color 添加 alpha，或修改alpha的值。

// Returns a color in the same color space as the receiver with the specified alpha component.
- (UIColor *)colorWithAlphaComponent:(CGFloat)alpha;
// eg.
[view.backgroundColor colorWithAlphaComponent:0.5];

十九、将color转为UIImage

//将color转为UIImage
- (UIImage *)createImageWithColor:(UIColor *)color
{
CGRect rect = CGRectMake(0.0f, 0.0f, 1.0f, 1.0f);
UIGraphicsBeginImageContext(rect.size);
CGContextRef context = UIGraphicsGetCurrentContext();
CGContextSetFillColorWithColor(context, [color CGColor]);
CGContextFillRect(context, rect);
UIImage *theImage = UIGraphicsGetImageFromCurrentImageContext();
UIGraphicsEndImageContext();
return theImage;
}

二十、NSTimer 用法

NSTimer *timer = [NSTimer scheduledTimerWithTimeInterval:.02 target:self selector:@selector(tick:) userInfo:nil repeats:YES];
[[NSRunLoop currentRunLoop] addTimer:timer forMode:NSRunLoopCommonModes];

在NSRunLoop 中添加定时器。

二十一、Bundle identifier 应用标示符

Bundle identifier 是应用的标示符，表明应用和其他APP的区别。

二十二、NSDate 获取几年前的时间

eg. 获取到40年前的日期

NSCalendar *gregorian = [[NSCalendar alloc] initWithCalendarIdentifier:NSGregorianCalendar];
NSDateComponents *dateComponents = [[NSDateComponents alloc] init];
[dateComponents setYear:-40];
self.birthDate = [gregorian dateByAddingComponents:dateComponents toDate:[NSDate date] options:0];

二十三、iOS加载启动图的时候隐藏statusbar

只需需要在info.plist中加入Status bar is initially hidden 设置为YES就好

￼

二十四、iOS 开发，工程中混合使用 ARC 和非ARC

Xcode 项目中我们可以使用 ARC 和非 ARC 的混合模式。

如果你的项目使用的非 ARC 模式，则为 ARC 模式的代码文件加入 -fobjc-arc 标签。如果你的项目使用的是 ARC 模式，则为非 ARC 模式的代码文件加入 -fno-objc-arc 标签。添加标签的方法：

* 打开：你的target -> Build Phases -> Compile Sources.
* 双击对应的 *.m 文件
* 在弹出窗口中输入上面提到的标签 -fobjc-arc / -fno-objc-arc
* 点击 done 保存  
二十五、iOS7 中 boundingRectWithSize:options:attributes:context:计算文本尺寸的使用

之前使用了NSString类的sizeWithFont:constrainedToSize:lineBreakMode:方法，但是该方法已经被iOS7 Deprecated了，而iOS7新出了一个boudingRectWithSize:options:attributes:context方法来代替。而具体怎么使用呢，尤其那个attribute

NSDictionary *attribute = @{NSFontAttributeName: [UIFont systemFontOfSize:13]};
CGSize size = [@"相关NSString" boundingRectWithSize:CGSizeMake(100, 0) options: NSStringDrawingTruncatesLastVisibleLine | NSStringDrawingUsesLineFragmentOrigin | NSStringDrawingUsesFontLeading attributes:attribute context:nil].size;

二十六、NSDate使用 注意

NSDate 在保存数据，传输数据中，一般最好使用UTC时间。
在显示到界面给用户看的时候，需要转换为本地时间。

二十七、在UIViewController中property的一个UIViewController的Present问题

如果在一个UIViewController A中有一个property属性为UIViewController B，实例化后，将BVC.view 添加到主UIViewController A.view上，如果在viewB上进行 - (void)presentViewController:(UIViewController *)viewControllerToPresent animated: (BOOL)flag completion:(void (^)(void))completion NS_AVAILABLE_IOS(5_0);的操作将会出现，“ Presenting view controllers on detached view controllers is discouraged ” 的问题。

以为BVC已经present到AVC中了，所以再一次进行会出现错误。可以使用以下代码解决：

[self.view.window.rootViewController presentViewController:imagePicker
animated:YES
completion:^{
NSLog(@"Finished");
}];

二十八、UITableViewCell indentationLevel 使用

UITableViewCell 属性 NSInteger indentationLevel 的使用， 对cell设置 indentationLevel的值，可以将cell 分级别。
还有 CGFloat indentationWidth; 属性，设置缩进的宽度。
总缩进的宽度: indentationLevel * indentationWidth

二十九、ActivityViewController 使用AirDrop分享

使用AirDrop 进行分享：

NSArray *array = @[@"test1", @"test2"];
UIActivityViewController *activityVC = [[UIActivityViewController alloc] initWithActivityItems:array applicationActivities:nil];
[self presentViewController:activityVC animated:YES
completion:^{
NSLog(@"Air");
}];

就可以弹出界面：

￼

三十、获取CGRect的height

获取CGRect的height， 除了 self.createNewMessageTableView.frame.size.height 这样进行点语法获取。

还可以使用CGRectGetHeight(self.createNewMessageTableView.frame) 进行直接获取。

除了这个方法还有  func CGRectGetWidth(rect: CGRect) -> CGFloat 等等简单地方法。

func CGRectGetMinX(rect: CGRect) -> CGFloat
func CGRectGetMidX(rect: CGRect) -> CGFloat
func CGRectGetMaxX(rect: CGRect) -> CGFloat
func CGRectGetMinY(rect: CGRect) -> CGFloat

三十一、打印 %

NSString *printPercentStr = [NSString stringWithFormat:@"%%"];

三十二、在工程中查看是否使用 IDFA

allentekiMac-mini:JiKaTongGit lihuaxie$ grep -r advertisingIdentifier .
grep: ./ios/Framework/AMapSearchKit.framework/Resources: No such file or directory
Binary file ./ios/Framework/MAMapKit.framework/MAMapKit matches
Binary file ./ios/Framework/MAMapKit.framework/Versions/2.4.1.e00ba6a/MAMapKit matches
Binary file ./ios/Framework/MAMapKit.framework/Versions/Current/MAMapKit matches
Binary file ./ios/JiKaTong.xcodeproj/project.xcworkspace/xcuserdata/lihuaxie.xcuserdatad/UserInterfaceState.xcuserstate matches
allentekiMac-mini:JiKaTongGit lihuaxie$
打开终端，到工程目录中， 输入：
grep -r advertisingIdentifier .
可以看到那些文件中用到了IDFA，如果用到了就会被显示出来。

三十三、APP 屏蔽 触发事件

// Disable user interaction when download finishes
[[UIApplication sharedApplication] beginIgnoringInteractionEvents];




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

