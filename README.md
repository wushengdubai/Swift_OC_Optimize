# Swift 和 OC混编
## 1. Swift继承OC类时，OC类中要指定Designated Initializer
如果不指定Designated Initializer，切存在大于1个的初始化方法，则会在Swift侧会存在多个Designated Initializer初始化方法；初始化Swift类时，会存在多次初始化，其中多初始化的实例会变成无主引用，导致内存泄露；
解决版本，在OC类中，指定一个Designated Initializer。
示例代码：
```
// Objc Base Class

@interface BaseObjcCls : NSObject

- (instancetype)initWithFoo1:(NSInteger)foo1;

- (instancetype)initWithFoo1:(NSInteger)foo1 foo2:(NSInteger)foo2;

@end

@implementation BaseObjcCls

- (instancetype)initWithFoo1:(NSInteger)foo1 {
    if (self = [super init]) {
        NSLog(@"%@ - [initWithFoo1:]", self);
    }
    return self;
}

- (instancetype)initWithFoo1:(NSInteger)foo1 foo2:(NSInteger)foo2 {
    if (self = [self initWithFoo1:foo1]) { // Pay Attention⚠️
        NSLog(@"%@ - [initWithFoo1:foo2:]", self);
    }
    return self;
}

@end
```
```
// Swift Subclass

class SwiftSubCls: BaseObjcCls {    
    deinit {
        print("\(self) - deinit")
    }
    
    let obj = Foo()
}

class Foo: NSObject {
    override init() {
        super.init()
        print("\(self) - init")
    }

    deinit {
        print("\(self) - deinit")
    }
}
```
```
// Test Demo

var obj: AnyObject? = SwiftSubCls(foo1: 1, foo2: 2)
obj = nil
print("obj is released.")
```
运行测试代码，输出如下：
```
// The following logs have been formatted for readability.

<Foo: 0x60000125c080> - init
<Foo: 0x60000125c0a0> - init
<SwiftSubCls: 0x60000125c030> - [initWithFoo1:]
<SwiftSubCls: 0x60000125c030> - [initWithFoo1:foo2:]
<SwiftSubCls: 0x60000125c030> - deinit
<Foo: 0x60000125c0a0> - deinit
obj is released.
```
可以非常直观地看到<Foo: 0x60000125c080>出现了内存泄漏的现象。

## 2.
