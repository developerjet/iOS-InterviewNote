#iOS-InterviewNote

# iOS面试 - Objective-C

## 分类
1.分类为甚不能添加属性？
这层楼正解，方法可以运行时改变，结构体不能运行时改变，要想改变原结构体，增加一个属性，只能用关联变量的方式，而关联本质是在类的定义之外为类增加额外的存储空间，是一层映射关系

## IOS 基础之nil，NULL，NSNULL区别详解
① nil：一般赋值给空对象。
② NULL：NULL 是一个通用指针（泛型指针）。

## iOS开发runtime机制
> https://www.jianshu.com/p/ea2d0a6fa8d6

## iOS开发信号量
> https://www.jianshu.com/p/2f0225b67f6c

## 讲讲你对atomic & nonatomic的理解：
> https://www.jianshu.com/p/80ac9cfdff5b

1、原子操作对线程安全并无任何安全保证。被atomic修饰的属性(不重载设置器和访问器)只保证了对数据读写的完整性，也就是原子性，但是与对象的线程安全无关。
2、线程安全有保障、对性能有要求的情况下可使用nonatomic替代atomic，当然也可以一直使用atomic。

## KVC
1.KVC实现原理
KVC，键-值编码，使用字符串直接访问对象的属性。
底层实现，当一个对象调用setValue方法时，方法内部会做以下操作：

1.检查是否存在相应key的set方法，如果存在，就调用set方法
2.如果set方法不存在，就会查找与key相同名称并且带下划线的成员属性，如果有，则直接给成员属性赋值
3.如果没有找到_key，就会查找相同名称的属性key，如果有就直接赋值
4.如果还没找到，则调用valueForUndefinedKey：和setValue：forUndefinedKey：方法

## KVO
### KVO的实现原理
KVO-键值观察机制，原理如下：

(1)当给A类添加KVO的时候，runtime动态的生成了一个子类NSKVONotifying_A，让A类的isa指针指向NSKVONotifying_A类，重写class方法，隐藏对象真实类信息
(2)重写监听属性的setter方法，在setter方法内部调用了Foundation 的 _NSSetObjectValueAndNotify 函数
(3)_NSSetObjectValueAndNotify函数内部
a) 首先会调用 willChangeValueForKey
b) 然后给属性赋值
c) 最后调用 didChangeValueForKey
d) 最后调用 observer 的 observeValueForKeyPath 去告诉监听器属性值发生了改变 .

- 一、iOS用什么方式实现对一个对象的KVO? (其实就是在问KVO的本质是什么)
当一个对象添加了KVO监听，iOS系统会通过Runtime动态创建一个子类，并让这个对象的isa指向这个新创建的子类。子类拥有自己的setter方法实现，setter方法内部会顺序调用willChangeValueForKey方法、原来的setter方法、didChangeValueForKey方法，在didChangeValueForKey方法内部又会调用监听器的observeValueForKeyPath监听方法。

- 二、如何手动触发KVO?
答: 手动调用 willChangeValueForKey: 和 didchangeValueForKey: 就可以触发KVO.

- 三、直接修改成员变量会触发KVO吗?
答: 不会. 结合之前说的KVO的本质, 我们知道KVO是通过 setter 方法内部调用 willChangeValueForKey: 和 didchangeValueForKey: 触发的

## 自旋锁和互斥锁

1.互斥锁

定义:当上一个线程的任务没有执行完毕的时候(被锁住),那么下一个线程会进入睡眠状态等待任务执行完毕, 直到上一个执行完成,下一个线程会自动唤醒,然后开始珍惜任务
互斥锁原理：线程会从sleep(加锁)-->running(解锁), 过程中有上下文的切换(主动出让时间片, 线程休眠, 等待下一次唤醒).CPU的抢占,信号的发送等开销.
互斥锁会休眠: 所谓休眠, 即在访问被锁资源时, 调用者线程会休眠, 此时CPU可以调度其他线程工作.直到被锁资源释放锁.此时会唤醒休眠线程.

2.自旋锁

