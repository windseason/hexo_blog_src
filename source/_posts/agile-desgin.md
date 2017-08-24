---
title: 敏捷设计之SOLID原则
date: 2017-08-24 15:38:11
tags: OOD
---

## 敏捷设计之SOLID原则

### 把握所学东西的本质

在讨论SOLID原则之前，我们需要明确一点，就是这个规则是为了解决什么问题而被发明出来的。只有弄清楚这一点，才能够在实际工作中正确应用。举个栗子吧，比如为什么要采用MVC？教科书般的回答就是MVC就是Model-View-Controller，然后将Model,View和Controller各自按照教科书说一遍。你觉得你真的理解为什么要用MVC了吗？举个简单的例子说明非MVC是什么样的吧？

```javascript
<!DOCTYPE html>
<html>
<body>

<%
set conn=Server.CreateObject("ADODB.Connection")
conn.Provider="Microsoft.Jet.OLEDB.4.0"
conn.Open(Server.Mappath("/db/northwind.mdb"))
set rs = Server.CreateObject("ADODB.recordset")
rs.Open "Select * from Customers", conn

do until rs.EOF
    for each x in rs.Fields
       Response.Write(x.name)
       Response.Write(" = ")
       Response.Write(x.value & "<br>") 
    next
    Response.Write("<br>")
    rs.MoveNext
loop

rs.close
conn.close
%>

</body>
</html>
```

这是一段ASP classic的代码。这段代码的问题：

- 无法在类似逻辑的页面重用使用数据库的代码。为了满足快速迭代的需求，将会促使程序员自然而然的使用copy-paste大法。
- 页面的显示与数据库代码混在一起，如果需求变化，要调整页面显示将会变得很麻烦。一个页面还好，如果是几十个这种代码的页面呢？

所以，从如何能够让这段代码重用为出发点，MVC在不断的试验与测试中诞生了。MVC的优点总结请自行Google。


### 设计臭味

设计一个软件的结构或者架构的时候，如果没有足够的经验，经常会产生如下问题:

#### 僵化性

僵化性是指对软件进行改动，即使是简单的改动。如果单一的改动会导致有依赖关系的模块中的连锁改动，那么设计就是僵化的。

#### 脆弱性

脆弱性是指，在进行一个改动时，可能会导致程序的许多地方出现问题。

#### 顽固性

顽固性是指，设计中包含了对其他有用的部分，但是要把这些部分从系统中分离出来所需要的努力和风险却是巨大的，这是一种令人遗憾，但非常常见的情况。

#### 粘滞性

粘滞性是指，当面临一个改动时，开发人员尝尝发现有多种改动的方法。其中，有一些会保持敏捷设计，而另外一些则会破坏设计。比如，如果编译所花费的时间很长，那么开发人员就会被诱导去做不会导致大规模编译的举动。

#### 不必要的复杂性

如果设计中包含了当前没用的组成部分，它就含有不必要的复杂性。当开发人员预测需求的变化，并在软件中放置了处理潜在变化的代码时，尝尝出现这种情况。起初，这样做看起来貌似是一件好事，但是结果尝尝正好相反。为过多的可能性做准备，致使设计由于含有绝不会用到的结构而变得混乱。

#### 不必要的重复性

copy-paste

#### 晦涩性

晦涩性是指模块难以理解。

怎么能尽量避免产生以上问题呢？那就需要借鉴敏捷设计。

### 敏捷设计原则之SOLID

大家都听说过敏捷开发宣言， 其中最后一条是"随时应对变化**重于**遵循计划"。要达这目的并且避免写出有臭味的代码，就需要引入敏捷设计，就是要让自己的代码够“敏捷”。我认为只有代码具有敏捷设计思想，敏捷开发宣言所倡导的才能够顺利实现。打个比喻，中国古代打战有许多排兵布阵的阵法，想想看如果使用的是一个玄妙无比的阵法，比如八卦阵，但是执行阵法的士兵大部分都是老弱残兵以及老幼妇孺，恐怕连阵势都摆不出来就战败了吧。我们这里说的敏捷设计原则就像士兵一样，是基石。

那到底什么是敏捷设计呢？敏捷设计是一个过程，不是一个事件。它是一个持续的应用原则、模式以及实践来改进软件的**结构**和**可读性**的过程。它致力于保持系统在任何事件都尽可能的**简单干净**以及富有表达力。

接下来，我们学习敏捷设计原则，但请记住，敏捷开发人员不会把这些原则应用到一个庞大的预先设计中。相反，这些原则被应用在一次次的迭代中，力图使代码以及代码所表达的设计保持干净。

#### 单一职责原则(SRP)

