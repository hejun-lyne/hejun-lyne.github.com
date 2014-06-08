---
layout: post
title: 关于Object-C的属性修饰词strong,weak等
category: object-c
url: object-c-property.html
summary: 关于Object-C的Property的修饰词strong,weak...的一些分析
---
####retain
我们先说retain，retain的意思是“保留，保持”，其实Object-C里面的关键字都是非常直白的，这里“保持”的意思就是“我要保持一个引用，不关心别人怎么做”，体现出来就是`count++`；除非你主动释放，否则所引用的对象会一直有效，我们可以做下面的代码测试：

{% highlight objc %}
// 声明一个class
@interface TestObject : NSObject
@end
@implementation TestObject
- (NSString *)description {
    return @"Test";
}
@end
//
// 声明retain property
@interface Tester : NSObject
@property (nonatomic, retain)TestObject *retainObject;
@end
@implementation Tester
@synthesize retainObject;
@end
//
// main.m
Tester *tester = [Tester new];
TestObject *obj = [TestObject new];
tester.retainObject = obj;
NSLog(@"%@", tester.retainObject);
obj = nil;
NSLog(@"%@", tester.retainObject);
// Outputs: 
// Test
// Test
//
{% endhighlight %}

####strong