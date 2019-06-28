# 谈谈关于 iOS 的架构以及应用
&emsp;&emsp;一直以来想写一篇文章，但是没找到合适的主题，前段时间一直在看 [Flutter](https://flutter.dev/) 的一些东西，本来有意向想写关于 Flutter 的一些总结，但是看的有些零零散散，并且没有实际应用过，所以也就搁置了。正好最近一段时间除主业务之余，一直在做我们 [甘草医生](https://www.igancao.com/) 用户端的重构，刚好有一些对于 iOS 架构方面的看法与感悟，在这里与大家分享。
万事开头难！其实在开始重构之前，我是很纠结的，一直很难开始。我也曾翻阅过很多资料，想找到一个合适的符合我们自己目前业务的架构，最后做了种种的比对与测试，选择了 `MVVM + 组件化 + AOP` 的模式来重构。可能有人会疑问，你为什么选择这样的架构模式？使用这些模式有什么好处？这些抽象的模式概念具体应该怎么在实际项目中运用？OK，那我们就带着这些疑问一步步往下看。
## 一、关于架构模式
&emsp;&emsp;我们先来了解一下在 iOS 中常用的一些架构模式
##### 1. [MVC](https://zh.wikipedia.org/wiki/MVC)
&ensp;&ensp;&ensp;&ensp;关于 MVC（Model-View-Controller）这个设计模式我相信稍有些编程经验的人都了解至少听说过，作为应用最为广泛的架构模式，大家应该都是耳熟能详了，但是不同的人对 MVC 的理解是不同的。在 iOS 中，Cocoa Touch 框架使用的就是 MVC ，如下

<p align='center'>
<img src='https://github.com/loveway/LearnBlog/blob/master/Notes/Objective-C/image/ar_mvc.png'>
</p>

这是苹果典型的 MVC 模式，用户通过 View 将交互（点击、滑动等）通知给 Controller，Controller 收到通知后更新 Model，Model 状态改变以后再通知 Controller 来改变他们负责的 View。由于在 iOS 中我们常用的 UIViewController 本身就自带一个 View，所以在 iOS 开发中 Controller  层和 View 层总是紧密的耦合在一起，如果一个页面业务逻辑量大的话，一个视图控制器经常会很多行的代码，导致视图控制器非常的臃肿。
可见，MVC 模式虽然能带来简单的业务分层，但是想必各位使用 MVC 模式的 iOSer 们经常会被以下几个问题困扰
 1. 厚重的 ViewController
    在日常的处理中，我们一般将我们的一些网络请求、数据存储、视图逻辑等一些处理全部扔在我们的 ViewController 里，在业务量大的情况下，一个 ViewController 里面就会有几千行代码
 2. 较差的可测试性 
    对一个有几千甚至上万行的 ViewController 进行单元测试是一个非常难以接受的事情，可以说，谁接到这个任务都是难以接受的
 3. 较差的可读性 
    我相信大家都有接手一个项目然后改 bug 的经历，当你看到一个有 10000 行的代码的 ViewController 的时候，你肯定吐槽过
    
##### 2. [MVVM](https://zh.wikipedia.org/wiki/MVVM)
&emsp;&emsp;MVVM （Model-View-ViewModel），其实也是基于 MVC 的。上面我们说的 MVC 臃肿的问题，在 MVVM 的架构模式中得到了解决，我们一些常用的网络请求、数据存储等都交给它处理，这样就可以分离出 ViewController 里面的一些代码使其“减肥”。

<p align='center'>
<img src='https://github.com/loveway/LearnBlog/blob/master/Notes/Objective-C/image/ar_mvvm.gif'>
</p>

如图，就是 MVC 到 MVVM 的演变过程，在 MVVM 中 V 包含 View 和 ViewController，可以看出来 MVVM 其实就是把 MVC 中的 C 分离出来一个 ViewModel 用来做一些数据加工的事情。在上面 MVC 模式中讲了，一个 ViewController 经常会有很多东西要处理，数据加工、网络请求等，现在都可以交给 ViewModel 去做了。这样，Controller 就可以实现“减肥”，而更加专注于自己的数据调配的工作，绑定 ViewModel 和 View 的关系

<p align='center'>
<img src='https://github.com/loveway/LearnBlog/blob/master/Notes/Objective-C/image/ar_mvvm2.png'>
</p>

可以看出 MVVM 的模式解决了 MVC 模式中的一些问题，使得 ViewController 代码量减少、使得可读性变高、代码单元测试变得简单。但是 MVVM 也有其一些缺陷，比如由于 ViewModel 和 View 的绑定，那么出现了 bug 第一时间定位不到 bug 的位置，有可能是 View 层的 bug 传到了 Model 层。还有一点就是对于较大的工程的项目，数据的绑定和转换需要较大的成本。关于其缺点以及可行的解决方式，在 [Casa Taloyum](https://casatwy.com/) 的 [iOS应用架构谈 网络层设计方案](https://casatwy.com/iosying-yong-jia-gou-tan-wang-luo-ceng-she-ji-fang-an.html) 已经说明的比较详细，有兴趣的童鞋可以去看一下，几篇关于架构方面的文章都很值得一读。

##### 3. 其他的一些架构模式
&emsp;&emsp;还有一些其他的架构模式，比如 MVP（Model-View-Presenter）、VIPER（View-Interactor-Presenter-Entity-Routing）、MVCS（Model-View-Controller-Store）等，其实都是基于 MVC 思想派生出来的一些架构模式，基本都是为了给 Controller 减负而生的，所以还是那句话，万变不离 MVC ！

## 二、架构模式的选用
&emsp;&emsp;了解到每个架构模式的优缺点之后，这里，我决定用 MVVM 的架构模式来重构我们的 APP。那么说到 MVVM ，我们就肯定是要提到 RAC ，也就是 [ReactiveCocoa](https://github.com/ReactiveCocoa/ReactiveCocoa)，它是一个响应式编程的框架，可以使每层交互起来更加方便清晰。当然， RAC 肯定不是实现数据绑定的唯一方案，在 iOS 中比如 KVO、Notification、Delegate、Block等都可以实现，只不过是 RAC 的实现更加优雅一些，所以我们经常会采用 RAC 来实现数据的绑定。关于 RAC ，下面一张图很清晰的解释了它的思想，也就是 [FRP](https://zh.wikipedia.org/wiki/%E5%87%BD%E6%95%B0%E5%BC%8F%E5%8F%8D%E5%BA%94%E5%BC%8F%E7%BC%96%E7%A8%8B)（Function Reactive Programming）函数响应式编程

<p align='center'>
<img src='https://github.com/loveway/LearnBlog/blob/master/Notes/Objective-C/image/ar_frp.png'>
</p>

上图可以看到 c 根据 a 和 b 的值变化的过程。举个例子，我们一般在登录的时候，会限制输入手机号的长度，那么按着以往的做法，就是实现 UITextField 的代理，监听输入文字的变化，如下

```objc
//1、导入代理
<UITextFieldDelegate>
//2、设置代理
self.phoneTextField.delegate = self;
//3、实现代理
- (void)textFieldDidChange:(UITextField *)textField {

        if (textField == self.phoneTextField) {
            if (textField.text.length > 11) {
                textField.text = [textField.text substringToIndex:11];
            }
        }
}
```
那么如果使用 RAC ，如下
```objc
    @weakify(self);
    [[self.phoneTextField.rac_textSignal map:^id _Nullable(NSString * _Nullable value) {
        return value.length > 11 ? [value substringToIndex:11] : value;
    }] subscribeNext:^(NSString *x) {
        @strongify(self);
        self.phoneTextField.text = x;
    }];
```
可以看出代码变得更加清晰了，我们只需要实现对 `phoneTextField` 信号的监听，就可以实现了。我们再来看一个例子，比如在我们用户端的登录界面，如下图

<p align='center'>
<img src='https://github.com/loveway/LearnBlog/blob/master/Notes/Objective-C/image/ar_login.gif'>
</p>

按着正常的逻辑就是用户输入 11 位手机号码后再输入密码才能使其登录，这个时候我们的登录按钮才能点击，要想实现这个逻辑，正常的做法应该如下，
```objc
//1、导入代理
<UITextFieldDelegate>
//2、设置代理
self.phoneTextField.delegate = self;
self.passwordTextField.delegate = self;
//3、实现代理
- (void)textFieldDidChange:(UITextField *)textField {

        if (self.phoneTextField.text.length == 11 && [self.passwordTextField isNotBlank]) {
             self.loginButton.enabled = YES;
        } else {
             self.loginButton.enabled = NO;
        }
}
```
而使用 RAC 则如下
```objc
    @weakify(self);
    [[[RACSignal combineLatest:@[self.phoneTextField.rac_textSignal,
                                 self.passwordTextField.rac_textSignal]] map:^id _Nullable(RACTuple * _Nullable value) {
        
        RACTupleUnpack(NSString *phone, NSString *password) = value;
        return @([password isNotBlank] && phone.length == 11);
        
    }] subscribeNext:^(NSNumber *x) {
        @strongify(self);
        if (x.boolValue) {
            self.loginButton.enabled = YES;
        } else {
            self.loginButton.enabled = NO;
        }
    }];
```
我们将 `self.phoneTextField.rac_textSignal` 和 `self.passwordTextField.rac_textSignal` 这两个信号合并成一个信号并且监听，实现一定的逻辑，简单明了。当然， RAC 的好处远远不止这些，这里只是冰山一角，有兴趣的可以去自己用一用这个库，体验更多的功能，这里也就不多赘述了。

## 三、关于组件化
&emsp;&emsp;组件化这个概念相信大家都听说过，使用组件化的好处就是使我们项目更好的解耦，降低各个分层之间的耦合度，使项目始终保持着 `高聚合，低耦合` 的特点。举个简单的例子，在 iOS 中页面之间的跳转，两个开发人员负责开发两个页面，小 A 负责开发的 AViewController 已经开发完毕，然后需要点击按钮跳到小 B 负责的 BViewController，并且需要传一个值，如下
```objc
//1、导入BViewController
#import "BViewController"
//2、跳转
BViewController *bViewController = [[BViewController alloc]init];
bViewController.uid = @"123";
[self.navigationController pushViewController:bViewController animated:YES];
```
这时候小 A 已经准备去写其他业务了，但是一问才发现小 B 并没有开始写 BViewController，还需要一段时间才能写，那么小 A 就郁闷了，要么就等着小 B 写完我再去做其他的，要么就先注释我这段代码，等到小 B 写完我再解注释。造成这种情况的原因就是因为两个页面之间紧紧地耦合在一起了，在开发人员少或者独立开发的情况下我们经常使用这种方式进行页面间的跳转和传值，页面基本都是一个人负责，所以感觉不到问题，试想一下在几十人开发的工作组中，划分很细的情况下，你自己的脱节是不是给别人带去了不必要的麻烦。我相信这是所有人都不想发生的，那么我们就需要对页面进行组件化解耦，这里我所使用的组件化方案是 `target-action` 方式，使用的是 [Casa Taloyum](https://casatwy.com/) 的 [CTMediator](https://github.com/casatwy/CTMediator)，其主要的思想就是通过一个中间者来提供服务，通过 `runtime`来调用组件服务，比如以前的依赖关系如下

<p align='center'>
<img src='https://github.com/loveway/LearnBlog/blob/master/Notes/Objective-C/image/ar_unmodularization.png'>
</p>

那么使用 CTMediator 实现组件化以后，各组件之间的依赖关系变成下图

<p align='center'>
<img src='https://github.com/loveway/LearnBlog/blob/master/Notes/Objective-C/image/ar_ modularization.png'>
</p>

这样各模块之间就实现了解耦，模块之间的通信就全部通过中间层来进行。我们回过头来再看之前的小 A 和小 B，如果使用这种方式，那么小 A 的跳转代码应该如下
```objc
//1、导入Mediator
#import "CTMediator+BViewControllerActions.h"
//2、跳转
UIViewController *viewController = [[CTMediator sharedInstance] gc_bViewController:@{@"uid": @"123"}];
[self.navigationController pushViewController:viewController animated:YES complete:nil];
```
这样小 A 就不用管小 B 是不是写完没，也不需要导入小 B 的页面，就可以跳转到小 B 的页面，实现了页面间的解耦。能达到这一目的的功臣就是我们的中间者，我们来看看它做了什么，我们还是以我们登录页面为例，我们从登陆跳转到注册页面的代码如下
```objc
//1、导入Mediator
#import "CTMediator+RegistViewControllerActions.h"
//2、跳转
UIViewController *viewController = [[CTMediator sharedInstance] gc_registViewController];
[self.navigationController pushViewController:viewController animated:YES complete:nil];
```
其中 `CTMediator+RegistViewControllerActions.h` 中的代码如下
```objc
//
//  CTMediator+RegistViewControllerActions.m
//  GCUser
//
//  Created by HenryCheng on 2019/4/15.
//  Copyright © 2019 HenryCheng. All rights reserved.
//
#import "CTMediator+RegistViewControllerActions.h"
NSString *const gc_targetRegistVC = @"RegistViewController";
NSString *const gc_actionRegistVC = @"registViewController";
@implementation CTMediator (RegistViewControllerActions)
- (UIViewController *)gc_registViewController {
        UIViewController *viewController = [self performTarget:gc_targetRegistVC
                                                        action:gc_actionRegistVC
                                                        params:@{@"title": @"注册"}
                                             shouldCacheTarget:NO
                                            ];
        if ([viewController isKindOfClass:[UIViewController class]]) {
            return viewController;
        } else {
            return [[UIViewController alloc] init];
        }
}
@end
```
其中重要的就是 ` performTarget:action:params:shouldCacheTarget:` 这个方法，内部的实现方式如下
```objc
- (id)performTarget:(NSString *)targetName action:(NSString *)actionName params:(NSDictionary *)params shouldCacheTarget:(BOOL)shouldCacheTarget {
        NSString *swiftModuleName = params[kCTMediatorParamsKeySwiftTargetModuleName];
        // generate target
        NSString *targetClassString = nil;
        if (swiftModuleName.length > 0) {
            targetClassString = [NSString stringWithFormat:@"%@.Target_%@", swiftModuleName, targetName];
        } else {
            targetClassString = [NSString stringWithFormat:@"Target_%@", targetName];
        }
        NSObject *target = self.cachedTarget[targetClassString];
        if (target == nil) {
            Class targetClass = NSClassFromString(targetClassString);
            target = [[targetClass alloc] init];
        }
        // generate action
        NSString *actionString = [NSString stringWithFormat:@"Action_%@:", actionName];
        SEL action = NSSelectorFromString(actionString);
        
        if (target == nil) {
            // 这里是处理无响应请求的地方之一，这个demo做得比较简单，如果没有可以响应的target，就直接return了。实际开发过程中是可以事先给一个固定的target专门用于在这个时候顶上，然后处理这种请求的
            [self NoTargetActionResponseWithTargetString:targetClassString selectorString:actionString originParams:params];
            return nil;
        }
        
        if (shouldCacheTarget) {
            self.cachedTarget[targetClassString] = target;
        }
    
        if ([target respondsToSelector:action]) {
            return [self safePerformAction:action target:target params:params];
        } else {
            // 这里是处理无响应请求的地方，如果无响应，则尝试调用对应target的notFound方法统一处理
            SEL action = NSSelectorFromString(@"notFound:");
            if ([target respondsToSelector:action]) {
                return [self safePerformAction:action target:target params:params];
            } else {
                // 这里也是处理无响应请求的地方，在notFound都没有的时候，这个demo是直接return了。实际开发过程中，可以用前面提到的固定的target顶上的。
                [self NoTargetActionResponseWithTargetString:targetClassString selectorString:actionString originParams:params];
                [self.cachedTarget removeObjectForKey:targetClassString];
                return nil;
            }
        }
}
```
可以看到如果有响应则调用 `safePerformAction:target: params:` 这个方法，如下
```objc
- (id)safePerformAction:(SEL)action target:(NSObject *)target params:(NSDictionary *)params {
        NSMethodSignature* methodSig = [target methodSignatureForSelector:action];
        if(methodSig == nil) {
            return nil;
        }
        const char* retType = [methodSig methodReturnType];
    
        if (strcmp(retType, @encode(void)) == 0) {
            NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:methodSig];
            [invocation setArgument:&params atIndex:2];
            [invocation setSelector:action];
            [invocation setTarget:target];
            [invocation invoke];
            return nil;
        }
    
        if (strcmp(retType, @encode(NSInteger)) == 0) {
            NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:methodSig];
            [invocation setArgument:&params atIndex:2];
            [invocation setSelector:action];
            [invocation setTarget:target];
            [invocation invoke];
            NSInteger result = 0;
            [invocation getReturnValue:&result];
            return @(result);
        }
    
        if (strcmp(retType, @encode(BOOL)) == 0) {
            NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:methodSig];
            [invocation setArgument:&params atIndex:2];
            [invocation setSelector:action];
            [invocation setTarget:target];
            [invocation invoke];
            BOOL result = 0;
            [invocation getReturnValue:&result];
            return @(result);
        }
    
        if (strcmp(retType, @encode(CGFloat)) == 0) {
            NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:methodSig];
            [invocation setArgument:&params atIndex:2];
            [invocation setSelector:action];
            [invocation setTarget:target];
            [invocation invoke];
            CGFloat result = 0;
            [invocation getReturnValue:&result];
            return @(result);
        }
    
        if (strcmp(retType, @encode(NSUInteger)) == 0) {
            NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:methodSig];
            [invocation setArgument:&params atIndex:2];
            [invocation setSelector:action];
            [invocation setTarget:target];
            [invocation invoke];
            NSUInteger result = 0;
            [invocation getReturnValue:&result];
            return @(result);
        }
    
    #pragma clang diagnostic push
    #pragma clang diagnostic ignored "-Warc-performSelector-leaks"
        return [target performSelector:action withObject:params];
    #pragma clang diagnostic pop
}
```
通过这两个方法我们就可以看到整个 CTMediator 实现的思路了，为什么 AViewController 不引用 BViewController 还能向其进行跳转传值，原来都是由于 `runtime` 在中间起作用。
当然，虽然中间者这个方案能很好地实现各页面之间的解耦，但是也有它的缺点。我们可以看到我们在 `CTMediator+RegistViewControllerActions.h` 中定义的 `gc_targetRegistVC` 和 `gc_actionRegistVC` 这两个常量，分别对应 ‘target’ 和 ‘action’，这里面需要注意的是一定要细心，如果这儿写错，会引发未知的错误，但是编译器并不会提示，对应的 `Target_...`一定要和这里的 `target` 一致，否则就会引发错误。这种方案的实施对开发人员的细心程度是有很大要求的，因为如果有错误，在编译中无法发现的。
组件化的方案的实施还有很多其他的方案，比如 `url-block`、`protocol-class`方式，有兴趣的可以看看蘑菇街的 [MGJRouter](https://github.com/meili/MGJRouter)，还有就是阿里的 [BeeHive](https://github.com/alibaba/BeeHive) ，它是基于 Spring 的 Service 理念，使用 `Protocol` 的方式进行模块间的解耦。

## 四、关于 AOP
&emsp;&emsp;先看一个案例，小 C 最近愁眉苦脸，你发现了他状态不对劲，于是就发生了下面的对话

```
你：“小 C，你这是怎么啦，是不是工作上有什么不顺心的？”

小 C：“是啊，最近接到一个需求，让我很头疼！”

你：“接到需求不是很正常，做就是了啊！”

小 C：“你不知道，这个需求是统计每个页面的浏览情况，就是用户到了这个页面我就要统计一下，
运营产品他们要看 PV，于是我就在基类里面的 `viewDidLoad` 方法加了一下，这样很简单就解决了”

小 C：“可是他们又说还要我做每个页面按钮的点击统计，你说这 APP 几百个页面，这么多按钮，我怎么加啊，
就算我加了，我的代码也会因为这些与业务无关的代码而变得混乱，万一哪天不统计再让我删了，那我不是要命了啊！愁死我了！”

你：“那你这使用 AOP 就可以了啊”

小 C ：“A...OP？？？”
```
[AOP](https://zh.wikipedia.org/wiki/%E9%9D%A2%E5%90%91%E5%88%87%E9%9D%A2%E7%9A%84%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1)（Aspect-oriented programming），面向切面编程，是计算机科学中的一种程序设计思想，旨在将横切关注点与业务主体进行进一步分离，以提高程序代码的模块化程度。在 iOS 中有一个应用非常多的轻量级的 AOP 库 [Aspects](https://github.com/steipete/Aspects) ，它允许你能在任何一个类和实例的方法中插入新的代码。看到这里，你可能就已经知道小 C 的问题该如何解决了，下面是使用 Aspects 实现页面统计的代码

```objc
//
//  GCViewControllerIntercepter.m
//  GCUser
//
//  Created by HenryCheng on 2019/4/25.
//  Copyright © 2019 HenryCheng. All rights reserved.
//

#import "GCViewControllerIntercepter.h"
#import <Aspects/Aspects.h>

@implementation GCViewControllerIntercepter

+ (void)load {
    [super load];
    
}

+ (GCViewControllerIntercepter *)sharedInstance {
    static GCViewControllerIntercepter *_sharedClient = nil;
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _sharedClient = [[GCViewControllerIntercepter alloc] init];
    });
    return _sharedClient;
}

- (instancetype)init {
    if (self == [super init]) {
    
        [UIViewController aspect_hookSelector:@selector(viewDidLoad)
                           withOptions:AspectPositionAfter
                            usingBlock:^(id<AspectInfo> aspectInfo, UITouch *touch, UIEvent *event) {
                                
                                if ([aspectInfo.instance isKindOfClass:[UIViewController class]]) {
                                    
//                                    页面统计的代码
                                }
                            } error:NULL];
        
        [UIControl aspect_hookSelector:@selector(beginTrackingWithTouch:withEvent:)
                           withOptions:AspectPositionAfter
                            usingBlock:^(id<AspectInfo> aspectInfo, UITouch *touch, UIEvent *event) {
                                
                                if ([aspectInfo.instance isKindOfClass:[UIButton class]]) {
//                                    按钮统计的代码
                                    
                                }
                            } error:NULL];
        
        
    }
    return self;
}

@end
```
我们可以看到，通过新建 `GCViewControllerIntercepter` 这个类就实现了页面的统计和按钮点击统计功能，你只需要实现就行，连导入都不用，如果哪天你不需要这些统计的代码了，你直接从项目中移除这个类就可以了。是不是很简单！这就是 AOP 的一个使用实例，通过 `+ (void)load ` 这个方法（`+ load` 作为 Objective-C 中的一个方法，与其它方法有很大的不同。它只是一个在整个文件被加载到运行时，在 main 函数调用之前被 ObjC 运行时调用的钩子方法），实现了 `GCViewControllerIntercepter` 这个类被调用，然后通过 Aspects 实现对 UIViewController 和 UIControl 的 hook。这样在每个页面被加载、每个按钮被点击之前这边就可以捕捉到。
还有就是有人提到过去基类，也就是抛弃厚重的 base ，直接使用 AOP ，这样的话比如我想写个新 demo 就不用引入各种父类了，直接 hook 拿来用就好了。这种方法个人觉得没有到大工程的时候还是用继承来实现比较好。如果工程量比较大便于各个开发人员调试，可以使用这种方法。
当然 AOP 的作用也不仅如此，这里就说这么一个我们常用的 hook 的例子，有兴趣可以下去好好了解下。
> 1、`AspectOptions` 有四个值，分别是 `AspectPositionAfter`、`AspectPositionInstead`、`AspectPositionBefore`和 `AspectOptionAutomaticRemoval`，这样你可以决定你 hook 的位置
> 
> 2、对于 `+ (void)load` 还有 `+ (void)initialize` 这两个方法不是太了解的童鞋可以看看大左 [Draveness](https://draveness.me/) 的 [你真的了解 load 方法么？](https://draveness.me/load) 和 [懒惰的 initialize 方法](https://draveness.me/initialize) 这两篇文章，了解这两个方法相信对你会很有帮助

## 五、实际项目中的应用
&emsp;&emsp;了解了上面的内容，接下来我们看看在实际项目中的应用

##### 1、项目的目录结构

<p align='center'>
<img src='https://github.com/loveway/LearnBlog/blob/master/Notes/Objective-C/image/ar_catalog1.png'>
</p>

&emsp;&emsp;重构的项目结构如上图，相信大家一看名称就大概知道每个文件夹是做什么的，由于 `Model`、`View`、`ViewController` 和 `ViewModel` 这几个类联系比较紧密，所以建议这几个类的项目结构保持一致，如下图     

<p align='center'>
<img src='https://github.com/loveway/LearnBlog/blob/master/Notes/Objective-C/image/ar_catalog2.png'>
</p>

这样目录一目了然，比如你想找一个登录相关的东西，那么你就知道可以在各大目录下的 `Login` 模块里面去寻找。而且建议目录不要过深，一般三层就够了，过深的话查找起来比较麻烦。
##### 2、Category 的使用
&emsp;&emsp;可能大家已经看到了，我的项目目录里面有一项是 `AppDelegate+Config` 这一项，这其实就是 `AppDelegate` 的一个 `Category` 。在 iOS 开发中 `Category` 随处可见，如何应用那就是看自己的需求情况了，这里我用  `AppDelegate+Config` 这个类来处理 `AppDelegate` 里面的一些配置，减少 `AppDelegate` 的代码，让项目更加清晰，使用了以后我们可以看到 `AppDelegate` 目录的代码片段

 ```objc
    #import "AppDelegate.h"
    #import "AppDelegate+Config.h"
    #import "GCPushManager.h"
    
    @interface AppDelegate ()
    
    @end
    
    @implementation AppDelegate
    - (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    
        [self configTabbar];
        [self registWeChat];
        [self configNetWork];
        [self configJPushWithLaunchOptions:launchOptions];
        [self configKeyboard];
        [self configBaiduMobStat];
        [self configShareSDKWithLaunchOptions:launchOptions];
        return YES;
    }
    // between iOS 4.2 - iOS 9.0
    - (BOOL)application:(UIApplication *)application openURL:(NSURL *)url sourceApplication:(nullable NSString *)sourceApplication annotation:(id)annotation {
        [self handleOpenURL:url];
        return NO;
    }
    // after iOS 9.0
    - (BOOL)application:(UIApplication *)app openURL:(NSURL *)url options:(NSDictionary<UIApplicationOpenURLOptionsKey, id> *)options {
        [self handleOpenURL:url];
        return NO;
    }
   ```
这样代码看起来很清晰，相信大家都有过一打开 `AppDelegate` 这个类看到一大堆代码，东找西找很不规范的经历。由于是项目重构初期，`AppDelegate` 和 `AppDelegate+Config` 使用的比较多，暂时先放在这里，后期再将其移动到合适的位置。

##### 3、[CocoaPods](https://cocoapods.org/) 的使用
&emsp;&emsp;相信这个东西大家都用过，为什么要强调一下 CocoaPods 的使用，因为在我整理之前项目时发现，有的地方（比如微信支付、支付宝支付）就是直接将 lib 直接拖进工程，有的还需要各种配置，这样如果升级或者移除的时候就很麻烦。使用 CocoaPods 管理的话那么升级或者移除就很方便，所以建议还是能使用 CocoaPods 安装的就直接使用其安装，最好不要直接在项目中添加第三方。
  还有一种情况就是有时候第三方满足不了我们的需求，需要修改一下，所以有些就不集成在 CocoaPods 里面了（万一一不小心 update 以后修改的内容被覆盖）。这里我想说的是，对于这种情况你仍然可以使用 CocoaPods，那么怎么解决需要修改代码的问题？没错，就是 Category ！
>Tips:
>
>如果是团队多人协作开发，每个成员安装有不同版本 cocoapods gem，以至于在 pod install 的时候会安装不同版本的 cocoapods。为了解决这个问题，使团队成员拥有相同版本的 cocoapods，我们可以用 [Gemfile](https://guides.cocoapods.org/using/a-gemfile.html)，具体步骤是
>
>1、`gem install bundler`
>
>2、`cd Your Project`
>
>3、`bundle init`
>
>4、`vim Gemfile` and add 
>
```
    # frozen_string_literal: true

    #source 'https://rubygems.org
    source 'https://gems.ruby-china.com'

    git_source(:github) {|repo_name| "https://github.com/#{repo_name}" }

    # gem "rails"
    gem 'cocoapods', '1.7.0'
```
>
>5、`bundle exec pod install --verbose --no-repo-update` or `bundle exec pod update --verbose --no-repo-update`
  
##### 4、MVVM 的运用
&emsp;&emsp;具体项目的实现我们还是以登录为例，在 ViewModel 中
  
   ```objc
- (void)initialize {
        [super initialize];
        RAC(self, isLoginEnable) = [[RACSignal combineLatest:@[
                                                               RACObserve(self, phone),
                                                               RACObserve(self, password)
                                                               ]] map:^id _Nullable(RACTuple * _Nullable value) {
                                        RACTupleUnpack(NSString *phone, NSString *password) = value;
                                        return @([phone isNotBlank] && [password isNotBlank] && phone.length == 11); }];
        
        RAC(self.loginRequest, params) = [[RACSignal combineLatest:@[
                                                        RACObserve(self, phone),
                                                        RACObserve(self, password)
                                                        ]] map:^id _Nullable(RACTuple * _Nullable value) {
                               
                                 RACTupleUnpack(NSString *phone, NSString *password) = value;
                                     return @{@"phone": GC_NO_BLANK(phone),
                                              @"pwd": GC_NO_BLANK(password)
                                              }; }];
}
- (RACCommand *)loginCommand {
        if (!_loginCommand) {
            @weakify(self);
            _loginCommand = [[RACCommand alloc] initWithSignalBlock:^RACSignal * _Nonnull(id  _Nullable input) {
                @strongify(self);
                return [self.loginRequest requestSignal] ;
            }];
        }
        return _loginCommand;
}
```
这里我们做了网络的请求以及一些数据的绑定，在 ViewController 中
```objc
- (void)gc_bindViewModel {
        [super gc_bindViewModel];
        
        RAC(self.viewModel, phone) = self.loginView.phoneTextField.rac_textSignal;
        RAC(self.viewModel, password) = self.loginView.passwordTextField.rac_textSignal;
        RAC(self.loginView.loginButton, enabled) = RACObserve(self.viewModel, isLoginEnable);
        
        @weakify(self);
        
        [[[self.loginView.loginButton rac_signalForControlEvents:UIControlEventTouchUpInside] throttle:0.25] subscribeNext:^(__kindof UIControl * _Nullable x) {
            @strongify(self);
            [self.viewModel.loginCommand execute:nil];
        }];
        
        [[self.viewModel.loginCommand.executing skip:1] subscribeNext:^(NSNumber * _Nullable x) {
            if (x.boolValue) {
                [GCHUDManager show];
            } else {
                [GCHUDManager dismiss];
            }
        }];
        // 登录命令监听
        [self.viewModel.loginCommand.executionSignals.switchToLatest subscribeNext:^(NSDictionary *x) {
            @strongify(self);
            UserModel *userModel = [UserModel modelWithDictionary:x];
            [[GCCacheManager sharedManager] updateDataWithDictionary:x key:GCUserInfoStoreKey()];
            [GCPushManager gc_setAlias:x[@"phone"]];
            if (userModel.is_agree.intValue == 0) {
    //            未同意甘草协议
    
            } else if (userModel.is_agree.intValue == 1 && userModel.pwd_status.intValue == 0) {
    //            同意协议但是没改过密码
    
            } else {
    //            登录
            }
        } error:^(NSError * _Nullable error) {
            
        }];
}
```
  可以看到 ViewController 将 View 和 ViewModel 进行了绑定，并且当登录按钮点击的时候监测登录信号的变化，根据其信号执行的开始和结束来控制 HUD 的显示和消失，然后再根据信号的返回结果来处理相关的登录配置和跳转（极光推送的登录、根据状态执行跳转逻辑等）。这里网络的请求都是在 ViewModel 中进行的，ViewController 只负责处理ViewModel、View 和 Model 之间的关系。
##### 5、[DRY](https://zh.wikipedia.org/wiki/%E4%B8%80%E6%AC%A1%E4%B8%94%E4%BB%85%E4%B8%80%E6%AC%A1)
&emsp;&emsp;DRY（Don't repeat yourself），能封装起来的类一定要封装起来，到时候使用也简单，千万不要为了一时之快而各种 `ctrl + c` 和 `ctrl + v`，这样会使你的代码混乱不堪，这其实也是项目臃肿的一个原因。在重构的过程中就封装了很多的类，管理起来很方便

<p align='center'>
<img src='https://github.com/loveway/LearnBlog/blob/master/Notes/Objective-C/image/ar_catalog3.png'>
</p>

## 六、一些感想
&emsp;&emsp;其实最开始的时候一直都有重构的想法，但是迟迟没有动手。其中一个原因就是不知道该如何动手，不知道该使用什么工具，该使用哪种方案。等到真正开始的时候发现其实没有想象中的那么难，所以当你有想法的时候你就去做，在做的过程中你可以慢慢体会。
在重构之前，我又重新读了一下代码规范，也就是 [禅与 Objective-C 编程艺术](https://github.com/objc-zen/objc-zen-book-cn) 这本书，并在重构的过程中严格执行，比如 `loginButton` 就绝不会写成 `loginBtn`，相信我，按着规范来，你会体会到其中的意义的。
在做一个 APP 之前，在我们新建工程的时候，就应该已经确定你的架构模式，并且在以后的业务处理中，严格的按着这种设计模式执行下去。如果在前面需求量不多的时候你还能按着最初的设计模式执行下去，在业务突然增多的时候，为了偷懒省事，直接各种代码混乱的糅合在一起，各种 `ctrl + c`和`ctrl + v`，导致架构的混乱引起蝴蝶效应，那么这个架构在后期如果再想重新规范起来将会是个费时费力的过程。所以，在最初设计的时候我们就应该确定架构方案，以及严格的执行下去。
还有就是平时的一些技术积累以及知识存储。知其然知其所以然，研究技术背后的底层原理，会对你有很大的帮助。比如说我要说来说说 ViewController 的生命周期，可能大家都会随口说出 `viewDidLoad`、`viewWillAppear` 等，我要问说说 View 的生命周期，可能就会有少数人茫然了。这些都是很基本的东西，可能你平时用不到，但是还是需要你去了解他，注意细节。很多人可能会经常有这样的困惑，比如我想写一个图片浏览器，但是我不知道该如何写？写完了性能如何？别人是怎么写的？这个就是需要平时的积累了，比如关于 `UIText` 相关的的你就得想到 [YYText](https://github.com/ibireme/YYText)，数据存储方面的你不仅要知道老的 [fmdb](https://github.com/ccgus/fmdb) ，微信开源的 [wcdb](https://github.com/Tencent/wcdb) 有没有去了解下呢？比如我就平时没事喜欢在 [GitHub](https://github.com/) 上看一些 star 比较高的开源库，看看别人是怎么实现的，想想在我的项目中怎么使用。举个例子，最近阿里开源的 [协程](https://zh.wikipedia.org/wiki/%E5%8D%8F%E7%A8%8B) 框架 [coobjc](https://github.com/alibaba/coobjc/blob/master/README_cn.md) ，就在项目中使用，用来判断用户是否登录

```objc
- (void)judgeLoginBlock:(void(^)(GCLoginStatus status))block {

    co_launch(^{
        NSDictionary *dic = await([self co_loginRequest]);
        if (co_getError()) {
            block(GCLoginStatusError);
        } else if (dic) {
            if ([dic[@"status"] intValue] == 1) {
                block(GCLoginStatusLogin);
            } else if ([dic[@"status"] intValue] == -99) {
                block(GCLoginStatusUnLogin);
            } else {
                block(GCLoginStatusError);
            }
        } else {
            block(GCLoginStatusError);
        }
    });
}
```
一眼看去逻辑就很简单明了，比 Block 嵌套 Block 这种方式优雅的多。
现在只是重构的开始，现在已经完成的登录的重构就 `LoginViewController` 而言，与之前相比就已经有很大的改变了（之前将近 800 行代码，重构后只有 200 行），可能总体上各个模块代码加起来都差不多，但是为 ViewController 减负后更加清晰明了了。后面重构完成后会出一个代码量、包大小、性能等的对比，到时候再与大家分享！


>Reference:
>
> 1、[浅谈 MVC、MVP 和 MVVM 架构模式](https://draveness.me/mvx) 
>
> 2、[iOS应用架构谈 view层的组织和调用方案](https://casatwy.com/iosying-yong-jia-gou-tan-viewceng-de-zu-zhi-he-diao-yong-fang-an.html)
>
> 3、[iOS 如何实现Aspect Oriented Programming](https://www.jianshu.com/p/dc9dca24d5de)
> 
> 4、[CTMediator](https://github.com/casatwy/CTMediator)
>  
> 5、[BeeHive](https://github.com/alibaba/BeeHive)
>  
> 6、[objc-zen-book](https://github.com/objc-zen/objc-zen-book-cn)
>  
> 7、[coobjc](https://github.com/alibaba/coobjc/blob/master/README_cn.md)
>
> 8、[Using a Gemfile](https://guides.cocoapods.org/using/a-gemfile.html)
