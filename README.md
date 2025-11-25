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

## 2.OC和Swift混编时，方法参数不加nonnull时，在Swift侧存在判空问题
方法参数不加nonnull时，在Swift侧可能存把当前参数当错非空参数，如果对参数进行判空，在Xcode配置项Swift Compiler下面的Optimization Level设置成optimize for speed[-O]条件下，编译器会默认通过判空，直接执行通过后的代码，导致崩溃；

## 3. 

# Swift侧
## 1. 类型转换：as?
先看示例：
```
let dic : [String: Any] = ["floatV" : 0.5]
let floatV = dic["floatV"] as? Float
print(floatV)
floatV = nil
```
当通过as?进行类型转换时，转换浮点类型时要特别注意，Swift中默认存储的是Double类型，
转换错误类型或导致取不到值；
解决办法：可以先尝试使用 as? 转成NSNumber或者NSString类型，再取值；
