---
layout: post
title: 关于Object-C的属性修饰词strong,weak...
category: object-c
url: object-c-property.html
summary: 关于Object-C的Property的修饰词strong,weak...的一些分析
---

####内存管理
在iOS中所有的对象都是从堆（heap）中分配的。
1. 所有的静态常量都是保存在数据段（Data Segement）中，你不需要关心其内存的释放。这些静态常量包括：NSString，NSInteger，BOOL，int，long，char等
2. 类对象，这些对象通过`Reference Count`管理，可以看这个图：
![Image]({{ site.baseurl }}/images/memory_management.png)

####retain
我们先说retain，retain的意思是“保留，保持”，其实Object-C里面的关键字都是非常直白的，这里“保持”的意思就是“我要保持一个引用，不关心别人怎么做”，体现出来就是`count++`；除非你主动释放，否则所引用的对象会一直有效，我们可以做下面的代码测试：
*声明class: TestObject*
{% highlight objc %}
@interface TestObject : NSObject
@end 
@implementation TestObject
- (NSString *)description {
    return @"Test";
}
@end
{% endhighlight %}

*在class: Tester中声明retain property*
{% highlight objc %}
@interface Tester : NSObject
@property (nonatomic, retain)TestObject *retainObject;
@end
@implementation Tester
@synthesize retainObject;
@end
{% endhighlight %}

*在main方法中执行测试*
{% highlight objc %}
// main.m
Tester *tester = [Tester new];
TestObject *obj = [TestObject new];
tester.retainObject = obj;
obj = nil;
// NSLog(@"%@", tester.retainObject); //NSLog refer to retainObject will cause count++
if (tester.retainObject) {
    NSLog(@"retain is not nil");
} else {
    NSLog(@"retain is nil");
}
// print "retain is not nil"
{% endhighlight %}

####strong
`strong`就是“强引用”的意思，和`weak`相对，可以认为就是ARC模式下的`retain`，在iOS5以后引入，在ARC模式下所有属性都是默认使用strong修饰，会触发“count++”。
我们建立同样的测试代码：
{% highlight objc %}
// main.m
Tester *tester = [Tester new];
TestObject *obj = [TestObject new];
tester.strongObject = obj;
obj = nil;
if (tester.strongObject) {
    NSLog(@"strong is not nil");
} else {
    NSLog(@"strong is nil");
}
// print "strong is not nil"
{% endhighlight %}
**预防强引用循环**<br />
强引用循环的对象将永远不能被释放，比如UITableView引用了其delegate，而其delegate有引用回了该tableView。<br />
![Image]({{ site.baseurl }}/images/strongreferencecycle2.png)<br /><br />
正确的应该是：<br /><br />
![Image]({{ site.baseurl }}/images/strongreferencecycle3.png)

####weak
`weak`是与`strong`相对的“弱引用”，不会影响“Reference count”，如果对象通过*garbage collection*从堆中被回收了，那么该指针会被自动置为`nil`。（reference count = 0与garbage collection无必然联系，意味着即使reference count = 0，该指针也不会马上变为`nil`）
{% highlight objc %}
// main.m
Tester *tester = [Tester new];
TestObject *obj = [TestObject new];
tester.weakObject = obj;
obj = nil;
if (tester.weakObject) {
    NSLog(@"weak is not nil");
} else {
    NSLog(@"weak is nil");
}
// print "strong is nil"
{% endhighlight %}

####assign
`assign`的意思就是“赋值”，我们知道在Object-C里面都是使用指针，那么如果一个属性为`assign`的话，那么就意味着是指针赋值，也就是把传入对象的内存地址赋值给了该属性，与“Reference Count”无关。
`assign`一般要用于静态常量类型，比如我们前面提到的NSString，NSInterger，BOOL, long, int, char等，因为这些在内存中是长期驻留的，不用担心野指针的问题；但如果你用在了一般的对象中的话，一旦该对象的“Reference Count = 0”（已经不能访问了）的话，这个属性就变成了一个野指针让你的程序出错（`EXC_BAD_ACCESS`）。（与之不同，weak会自动置为nil）
{% highlight objc %}
// main.m
Tester *tester = [Tester new];
TestObject *obj = [TestObject new];
tester.assignObject = obj;
obj = nil;
if (tester.assignObject) { // Cause EXC_BAD_ACCESS
    NSLog(@"assign is not nil");
} else {
    NSLog(@"assign is nil");
}
//  Cause EXC_BAD_ACCESS
{% endhighlight %}

####copy
`copy`的意思就是复制，相当于在堆中新申请了一块空间存放对象的副本（Reference Count = 1）。
使用`copy`的属性的类必须要实现`NSCopying`协议。
*class: TestObject实现协议`NSCopying`*
{% highlight objc %}
@interface TestObject : NSObject<NSCopying>
@end

@implementation TestObject

- (NSString *)description {
    return @"Test";
}

- (id)copyWithZone:(NSZone *)zone {
    return [TestObject new];
}
@end
{% endhighlight %}

添加测试代码：
{% highlight objc %}
// main.m
Tester *tester = [Tester new];
TestObject *obj = [TestObject new];
tester.cpObject = obj;
obj = nil;
if (tester.cpObject) {
	NSLog(@"copy is not nil");
} else {
    NSLog(@"copy is nil");
}
tester.weakObject = tester.cpObject;
tester.cpObject = nil;
if (tester.weakObject) {
    NSLog(@"weak is not nil");
} else {
    NSLog(@"weak is nil");
}
// print "copy is not nil"
// print "weak is nil"
{% endhighlight %}

#####readonly
`realonly`表示属性是只读的，不能通过`setter`方法对其进行修改。

####readwrite
`readwrite`是默认值，不需要显式的声明。

####atomic
所有的属性默认都是`atomic`的，`atomic`显然表示原子行为，说明是线程安全的。你不能为`atomic`的属性自定义setter和getter。

####nonatomic
`nonatomic`意味着不是线程安全的，可以同时被多个线程访问，如果同时有线程调用getter和setter的话，没办法保证返回的值。