> 一个类应该只有一个发生变化的原因

如果一个类承担的职责过多，就等于把这些职责耦合在了一起。一个职责的变化可能会削弱或者抑制这个类完成其他职责的能力。

考虑下图的设计：
![](http://ot51d7lis.bkt.clouddn.com/Rectangle%201.png)

这个设计违反了SRP，显然Rectangle有两个职责。导致的问题：

- 打包到GUI程序中的时候，需要将计算机几何应用库与图形绘制应用库一起打包
- 如果依赖的任何一个库发生了改变，则需要重新对程序打包。如果忘记做，而仅仅只是更新其中一个库的话，将会导致不可预料的错误。

改进：
![](http://ot51d7lis.bkt.clouddn.com/Rectangle_good%20%281%29.png)

#### 开放关闭原则(OCP)

> 一个类对修改关闭，对扩展开放。具体来说就是，一个类在不修改它本身的代码的前提下允许扩展它的行为(如果不修改做不到到话，尽量把修改做到最小化)

OCP 听起来并不复杂，但是却不是那么容易实践的。因为通常情况下，为一个功能而设计可以对未来需求开放的类并不简单。这里重要的点是**未来需求**。当我们并不知道未来软件的需求的时候，我们将怎样设计类，模块，功能等来迎合需求呢？这就是OCP不容易实践的原因，通常需要彻底的思考设计。

我们该如何去解决这个问题呢？当设计类结构的时候，需要用心确类的扩展简单。设计阶段，将所有类的用例记下来，并对所有用例给出合适的接口。抽象类是一种能够确保类能够被扩展的手段，但是要小心不要违反**SRP**，当想让一个类做更多事情的时候就很容易发生。

##### 例子

该例子是一个计算个人所得税的例子。

```swift
class Individual: NSObject {
    let salary : Double;
    let age : NSInteger;
    let name : String;
    let countryCode : String;
    
    init(salary : Double,
         age : NSInteger,
         name : String,
         countryCode : String) {
        self.countryCode = countryCode;
        self.salary = salary;
        self.age = age;
        self.name = name;
    }
}

class TaxCalculator : NSObject {
    public func calculate(_ individual : Individual) -> Double {
        var tax : Double;
        
        //Just use flat calculation algorithm for simplicity as 
        //here we just use it to demostrate OCP.
        switch individual.countryCode {
        case "cn":
            tax = individual.salary * 0.18;
        case "us":
            tax = individual.salary * 0.20;
        case "jp":
            tax = individual.salary * 0.50;
        default:
            tax = 0.1;
        }
        
        return tax;
    }
}
```

##### 现有设计的缺点

现有设计的缺点很明显，如果我们想要增加一个新的国家的个税计算，则一定需要更改现有的TaxCalculator的代码。显然，当前设计不对扩展开放。

##### 一个更好的符合OCP的设计

```swift
protocol CalculateTax {
    func calculate(_ individual : Individual) -> Double;
}

class ChinaTaxCalculator: NSObject, CalculateTax {
    public func calculate(_ individual: Individual) -> Double {
        return individual.salary * 0.18;
    }
}

class USATaxCalculator: NSObject, CalculateTax {
    public func calculate(_ individual: Individual) -> Double {
        return individual.salary * 0.20;
    }
}

class JapanTaxCalculator: NSObject, CalculateTax {
    public func calculate(_ individual: Individual) -> Double {
        return individual.salary * 0.50;
    }
}

class TaxCalculator: NSObject {
    public func calculate(taxHandler : CalculateTax, individual : Individual) -> Double {
        return taxHandler.calculate(individual);
    }
}

```

现在，如果增加一个国家的税收计算，只需要添加一个相应国家的税收计算类**XXTaxCalculator**即可，并不需要更改任何已有代码。如果项目按照以下划分程序依赖库的结构，则只需要重新编译具体实现了各个国家税收的库就行了。

![](http://ot51d7lis.bkt.clouddn.com/Better%20OCP%20design.png)

#### Liskov替换原则(LSP)

> Liskov 替换原则说的是如果D是B的派生类，则任何B的应用都必须能够被D的实例替换而不会影响到程序运行的逻辑或者功能。换句话说，就是派生类必须能够完全替换基类。

我们先看一个大概只要你学Liskov原则都会看到的一个违反Liskov原则的例子。

##### 违反Liskov替换原则的例子

```swift
class Rectangle {
    public var width : Double;
    public var height : Double;
    
    init(width : Double, height : Double) {
        self.width = width;
        self.height = height;
    }
    
    func area() -> Double {
        return self.width * height;
    }
}

class Square : Rectangle {
    fileprivate var _width : Double = 0;
    fileprivate var _height : Double = 0;
    
    override public var width: Double {
        get {
            return _width;
        }
        set(newWidth) {
            _width = newWidth;
            _height = newWidth;
        }
    }
    
    override public var height: Double {
        get {
            return _height;
        }
        set (newHeight) {
            _width = newHeight;
            _height = newHeight;
        }
    }
}

func createRectangle(width : Double, height : Double) -> Rectangle {
    return Square(width: width, height: height);
}

/////////////////////////////////////////////////////////////////////

//Someday we use createRectangle method somewhere in our application.
var rect:Rectangle = createRectangle(width : 3, height : 3);
rect.area();//result is 9 as expected

rect.height = 10;
rect.width = 5;

rect.area();//opps!result is 25 not 50!
```

##### 结论

这个原则是对OCP的一个扩展，他告诉我们从基类派生出新的派生类的时候，我们必须保证不要破坏原有的行为。

#### 接口隔离原则(ISP)

> 接口隔离原则讲的是接口实现者不能被强迫实现或者依赖一些不会用到的方法

举个例子，当我们将一个模块从应用程序中抽象出多个子模块的时候需要小心注意了。假设这个模块是基于类实现的，那我们能够从类抽象出接口（注：这些接口都会在现有系统被逐一实现）。但当我们把模块加入到一个与原系统相比仅含有某些接口的实现，有可能会发生这种结果：为了保证模块能够正常运行，不得不在新系统中被迫实现所有的接口并且有相当一部分接口不会被用到，这部分不会被用到的接口通常为空实现或者返回虚假值，就为了实现接口而实现。通常发生这种情况的接口叫做胖接口或者被污染的接口。

##### 一个违反了ISP的实际例子

在早期的ASP.NET 2.0版本中，如果网站想实现自己的membership provider，则需要将需要或者不需要的所有接口都实现一遍。如下：

```csharp
public class CustomMembershipProvider : MembershipProvider
{
    public override string ApplicationName
    {
        get
        {
            throw new Exception("The method or operation is not implemented.");
        }
        set
        {
            throw new Exception("The method or operation is not implemented.");
        }
    }

    public override bool ChangePassword(string username, string oldPassword, string newPassword)
    {
        throw new Exception("The method or operation is not implemented.");
    }

    public override bool ChangePasswordQuestionAndAnswer(string username, string password, 
        string newPasswordQuestion, string newPasswordAnswer)
    {
        throw new Exception("The method or operation is not implemented.");
    }

    public override MembershipUser CreateUser(string username, string password, string email, 
        string passwordQuestion, string passwordAnswer, bool isApproved, object providerUserKey, 
        out MembershipCreateStatus status)
    {
        throw new Exception("The method or operation is not implemented.");
    }

    public override bool DeleteUser(string username, bool deleteAllRelatedData)
    {
        throw new Exception("The method or operation is not implemented.");
    }

    public override bool EnablePasswordReset
    {
        get { throw new Exception("The method or operation is not implemented."); }
    }

    public override bool EnablePasswordRetrieval
    {
        get { throw new Exception("The method or operation is not implemented."); }
    }

    public override MembershipUserCollection FindUsersByEmail(string emailToMatch, int pageIndex, 
        int pageSize, out int totalRecords)
    {
        throw new Exception("The method or operation is not implemented.");
    }

    public override MembershipUserCollection FindUsersByName(string usernameToMatch, int pageIndex, 
        int pageSize, out int totalRecords)
    {
        throw new Exception("The method or operation is not implemented.");
    }

    public override MembershipUserCollection GetAllUsers(int pageIndex, int pageSize, out int totalRecords)
    {
        throw new Exception("The method or operation is not implemented.");
    }

    public override int GetNumberOfUsersOnline()
    {
        throw new Exception("The method or operation is not implemented.");
    }

    public override string GetPassword(string username, string answer)
    {
        throw new Exception("The method or operation is not implemented.");
    }

    public override MembershipUser GetUser(string username, bool userIsOnline)
    {
        throw new Exception("The method or operation is not implemented.");
    }

    public override MembershipUser GetUser(object providerUserKey, bool userIsOnline)
    {
        throw new Exception("The method or operation is not implemented.");
    }

    public override string GetUserNameByEmail(string email)
    {
        throw new Exception("The method or operation is not implemented.");
    }

    public override int MaxInvalidPasswordAttempts
    {
        get { throw new Exception("The method or operation is not implemented."); }
    }

    public override int MinRequiredNonAlphanumericCharacters
    {
        get { throw new Exception("The method or operation is not implemented."); }
    }

    public override int MinRequiredPasswordLength
    {
        get { throw new Exception("The method or operation is not implemented."); }
    }

    public override int PasswordAttemptWindow
    {
        get { throw new Exception("The method or operation is not implemented."); }
    }

    public override MembershipPasswordFormat PasswordFormat
    {
        get { throw new Exception("The method or operation is not implemented."); }
    }

    public override string PasswordStrengthRegularExpression
    {
        get { throw new Exception("The method or operation is not implemented."); }
    }

    public override bool RequiresQuestionAndAnswer
    {
        get { throw new Exception("The method or operation is not implemented."); }
    }

    public override bool RequiresUniqueEmail
    {
        get { throw new Exception("The method or operation is not implemented."); }
    }

    public override string ResetPassword(string username, string answer)
    {
        throw new Exception("The method or operation is not implemented.");
    }

    public override bool UnlockUser(string userName)
    {
        throw new Exception("The method or operation is not implemented.");
    }

    public override void UpdateUser(MembershipUser user)
    {
        throw new Exception("The method or operation is not implemented.");
    }

    public override bool ValidateUser(string username, string password)
    {
        throw new Exception("The method or operation is not implemented.");
    }
}
```

Holy crap! 这是一大堆东西！可能有的时候，你仅仅是想将现有的membership provider的一部分改变成自己的而已，但是你要面对的是全部实现，这就麻烦了。如果你不懂ASP.NET没关系，这里这个例子只是想展示一种违反了ISP后的结果。

##### 一个更简单通俗的例子来说明ISP

首先看代码：

```swift
protocol Animal {
    func feed();
}

class Dog : Animal {
    func feed() {
        print("feed with bone");
    }
}

class Snake : Animal {
    func feed() {
        print("feed with mice");
    }
}
```

随后，你注意到有些动物是宠物，有宠物的一些特性或者与人互动的方法。

```swift
protocol Animal {
    func feed();
    //梳理毛发
    func groom();
}

class Dog : Animal {
    func feed() {
        print("feed with bone");
    }
    
    func groom() {
        print("grooming and dog feel comfortable");
    }
}

class Snake : Animal {
    func feed() {
        print("feed with mice");
    }
    
    func groom() {
        //ignore this method as I am not going to groom a feaking snake.
    }
}
```

写到这里，你就发现问题了。Animal接口被污染了。在Snake类上它要求我们实现一个并不需要的方法。好的做法是把宠物相关的方法抽象出来。

```swift
protocol Animal {
    func feed();
}

protocol Pet {
    //梳理毛发
    func groom();
}

class Dog : Animal, Pet {
    func feed() {
        print("feed with bone");
    }
    
    func groom() {
        print("grooming and dog feel comfortable");
    }
}

class Snake : Animal {
    func feed() {
        print("feed with mice");
    }
}
```

#### 依赖反转(DIP)

> A. High-level 模块不应该依赖 low-level 模块。两者都应该依赖于抽象
> B. 抽象不能依赖于具体实现。具体实现应该依赖于抽象

##### 例子

一个简单的例子，我们经常使用DAO(Database access object)来封装数据库操作的逻辑。下面这个例子，将数据库具体的类OracleConnection直接在DAO类里面了。那如果我们想把数据库换成Mysql，怎么办呢？（想想一下，有很多DAO类，里面的逻辑都很复杂很多）

```swift

class OracleConnection {
    public func open() {
        //open database connection logic
    }
    
    public func close() {
        
    }
}

class DAO {
    public var databaseConnection : OracleConnection;
    
    init() {
        databaseConnection = OracleConnection();
    }
    
    public func commitChanges() {
        
        databaseConnection.open();
        
        //deal with data base changes
        
        databaseConnection.close();
    }
}
```

符合DIP的写法:

```swift
protocol DatabaseConnection {
    func open();
    func close();
}

class OracleConnection : DatabaseConnection {
    public func open() {
        //open database connection logic
    }
    
    public func close() {
        
    }
}

class MysqlConnection : DatabaseConnection {
    public func open() {
        //open database connection logic
    }
    
    public func close() {
        
    }
}

class DAO {
    public let databaseConnection : DatabaseConnection;
    
    init(databaseConnection : DatabaseConnection) {
        self.databaseConnection = databaseConnection;
    }
    
    public func commitChanges() {
        
        databaseConnection.open();
        
        //deal with data base changes
        
        databaseConnection.close();
    }
}
```

咋一看，细节上可能跟Liskov要求的很像，但是请注意，DIP主要强调的是模块之间的依赖。下面用一个例子来感受一下

![](http://ot51d7lis.bkt.clouddn.com/DIP%20%281%29.png)