定义:一种用于保护多线程共享资源的锁. 与一般互斥锁(mutex)不同之处在于当自旋锁尝试获取锁时以忙等待(busy waiting)的形式不断地循环检查锁是否可用.当上一个线程的任务没有执行完毕的时候,下一个线程处于一直等待状态,不会休眠,直到上一个执行完毕.
自旋锁原理：线程一直是running(加锁——>解锁), 死循环(忙等 do-while)检测锁的标志位,机制不复杂.
优点:自旋锁不会引起调用者睡眠,所以不会进行线程调度,CPU时间片轮转等耗时操作.如果能在很短的时间内获得锁,自旋锁的效率远高于互斥锁.适用于持有锁较短的程序.
缺点:自旋锁一直占用CPU,在未获得锁的情况下,自旋锁一直运行(忙等状态,询问), 占用着CPU，如果不能在很短的时间内获得锁，这无疑会使CPU效率降低. 自旋锁不能实现递归调用.

## iOS开发模式
### 构造模式：
构造模式用于将某个业务的属性和行为进行分离，当业务属性越多的时候该模式用起来就越方便。比如：我要自定义一个比较灵活的弹窗，这个弹窗有显示和隐藏、动画的功能，并且弹窗的大小、样式显示的位置都要可以自定义。这样我们就可以使用构造模式，将行为和属性分离出来，弹窗的显示和隐藏就是行为，其他的均为属性，这些属性的构造过程中就可以被定义好.

### 享元模式
系统有大量相似对象
这些对象消耗大量内存。
对象的外在状态可以放到外部而轻量化
移除了外在状态后，可以用较少的共享对象替代原来的那组对象
应用程序不依赖于对象标识，因为共享对象不能提供唯一的标识

## 组件化
### 组件化路由方案
1. URLRoute注册方案的优缺点
首先URLRoute也许是借鉴前端Router和系统App内跳转的方式想出来的方法。它通过URL来请求资源。不管是H5，RN，Weex，iOS界面或者组件请求资源的方式就都统一了。URL里面也会带上参数，这样调用什么界面或者组件都可以。所以这种方式是最容易，也是最先可以想到的。

URLRoute的优点很多，最大的优点就是服务器可以动态的控制页面跳转，可以统一处理页面出问题之后的错误处理，可以统一三端，iOS，Android，H5 / RN / Weex 的请求方式。
但是这种方式也需要看不同公司的需求。如果公司里面已经完成了服务器端动态下发的脚手架工具，前端也完成了Native端如果出现错误了，可以随时替换相同业务界面的需求，那么这个时候可能选择URLRoute的几率会更大。
但是如果公司里面H5没有做相关出现问题后能替换的界面，H5开发人员觉得这是给他们增添负担。如果公司也没有完成服务器动态下发路由规则的那套系统，那么公司可能就不会采用URLRoute的方式。因为URLRoute带来的少量动态性，公司是可以用JSPatch来做到。线上出现bug了，可以立即用JSPatch修掉，而不采用URLRoute去做。

所以选择URLRoute这种方案，也要看公司的发展情况和人员分配，技术选型方面。

URLRoute方案也是存在一些缺点的，首先URL的map规则是需要注册的，它们会在load方法里面写。写在load方法里面是会影响App启动速度的。
其次是大量的硬编码。URL链接里面关于组件和页面的名字都是硬编码，参数也都是硬编码。而且每个URL参数字段都必须要一个文档进行维护，这个对于业务开发人员也是一个负担。而且URL短连接散落在整个App四处，维护起来实在有点麻烦，虽然蘑菇街想到了用宏统一管理这些链接，但是还是解决不了硬编码的问题。
真正一个好的路由是在无形当中服务整个App的，是一个无感知的过程，从这一点来说，略有点缺失。
最后一个缺点是，对于传递NSObject的参数，URL是不够友好的，它最多是传递一个字典。

2. Protocol-Class注册方案的优缺点
Protocol-Class方案的优点，这个方案没有硬编码。
Protocol-Class方案也是存在一些缺点的，每个Protocol都要向ModuleManager进行注册。
这种方案ModuleEntry是同时需要依赖ModuleManager和组件里面的页面或者组件两者的。当然ModuleEntry也是会依赖ModuleEntryProtocol的，但是这个依赖是可以去掉的，比如用Runtime的方法NSProtocolFromString，加上硬编码是可以去掉对Protocol的依赖的。但是考虑到硬编码的方式对出现bug，后期维护都是不友好的，所以对Protocol的依赖还是不要去除。
最后一个缺点是组件方法的调用是分散在各处的，没有统一的入口，也就没法做组件不存在时或者出现错误时的统一处理。

