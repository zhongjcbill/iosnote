# 代理模式
## 代理
* 代理设计模式的作用:
    * 1.A对象监听B对象的一些行为，A成为B的代理
    * 2.B对象想告诉A对象一些事情，A成为B的代理

* 代理设计模式的总结：
    * 如果你想监听别人的一些行为，那么你就要成为别人的代理
    * 如果你想告诉别人一些事情，那么就让别人成为你的代理

* 代理设计模式的开发步骤
    * 1.拟一份协议（协议名字的格式：控件名 + Delegate），在协议里面声明一些代理方法（一般代理方法都是@optional）
    * 2.声明一个代理属性：@property (nonatomic, weak) id<代理协议> delegate;
    * 3.在内部发生某些行为时，调用代理对应的代理方法，通知代理内部发生什么事
    * 4.设置代理：xxx.delegate = yyy;
    * 5.yyy对象遵守协议，实现代理方法

## 代理和通知的区别
- 代理：1个对象只能告诉另1个对象发生了什么事
- 通知：1个对象可以告诉N个对象发生了什么事

## KVC\KVO
- KVC(Key Value Coding)常见作用：给模型属性赋值
- KVO(Key Value Observing)常用作用：监听模型属性值的改变
- KVO的使用步骤<br>

```objc
    self.deall = [[XMGDeal alloc] init];
    self.deall.price = @"110";
    self.deall.title = @"新增加的";
    self.deall.buyCount = @"22";

//添加监听
[self.deall addObserver:self forKeyPath:@"title" options:NSKeyValueObservingOptionNew context:@"大家好，我是context"];

    //监听了的title属性的改变
self.deall.title = @"晓勇哥带我拿首胜";
}

//当监听到object的keyPath属性发生了改变
- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSString *,id> *)change context:(void *)context
{
    NSString *str = [NSString stringWithFormat:@"keyPath---->%@,object--->%@,change--->%@",keyPath,object,change[@"new"]];
    NSLog(@"%@",str);
}

//log [3175:269829] keyPath---->title,object---><XMGDeal: 0x7fe2185376f0>,change--->晓勇哥带我拿首胜
```
