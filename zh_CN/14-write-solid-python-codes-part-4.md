# 写好面向对象代码的原则（补充）

## D：迪米特原则

迪米特原则(Law of Demeter) 也称之为最少知道原则（The Least Knowledge Principle）, 
它认为： **"即一个软件实体需尽可能少地与其他实体发生相互作用。"** 换言之，—-**不要和陌生人说话**

> a given object should assume as little as possible about the structure or properties of anything else (including its subcomponents), in accordance with the principle of "information hiding"

一个类对另一个类应知道的越少越好，尽量降低类中成员变量及成员函数的访问权限，就相当于不把相应的变量和方法暴露给其他类，
这样可以尽量少地影响其他类或模块，这样的代码扩展起来会相对容易。

迪米特法则首先强调的前提是在类的结构设计上，每一个类都应当尽量降低成员的访问权限，也就是说，一个类包装好自己的private状态，不需要让别的类知道的字段或行文就不要公开。

当前类对象的直接朋友包括：   
>
> 1.当前类对象本身 
> 2.当前类对象创建的实例对象  
> 3.当前类对象的成员对象  
> 4.当前类对象的实例所引用的对象  
> 5.以参数形式传入到当前类对象方法中的对象  

迪米特法则强调了以下两点：
>
> 从依赖者的角度：只依赖应该依赖的对象  
> 从被依赖者的角度：只暴露应该暴露的方法或属性，即编写相关的类时确定方法和属性的权限(public or private)  

## 一个违反 D 原则的样例
张三去去找李四帮忙做一件事，对于李四来说这件事也很难办李四也做不了，但是李四的朋友王五却能完成这件事，所以李四就把这件事交给王五去办。
(注意：在本例中张三和王五是不认识的)  
```python
class A(object):
    def __init__(self, name):
        self.name = name

    def get_b(self, name):
        return B(name)

    def work(self):
        b = self.get_b('李四')
        c = b.get_c('王五')
        c.work()

class B(object):
    def __init__(self, name):
        self.name = name
    
    def get_c(self, name):
        return C(name)

class C(object):
    def __init__(self,name):       
        self.name = name
    
    def work(self):
        print('task finished by {name}'.format(name=self.name))

if __name__ == '__main__':
    a = A('张三')
    a.work()
```

1 `a`: 张三  
2 `b`: 李四  
3 `c`: 王五  

上面的代码输出结果是符合期望的。“王五确实把事情办妥了”，但是我们仔细看代码逻辑确发现这样做是不对的，
因为张三和王五我们已经预设了互相不认识的前提，那为什么代表张三的A类中会有代表王五的C类呢？所以这样的代码明显是违背了迪米特原则的。  

### 正确的修改办法

在本例中张三只认识李四，那么只能依赖李四，重构后代码如下：

```python
class A(object):
    def __init__(self, name):
        self.name = name

    def get_b(self, name):
        return B(name)

    def work(self):
        b = self.get_b('李四')
        b.work()

class B(object):
    def __init__(self, name):
        self.name = name
    
    def get_c(self, name):
        return C(name)
    
    def work(self):
        c = self.get_c('王五')
        c.work()

class C(object):
    def __init__(self,name):       
        self.name = name
    
    def work(self):
        print('task finished by {name}'.format(name=self.name))

if __name__ == '__main__':
    a = A('张三')
    a.work()
```

## C：合成复用原则

合成复用原则 “Composite Reuse Principle, CRP” 认为：**“在一个新的对象里通过关联关系（包括组合关系和聚合关系）来使用一些已有的对象，
使之成为新对象的一部分；新对象通过委派调用已有对象的方法达到复用功能的目的。”**，换言之，**尽量使用对象组合，而不是继承来达到复用的目的。**

通过继承来进行复用的主要问题在于继承复用会破坏系统的封装性，因为继承会将基类的实现细节暴露给子类，由于基类的内部细节通常对子类来说是可见的，
所以这种复用又称 **“白箱”复用**，如果基类发生改变，那么子类的实现也不得不发生改变；从基类继承而来的实现是静态的，不可能在运行时发生改变，
没有足够的灵活性；而且继承只能在有限的环境中使用（如类没有声明为不能被继承）。  

由于组合或聚合关系可以将已有的对象（也可称为成员对象）纳入到新对象中，使之成为新对象的一部分，因此新对象可以调用已有对象的功能，
这样做可以使得成员对象的内部实现细节对于新对象不可见，所以这种复用又称为 **“黑箱”复用**，相对继承关系而言，其耦合度相对较低，
成员对象的变化对新对象的影响不大，可以在新对象中根据实际需要有选择性地调用成员对象的操作；合成复用可以在运行时动态进行，
新对象可以动态地引用与成员对象类型相同的其他对象。  

如果两个类之间是 **“Has-A”** 的关系应使用组合或聚合，如果是 **“Is-A”** 关系可使用继承  

## 一个违反 C 原则的样例
Sunny软件公司开发人员在初期的CRM系统设计中，考虑到客户数量不多，系统采用MySQL作为数据库，  
与数据库操作有关的类如CustomerDAO类等都需要连接数据库，连接数据库的方法getConnection()封装在DBUtil类中，  
由于需要重用DBUtil类的getConnection()方法，设计人员将CustomerDAO作为DBUtil类的子类，初始设计方案结构如图1所示：  
![图1](https://raw.githubusercontent.com/wenb/one-python-craftsman/master/img/dao1.jpg)

## 改造方法
在图2中，CustomerDAO和DBUtil之间的关系由继承关系变为关联关系，采用依赖注入的方式将DBUtil对象注入到CustomerDAO中，可以使用构造注入，  
也可以使用Setter注入。如果需要对DBUtil的功能进行扩展，可以通过其子类来实现，如通过子类OracleDBUtil来连接Oracle数据库。  
由于CustomerDAO针对DBUtil编程，根据里氏代换原则，DBUtil子类的对象可以覆盖DBUtil对象，只需在CustomerDAO中注入子类对象即可使用子类所扩展的方法。  
例如在CustomerDAO中注入OracleDBUtil对象，即可实现Oracle数据库连接，原有代码无须进行修改，而且还可以很灵活地增加新的数据库连接方式。  
![图2](https://raw.githubusercontent.com/wenb/one-python-craftsman/master/img/dao2.jpg)
  

## 总结

- **“D：迪米特原则”** 认为在不违反需求的情况下尽量降低每个类中成员变量和成员函数的访问权限
- 合理引入第三方降低对象之间直接交互的耦合度更不容易违反 D 原则
- 过度应用迪米特法则会产生大量的代理类或中介类，导致系统过于复杂。所以需要权衡使用， 使得代码既满足高内聚低耦合，又能够做到结构清晰。
- **“C: 合成复用原则”** 认为尽量使用对象组合，而不是继承来达到复用的目的。