3. Target-Action方案的优缺点
Target-Action方案的优点，充分的利用Runtime的特性，无需注册这一步。Target-Action方案只有存在组件依赖Mediator这一层依赖关系。在Mediator中维护针对Mediator的Category，每个category对应一个Target，Categroy中的方法对应Action场景。Target-Action方案也统一了所有组件间调用入口。
Target-Action方案也能有一定的安全保证，它对url中进行Native前缀进行验证。
Target-Action方案的缺点，Target_Action在Category中将常规参数打包成字典，在Target处再把字典拆包成常规参数，这就造成了一部分的硬编码。

## iOS包活方案
### 1.iOS - 后台保活(后台持续运行代码)
iOS有两种后台运行保活方式，第一种叫无声音乐保活（即在后台开启音频播放，只不过不需要播放出音量且不能影响其他音乐播发软件），第二种叫Background Task，但是这种方法在iOS 13以后只能申请短短的30秒钟时间，但是在iOS7-iOS13以前是可以申请到3分钟的保活时间的，当然我们也可以经过处理来申请到更多的保活时间。

- 无声音乐保活
1.打开应用的Target页面Signing & Cabailities，添加Capability（Background Modes）勾选Audio，AirPlay，and Picture in Picture选项
2.在info.plist文件的【Required background modes】中，添加对应的key：【App plays audio or streams audio/video using AirPlay】
3.在AppDelegate.m中添加监听
UIApplicationWillEnterForegroundNotification（应用进入前台通知）UIApplicationDidEnterBackgroundNotification（应用进入后台通知）

```
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(appWillEnterForeground) name:UIApplicationWillEnterForegroundNotification object:nil];

[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(appDidEnterBackground) name:UIApplicationDidEnterBackgroundNotification object:nil];
```

4.编写音乐播放类
- .h文件
```
#import <Foundation/Foundation.h>
#import <AVFoundation/AVFoundation.h>

@interface BackgroundPlayer : NSObject <AVAudioPlayerDelegate>
{
    AVAudioPlayer* _player;
}

- (void)startPlayer;

- (void)stopPlayer;

@end

```
- .m文件

```

#import "BackgroundPlayer.h"

@implementation BackgroundPlayer

- (void)startPlayer
{
    if (_player && [_player isPlaying]) {
        return;
    }
    AVAudioSession *session = [AVAudioSession sharedInstance];
    [[AVAudioSession sharedInstance] setMode:AVAudioSessionModeDefault error:nil];

    NSString* route = [[[[[AVAudioSession sharedInstance] currentRoute] outputs] objectAtIndex:0] portType];

    if ([route isEqualToString:AVAudioSessionPortHeadphones] || [route isEqualToString:AVAudioSessionPortBluetoothA2DP] || [route isEqualToString:AVAudioSessionPortBluetoothLE] || [route isEqualToString:AVAudioSessionPortBluetoothHFP]) {
        if (@available(iOS 10.0, *)) {
            [[AVAudioSession sharedInstance] setCategory:AVAudioSessionCategoryPlayAndRecord
                                             withOptions:(AVAudioSessionCategoryOptionMixWithOthers | AVAudioSessionCategoryOptionAllowBluetooth | AVAudioSessionCategoryOptionAllowBluetoothA2DP)
                                                   error:nil];
        } else {
            // Fallback on earlier versions
        }
    }else{
        [[AVAudioSession sharedInstance] setCategory:AVAudioSessionCategoryPlayAndRecord
                                         withOptions:(AVAudioSessionCategoryOptionMixWithOthers | AVAudioSessionCategoryOptionDefaultToSpeaker)
                                               error:nil];
    }

    [session setActive:YES error:nil];

    NSString *path = [[NSBundle mainBundle] pathForResource:@"music" ofType:@"wav"];
    NSURL *url = [NSURL fileURLWithPath:path isDirectory:NO];

    _player = [[AVAudioPlayer alloc] initWithContentsOfURL:url error:nil];
    [_player prepareToPlay];
    [_player setDelegate:self];
    _player.numberOfLoops = -1;
    BOOL ret = [_player play];
    if (!ret) {
        NSLog(@"play failed,please turn on audio background mode");
    }
}

- (void)stopPlayer
{
    if (_player) {
        [_player stop];
        _player = nil;
        AVAudioSession *session = [AVAudioSession sharedInstance];
        [session setActive:NO error:nil];
        NSLog(@"stop in play background success");
    }
}

@end

```

