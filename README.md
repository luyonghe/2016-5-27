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
# 2016-7-27
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