- 将使用到的音乐文件放入项目：
5.在应用进入后台时开启保活
在AppDelegate.m文件中，编写如下代码
```
// 语音播报
#import <MediaPlayer/MediaPlayer.h>
#import <AVFoundation/AVFoundation.h>

@property (nonatomic, strong) BackgroundPlayer* player;
- (void)appWillEnterForeground {
    if (self.player) {
        [self.player stopPlayBackgroundAlive];
    }
}

- (void)appDidEnterBackground {
    if (_player == nil) {
        _player = [[BackgroundPlayer alloc] init];
    }
    [self.player startPlayer];
}

```

## Background Task保活
1.在info.plist文件的【Required background modes】中，添加对应的key：【App registers for location updates】
2.同样我们需要监听
UIApplicationWillEnterForegroundNotification（应用进入前台通知）和UIApplicationDidEnterBackgroundNotification（应用进入后台通知）
在AppDelegate.m文件中编写如下代码：
```
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(appWillEnterForeground) name:UIApplicationWillEnterForegroundNotification object:nil];
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(appDidEnterBackground) name:UIApplicationDidEnterBackgroundNotification object:nil];
- (void)appWillEnterForeground {}

- (void)appDidEnterBackground {}
```

3.使用Background Task申请保活时间，在应用进入后台时开启保活，在应用进入前台时关闭保活：
在AppDelegate.m文件中编写如下代码：

```
@property (assign, nonatomic) UIBackgroundTaskIdentifier backgroundId;
- (void)appWillEnterForeground {
   [self stopKeepAlive];
}

- (void)appDidEnterBackground {
    _backgroundId = [[UIApplication sharedApplication] beginBackgroundTaskWithExpirationHandler:^{
        //申请的时间即将到时回调该方法
        NSLog(@"BackgroundTask time gone");
        [self stopKeepAlive];
    }];
}

- (void)stopKeepAlive{
  if (_backgroundId) {
        [[UIApplication sharedApplication] endBackgroundTask:_backgroundId];
        _backgroundId = UIBackgroundTaskInvalid;
    }
}
```
4.使用NSTimer循环申请保活时间，但是建议不要无限申请保活时间，因为系统如果发现该应用一直在后台运行时，是可能会直接crash掉你的应用的 ，错误码0x8badf00d.

将需要在后台一直运行的代码，写在AppDelegate.m中运行：
例如：像后台发送手机定位：
```
// 高德地图
#import <AMapFoundationKit/AMapFoundationKit.h>
#import <AMapLocationKit/AMapLocationKit.h>
//定位
@property (nonatomic,strong) AMapLocationManager *locationManager;
//注册定位信息
@property (nonatomic,strong) NSMutableDictionary * locationDic;

// 定位
    self.locationDic = [@{} mutableCopy];
    [self locationPresent];
    //开启定时器（后台每五分钟上传一次定位）
    [self StartTimer];

// 开启倒计时效果
- (void)StartTimer{
    __block NSInteger time = 10; //倒计时时间
    dispatch_queue_t queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
    dispatch_source_t timer = dispatch_source_create(DISPATCH_SOURCE_TYPE_TIMER, 0, 0, queue);
    dispatch_source_set_timer(timer,DISPATCH_TIME_NOW,1.0*NSEC_PER_SEC, 0);//每秒执行
    dispatch_source_set_event_handler(timer, ^{
        if(time <= 0){ //倒计时结束，关闭
            dispatch_source_cancel(timer);
            dispatch_async(dispatch_get_main_queue(), ^{
                [self StartTimer];
                //上传定位信息
                [self locationPresent];

            });
        }else{
            dispatch_async(dispatch_get_main_queue(), ^{

            });
            time--;
        }
    });
    dispatch_resume(timer);
}

//定位
-(void)locationPresent{

    self.locationManager = [[AMapLocationManager alloc] init];

    [self.locationManager setDelegate:self];

    @try {
       self.locationManager.allowsBackgroundLocationUpdates = YES;
    } @catch (NSException *exception) {
        NSLog(@"异常：%@", exception);
    } @finally {


    }

    self.locationManager.desiredAccuracy = kCLLocationAccuracyBest;
    self.locationManager.distanceFilter = kCLDistanceFilterNone;


    // 带逆地理信息的一次定位（返回坐标和地址信息）
    [self.locationManager setDesiredAccuracy:kCLLocationAccuracyHundredMeters];
    // 设置不允许系统暂停定位
    [self.locationManager setPausesLocationUpdatesAutomatically:NO];
    // 定位超时时间，最低5s，此处设置为5s
    self.locationManager.locationTimeout = 5;
    // 逆地理请求超时时间，最低5s，此处设置为5s
    self.locationManager.reGeocodeTimeout = 5;


    [self.locationManager requestLocationWithReGeocode:YES completionBlock:^(CLLocation *location, AMapLocationReGeocode *regeocode, NSError *error){
        if (error){
            LLLog(@"%@",error);
            NSLog(@"locError:{%ld - %@};", (long)error.code, error.localizedDescription);
            if (error.code == AMapLocationErrorLocateFailed){
                return;
            }
        }

        NSLog(@"location:%@", location);
        if (location) {
            [self.locationDic setValue:@(location.coordinate.latitude) forKey:@"latitude"];
            [self.locationDic setValue:@(location.coordinate.longitude) forKey:@"longitude"];
            // 上传定位信息
            [self UpdateLocationPlace];
        }

    }];
}


//上传当前位置
-(void)UpdateLocationPlace{

    NSString *Url = [NSString stringWithFormat:@"%@%@",BaseUrl,UploadForDriver];
    LLLog(@"UpdateLocationPlace===%@===%@",Url,self.locationDic);
    [SendHttpRequest PostNSMutableDictionary:self.locationDic withNsstring:Url result:^(NSDictionary *result, NSError *error) {
        LLLog(@"%@",result);

    }];
}

```

## iOS开发响应链
- 响应链
顾名思义，响应链就是对某些行为的响应，按照一定的机制进行流动，所形成的链条

在iOS中：用户点击屏幕时，会触发以下流程：

捕获事件：手机系统捕获到单击行为，也称为单击事件
封装事件：把包含该事件的信息封装成UITouch和UIEvent对象
查找程序：找到可以处理该事件的运行程序
查找第一响应者：利用hitTest:(CGPoint)point方法，在视图层级中，从下往上逐级查找可以响应该事件的响应者，将最后的响应者作为第一响应者
事件转发：如果第一响应者不处理该事件，再用nextResponder方法，从上往下，逐级转发，直到没有响应者响应

## iOS签名机制

apple后台就是apple的服务器也会生成一对公钥私钥，私钥保存在apple的后台，公钥在每一天iOS设备上，相当于每一台iOS设备都有一个相同的苹果公钥

第一步。app（mach-o可执行文件也就是项目的源代码，第二个包含图片资源，mp3资源，视频资源，storyBoard，xib等等这些资源）这些东西最终都会变成一个ipa包装到我们手机上。
app文件经过mac电脑的私钥进行签名生成两个文件 ，一个是签名（这个签名是把你的app的内容 生成一个散列值用mac私钥进行加密）然后加上你原来的app文件
第二步需要做一个生成证书的操作，证书是CA利用自己的私钥对你的公钥进行签名，这里面apple就是一个CA认证机构，apple这个CA对你的mac公钥进行一个签名然后生成一个证书，这个证书就是对mac公钥进行一个签名，证书里面也是包含两个文件一个是mac公钥原文件，一个是用apple私钥加密的签名，生成证书以后他会加上设备ID(devices)，app id，entitlements（就是一些权限）。
第三步就是他拿到第二步生成的证书还有设备ID(devices)，app id，entitlements这些经过apple的私钥在进行一个签名，这里面生成的签名就是步骤2里面的证书文件加上设备ID(devices)，app id，entitlements，生成的一个签名文件，当然还包含原来的设备ID(devices)，app id，entitlements，那么这个签名加上原来那一堆设备ID(devices)，app id，entitlements和证书就生成一个moblieProvision文件
第四步，经过这一系列操作以后相当于你的ipa里面（也就是第一步说的ipa文件）包含了app文件（代码mach-o，视频，图片等等），还有这些app数据文件对应的一个签名，再加上一个moblieProvision文件

## iOS开发AFNetworking源码解读:
> https://www.jianshu.com/p/f6741cf9c7a8

### AFNetworking 底层原理分析：

AFNetworking是封装的NSURLSession的网络请求，由五个模块组成：分别由NSURLSession,Security,Reachability,Serialization,UIKit五部分组成

NSURLSession：网络通信模块（核心模块） 对应 AFNetworking中的 AFURLSessionManager和对HTTP协议进行特化处理的AFHTTPSessionManager,AFHTTPSessionManager是继承于AFURLSessionmanager的

Security：网络通讯安全策略模块  对应 AFSecurityPolicy

Reachability：网络状态监听模块 对应AFNetworkReachabilityManager

Seriaalization：网络通信信息序列化、反序列化模块 对应 AFURLResponseSerialization

UIKit：对于iOS UIKit的扩展库

## +load和initialize
对于OC中的类来说，在runtime中会有两个方法被调用：

+load
+initialize
这两个方法看起来都是在类初始的时候调用的，但其实还是有一些异同，从而可以用来做一些行为。

+load
首先，load方法是一定会在runtime中被调用的，只要类被添加到runtime中了，就会调用load方法，所以我们可以自己实现laod方法来在这个时候执行一些行为。

而且有意思的一点是，load方法不会覆盖。也就是说，如果子类实现了load方法，那么会先调用父类的load方法，然后又去执行子类的load方法。同样的，如果分类实现了load方法，也会先执行主类的load方法，然后又会去执行分类的load方法。所以父类的load会执行很多次，这一点需要注意。而且执行顺序是 类 -> 子类 ->分类。而不同类之间的顺序不一定。

+initialize
与load不同的是，initialize方法不一定会执行。只有当一个类第一次被发送消息的时候会执行，注意是第一次。什么叫发送消息呢，就是执行类的一些方法的时候。也就是说这个方法是懒加载，没有用到这个类就不会调用，可以节省系统资源。

还有一点截然相反，却更符合我们预期的就是，initialize方法会覆盖。也就是说如果子类实现了initialize方法，就不会执行父类的了，直接执行子类本身的。如果分类实现了initialize方法，也不会再执行主类的。所以initialize方法的执行覆盖顺序是 分类 -> 子类 ->类。且只会有一个initialize方法被执行

## iOS野指针、僵尸对象、空指针
1.野指针：指向引用计数为0被释放，却没有销毁的对象

2.僵尸对象：一个已经被释放却没有被销毁的对象
其内存已经被系统回收，是不稳定对象，不可以再访问或者使用，因为它的内存是随时可能被别的对象申请而占用的
当使用野指针访问僵尸对象是，可能报EXC_BAD_ACCESS (code=EXC_I386_GPFLT)错误
如何检测僵尸对象：
1、Xcode-开启zombie Objects
不建议默认开启，因为一旦开启，每次通过指针访问对象的时候，都会去检查指针指向的对象是否为僵尸对象。会影响程序的执行效率
2、Xcode的Analyze静态分析
如何避免僵尸对象报错：
将指针的值置为nil

3.空指针：是指没有指向任何东西的指针（存储的东西是nil、NULL、0），给空指针发送消息不会报错


# iOS面试 - Swift

## dynamic framework 和 static framework 的区别是什么
可参考该文章

静态库和动态库, 静态库是每一个程序单独打包一份, 而动态库则是多个程序之间共享

静态库和动态库是相对编译期和运行期的：静态库在程序编译时会被链接到目标代码中，程序运行时将不再需要改静态库；而动态库在程序编译时并不会被链接到目标代码中，只是在程序运行时才被载入，因为在程序运行期间还需要动态库的存在。

静态库 好处：

模块化，分工合作，提高了代码的复用及核心技术的保密程度
避免少量改动经常导致大量的重复编译连接
也可以重用，注意不是共享使用
动态库 好处：

使用动态库，可以将最终可执行文件体积缩小，将整个应用程序分模块，团队合作，进行分工，影响比较小
使用动态库，多个应用程序共享内存中得同一份库文件，节省资源
使用动态库，可以不重新编译连接可执行程序的前提下，更新动态库文件达到更新应用程序的目的。
不同点：

静态库在链接时，会被完整的复制到可执行文件中，如果多个App都使用了同一个静态库，那么每个App都会拷贝一份，缺点是浪费内存。类似于定义一个基本变量，使用该基本变量是是新复制了一份数据，而不是原来定义的；
动态库不会复制，只有一份，程序运行时动态加载到内存中，系统只会加载一次，多个程序共用一份，节约了内存。类似于使用变量的内存地址一样，使用的是同一个变量；
共同点：

静态库和动态库都是闭源库，只能拿来满足某个功能的使用，不会暴露内部具体的代码信息


## Swift 属性观察者
willSet 会在该值被存储之前被调用
didSet 会在一个新值被存储后被调用

如果你实现了一个 willSet 观察者，新的属性值会以常量形式参数传递。你可以在你的 willSet 实现中为这个参数定义名字。如果你没有为它命名，那么它会使用默认的名 newValue
如果你实现了一个 didSet观察者，一个包含旧属性值的常量形式参数将会被传递。你可以为 它命名，也可以使用默认的形式参数名 oldValue 。如果你在属性自己的 didSet 观察者里给 自己赋值，你赋值的新值就会取代刚刚设置的值

## Swift 泛型：

Swift 提供了泛型让你写出灵活且可重用的函数和类型。
Swift 标准库是通过泛型代码构建出来的。
Swift 的数组和字典类型都是泛型集。
你可以创建一个Int数组，也可创建一个String数组，或者甚至于可以是任何其他 Swift 的类型数据数组。
以下实例是一个非泛型函数 exchange 用来交换两个 Int 值：

## Swift尾随闭包

如果您需要将一个很长的闭包表达式作为最后一个参数传递给函数，可以使用尾随闭包来增强函数的可读性。 尾随闭包是一个书写在函数括号之后的闭包表达式，函数支持将其作为最后一个参数调用。

```
func someFunctionThatTakesAClosure(closure: () -> ()) {
    // 函数体部分
}

// 以下是不使用尾随闭包进行函数调用
someFunctionThatTakesAClosure({
    // 闭包主体部分
})

// 以下是使用尾随闭包进行函数调用
someFunctionThatTakesAClosure() {
  // 闭包主体部分
}
```

## Swift 元组
元组（tuples）把多个值组合成⼀个复合值。元组内的值可以是任意类型，并不要求是相同类
型。

## OC协议与Swift协议的区别

- OC中的协议：
1、受限于委托代理的含义，多⽤于不同类之间的传值与回调。

- Swift的协议：
1、可以通过协议 (extension) 扩展，实现协议的⽅法（OC不⾏）
2、定义属性⽅法
3、通过抽取不同类中的相同⽅法和属性，实现模块化减少耦合。使面向协议编程成为可能
4、不需要单独声明协议对象和指定代理
5、协议可以继承其他协议

## 区块链
### 什么是ERC-20和ERC-721

1、ERC-20

ERC-20是最广为人知的标准，ERC-20标准里没有价值的区别，token之间是可以互换的。这就相当于说在ERC-20标准下，你的100块“钱”和我的100块“钱”是一样的。

ERC-20标准里规定了Token需要有它的名字、符号、总供应量以及包含转账、汇款等其他功能。这个标准带来的好处是：只要Token符合ERC-20标准，那么它将兼容以太坊钱包。也就是说，你可以在你的以太坊钱包里加入这个Token，还可以通过钱包把它发送给别人。

正因为ERC-20标准的存在，使得发行Token变得很简单。目前，以太坊上ERC-20 Token的数量超过了180000种。

2、ERC-721

既然ERC-20那么厉害，为什么还要多出一个ERC-721标准呢？前面提到ERC-20标准的Token没有价值的区别，那对于一些需要有独一无二属性的资产（比如加密收藏品、游戏道具）便不再适用。

ERC-721标准规定了符合它这种标准的每个Token都有唯一的Token ID。在ERC-721标准里，每个Token都是独一无二的。也就是说，在ERC-721标准下，你的100块“钱”和我的100块“钱”是不一样的，因为这两张100块钱的编号是不一样的。

