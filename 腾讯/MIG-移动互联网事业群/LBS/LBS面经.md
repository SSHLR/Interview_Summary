# 腾讯LBS面经
---

## 1. 自我介绍
 　　略过
## 2. 什么是**多态**，C++中的实现方式有哪些?
> 当时答的是通过重载和虚函数来实现多态，回来后查发现重载和多态的关系说法不一，一部分人认为重载和多态没一分钱关系，另一部分认为多态分为**类多态**和**函数多态**，而重载实现的是函数多态，最终也没看到比较权威的说法，暂将重载看作与多态没有关系。如果有哪位对此比较熟悉，请告知修改。谢谢~

  一个接口，多种方法。概括即为： **允许将子类类型的指针赋值给父类类型的指针**。赋值之后，父对象可以根据当前赋值给它的自对象的特性以不同的方式运作。
> 不要犯傻，如果它不是晚绑定，它就不是多态。 -- Bruce Eckel

  为了支持C++的多态性，引入了动态绑定和静态绑定。为了更好的理解动态绑定和静态绑定，首先谈一下**对象的类型**:
```
//对象的静态类型：对象在声明时采用的类型。是在编译期确定的。
//对象的动态类型：目前所指对象的类型。是在运行期决定的。对象的动态类型可以更改，但是静态类型无法更改。
class B
{
};
class C : public B
{
};
class D : public B
{
};
D* pD = new D();//pD的静态类型是它声明的类型D*，动态类型也是D*
B* pB = pD;//pB的静态类型是它声明的类型B*，动态类型是pB所指向的对象pD的类型D*
C* pC = new C();
pB = pC;//pB的动态类型是可以更改的，现在它的动态类型是C*
```
下面是动态绑定和静态绑定：
```
//静态绑定：绑定的是对象的静态类型，某特性（比如函数）依赖于对象的静态类型，发生在编译期。
//动态绑定：绑定的是对象的动态类型，某特性（比如函数）依赖于对象的动态类型，发生在运行期。
class B
{
    void DoSomething();
    virtual void vfun();
};
class C : public B
{
    void DoSomething();
    //首先说明一下，这个子类重新定义了父类的no-virtual函数，这是一个不好的设计，会导致名称遮掩；
	//这里只是为了说明动态绑定和静态绑定才这样使用。
    virtual void vfun();
};
class D : public B
{
    void DoSomething();
    virtual void vfun();
};
D* pD = new D();
B* pB = pD;
```
##### *pD->DoSomething()*和*pB->DoSomething()*调用的是同一个函数吗？ 　　　　不是。
虽然pD和pB都指向同一个对象。因为函数DoSomething是一个no-virtual函数，它是** *静态绑定的，也就是编译器会在编译期根据对象的静态类型来选择函数* **。pD的静态类型是D\*，那么编译器在处理pD->DoSomething()的时候会将它指向D::DoSomething()。同理，pB的静态类型是B\*，那pB->DoSomething()调用的就是B::DoSomething()。
##### *pD->vfun()*和*pB->vfun()*调用的是同一个函数吗？ 　　　　是的。
因为vfun是一个虚函数，它是动态绑定的，也就是说它绑定的是对象的动态类型，pB和pD虽然静态类型不同，但是他们同时指向一个对象，他们的动态类型是相同的，都是D\*，所以，他们的调用的是同一个函数：D::vfun()。
上面都是针对对象指针的情况，对于引用（reference）的情况同样适用。
		 指针和引用的动态类型和静态类型可能会不一致，但是对象的动态类型和静态类型是一致的。
D D;
D.DoSomething()和D.vfun()永远调用的都是D::DoSomething()和D::vfun()。
		只有虚函数才使用的是动态绑定，其他的全部是静态绑定。

## 3. 动态绑定的过程?
C++ 中，通过基类的引用或指针调用虚函数时，发生动态绑定。引用（或指针）既可以指向基类对象也可以指向派生类对象，这是动态绑定的关键。用引用（或指针）调用的虚函数在**运行时**确定，被调用的函数是引用（或指针）所指对象的实际类型（**即对象的动态类型**）所定义的。
### 联编
联编是指一个计算机程序自身彼此关联的过程，在这个联编过程中，需要确定程序中的操作调用(函数调用)与执行该操作(函数)的代码段之间的映射关系;按照联编所进行的阶段不同,可分为静态联编和动态联编；
* **静态联编**：是指联编工作是在程序**编译连接**阶段进行的，这种联编又称为**早期联编**；因为这种联编是在程序开始运行之前完成的；在程序编译阶段进行的这种联编又称静态绑定；在编译时就解决了程序中的操作调用与执行该操作代码间的关系，确定这种关系又被称为绑定；编译时绑定又称为静态绑定；
* **动态联编**：编译程序在编译阶段并不能确切地知道将要调用的函数，只有在程序执行时才能确定将要调用的函数，为此要确切地知道将要调用的函数，要求联编工作在程序运行时进行，这种在程序运行时进行的联编工作被称为动态联编，或动态绑定，又叫晚期联编；C++ 规定：动态联编是在虚函数的支持下实现的；
静态联编和动态联编都是属于多态性的 , 它们是在不同的阶段进对不同的实现进行不同的选择。

### 虚函数表
* ####无覆盖时的虚函数表
C++ 中动态绑定是通过虚函数实现的。而虚函数是通过一张虚函数表（ virtual table ）实现的。**这个表中记录了虚函数的地址，解决继承、覆盖的问题，保证动态绑定时能够根据对象的实际类型调用正确的函数。**
先说这个虚函数表。据说在 C++ 的标准规格说明书中说到，编译器必需要保证虚函数表的指针存在于对象实例中**最前面的位置**（这是为了保证正确取到虚函数的偏移量）。这意味着我们通过对象实例的地址得到这张虚函数表，然后就可以遍历其中函数指针，并调用相应的函数。
```
//假设有如下基类
class Base
{
public:
	virtual void f(){cout<<"Base::f"<<endl;}
	virtual void g(){cout<<"Base::g"<<endl;}
	virtual void h(){cout<<"Base::h"<<endl;}
};
void t1()
{
    typedef void (*pFun)(void);
    Base b;
    pFun pf=0;
    int * p = (int*)(&b);///强制转换为int*，这样就取得
    ///b的vptr的地址
    cout<<"VTable addr: "<<p<<endl;
    int *q;
    q=(int*)*p;///*p取得vtable，强转为int*，
    ///取得指向第一个函数地址的指针q
    cout<<"virtual table addr: "<<q<<endl;
    pf = (pFun)*q;///对指针解引用，取得函数地址
    cout<<"first virtual fun addr: "<<(int*)(*q)<<endl;
    pf();///invoke
}
```
运行结果：
![运行结果](http://i.imgur.com/jlskMQR.jpg)
按照同样的方法，继续调用 g() 和 h() ，只要移动指针即可
```
void t2()
{
    typedef void (*pFun)(void);
    Base b;
    pFun pf=0;
    int * p = (int*)(&b);///强制转换为int*，这样就取得
    ///b的vptr的地址
    cout<<"VTable addr: "<<p<<endl;
    int *q;
    q=(int*)*p;///*p取得vtable，强转为int*，
    ///取得指向第一个函数地址的指针q
    cout<<"virtual table addr: "<<q<<endl;
    pf = (pFun)*q;///对指针解引用，取得函数地址
    cout<<"first virtual fun addr: "<<(int*)(*q)<<endl;
    pf();///invoke
    ++q;///q指向第二个虚函数
    pf = (pFun)*q;///对指针解引用，取得函数地址
    cout<<"second virtual fun addr: "<<(int*)(*q)<<endl;
    pf();///invoke
    ++q;///q指向第3个虚函数
    pf = (pFun)*q;///对指针解引用，取得函数地址
    cout<<"third virtual fun addr: "<<(int*)(*q)<<endl;
    pf();///invoke
    /** ============================  **/
    ++q;///q指向第4个虚函数?
    cout<<"4th virtual fun addr: "<<(int*)(*q)<<endl;
}
```
分割线以后的那部分说明：显然只有 3 个虚函数，再往后移动就没有了，那没有是什么？取出来看发现是 0 ，如果把这个地址继续当做函数地址调用，那必然出错了。看来 **这个表的最后一个地址为 0 表示虚函数表的结束** 。结果如下：
![](http://i.imgur.com/cNQ841Z.jpg)
虚函数表如图，其中标 * 的地方这里是 0 。：
![](http://i.imgur.com/ide4BV5.jpg)
* #### 有覆盖时的虚函数表
没有覆盖的虚函数没有太大意义，要实现动态绑定，必须在派生类中覆盖基类的虚函数。
 1. ###### 继承时没有覆盖
  假设有如下继承关系：
![](http://i.imgur.com/REJ7b1v.jpg)
在这里，派生类没有覆盖任何基类函数。那么在派生类的实例中，其虚函数表如下所示：
![](http://i.imgur.com/spt3ePU.jpg)
测试代码也很简单，只要修改对象的地址为新的地址，然后继续往后移动指针就行了：
```
void t3()
{
    typedef void (*pFun)(void);
    Derive d;
    pFun pf=0;
    int * p = (int*)(&d);///强制转换为int*，这样就取得
    ///d的vptr的地址
    cout<<"VTable addr: "<<p<<endl;
    int *q;
    q=(int*)*p;///*p取得vtable，强转为int*，
    ///取得指向第一个函数地址的指针q
    cout<<"virtual table addr: "<<q<<endl;
    for(int i=0; i<6; ++i)
    {
        pf = (pFun)*q;///对指针解引用，取得函数地址
        cout<<i+1<<"th virtual fun addr: "<<(int*)(*q)<<endl;
        pf();///invoke
        ++q;
    }
    cout<<"*q="<<(*q)<<endl;
}
```
![](http://i.imgur.com/LfFlRTu.jpg)
 结论：
	* 虚函数按照其声明顺序放在 VTable 中；
	* 基类的虚函数在派生类虚函数前面；

 2. ###### 继承时有虚函数覆盖
 假设继承关系如下：
![](http://i.imgur.com/tBey3Mn.jpg)
这时函数 f() 在派生类中重写了。先测试下，看看虚函数表是什么样子的：测试代码：
```
void t4()
{
    typedef void (*pFun)(void);
    Derive d;
    pFun pf=0;
    int * p = (int*)(&d);///强制转换为int\*，这样就取得
    ///b的vptr的地址
    cout<<"VTable addr: "<<p<<endl;
    int *q;
    q=(int*)*p;///*p取得vtable，强转为int*，
    ///取得指向第一个函数地址的指针q
    cout<<"virtual table addr: "<<q<<endl;
    for(int i=0; i<6; ++i)
    {
        pf = (pFun)*q;///对指针解引用，取得函数地址
        if(pf==0) break;
        cout<<i+1<<"th virtual fun addr: "<<(int*)(*q)<<endl;
        pf();///invoke
        ++q;
    }
    cout<<"*q="<<(*q)<<endl;
}
```

结果：
![](http://i.imgur.com/IYZW5q9.jpg)
可以看到，第一个输出的是Derive::f，然后是Base::g、Base::h、Derive::g1、Derive::h1、 0 。据此，虚函数表就明了了：
![](http://i.imgur.com/OkqTQeL.jpg)
结论：
	* 派生类中重写的函数 f() 覆盖了基类的函数 f() ，并放在 虚函数表中 原来基类中 f() 的位置。
	* 没有覆盖的函数位置不变。
这样，对于下面的调用：
```
Base *b;
b = new Derive();
b->f();///Derive::f
```
由于 b 指向的对象的虚函数表的位置已经是 Derive::f() （就是上面的那个图），实际调用时，调用的就是 Derive::f() ，这就实现了多态。
 3. ###### 多重继承（无虚函数覆盖）
继承关系如下：
![](http://i.imgur.com/FX13R8G.jpg)
对于派生类实例中的虚函数表，如下图所示(图中写函数的地方实际为指向该函数的指针 )：
![](http://i.imgur.com/1Fm1CIR.jpg)
测试代码：
```
void t6()
{
    typedef void (*pFun)(void);
    Derive d;
    pFun pf=0;
    int * p = (int*)(&d);///强制转换为int*，这样就取得
    ///b的vptr的地址
    cout<<"Object addr: "<<p<<endl;
    for(int j=0; j<3; j++)
    {
        int *q;
        q=(int*)*p;///*p取得vtable，强转为int*，
        ///取得指向第一个函数地址的指针q
        cout<<j+1<<"th virtual table addr: "<<(int*)q<<endl;
        for(int i=0; i<6; ++i)
        {
            pf = (pFun)*q;///对指针解引用，取得函数地址
            if((int)pf<=0) break;///到末尾了
            cout<<i+1<<"th virtual fun addr: "<<(int*)(*q)<<endl;
            pf();///invoke
            ++q;
        }
        cout<<"*q="<<(*q)<<"\n"<<endl;
        p++;///下一个vptr
    }
}
```
结果验证：
![](http://i.imgur.com/XKB3BiT.jpg)
从上面的运行结果可以看出，第一个虚函数表的末尾不再是 0 了，而是一个负数，第二个变成一个更小的负数，最后一个 0 表示结束。【这个应该和编译器版本有关，看别人的只要没有结束都是 1 ，结束时是 0 。在本机上测试 vc6.0 每个虚函数表都是以 0 为结尾】
结论：
  * 每个基类都有自己的虚表；
	* 派生类虚函数放在第一个虚表的后半部分。
如果这时候运行如下代码：
```
b->f();
```
结果为： Base1::f ，因为在名字查找时，最先找到 Base1::f 后不再继续查找，然后类型检查没错，就调用这个了。
 4. ##### 多重继承（有虚函数覆盖）
继承关系修改为：
![](http://i.imgur.com/dgB3rKM.jpg)
这时派生类实例的虚函数表为：
![](http://i.imgur.com/TWVbdmA.jpg)
验证代码t6()。结果：
![](http://i.imgur.com/jrRq3cd.jpg)
结论：
	* 基类中的虚函数被替换为派生类的函数。

   但是为什么 3 个 Derive::f 的地址为什么不一样（见下图红框标注的部分）？
![](http://i.imgur.com/63LwS4s.jpg)
上述结果的完整代码:
```
class Base1
{
public:
    	virtual void f(){cout<<"Base1::f"<<endl;}
    	virtual void g(){cout<<"Base1::g"<<endl;}
    	virtual void h(){cout<<"Base1::h"<<endl;}
};
class Base2
{
public:
    	virtual void f(){cout<<"Base2::f"<<endl;}
    	virtual void g(){cout<<"Base2::g"<<endl;}
    	virtual void h(){cout<<"Base2::h"<<endl;}
};
class Base3
{
public:
    	virtual void f(){cout<<"Base3::f"<<endl;}
    	virtual void g(){cout<<"Base3::g"<<endl;}
    	virtual void h(){cout<<"Base3::h"<<endl;}
};
class Derive : public Base1, public Base2,public Base3
{
public:
    	virtual void f(){cout<<"Derive::f"<<endl;}
    	virtual void g1(){cout<<"Derive::g1"<<endl;}
    	//virtual void h1(){cout<<"Derive::h1"<<endl;}
};
void t6()
{
	    typedef void (*pFun)(void);
	    Derive d;
	    pFun pf=0;
	    int * p = (int*)(&d);///强制转换为int*，这样就取得
	    ///b的vptr的地址
	    cout<<"Object addr: "<<p<<endl;
	    for(int j=0; j<3; j++)
	    {
	        int *q;
	        q=(int*)*p;///*p取得vtable，强转为int*，
	        ///取得指向第一个函数地址的指针q
	        cout<<j+1<<"th virtual table addr: "<<(int*)q<<endl;
	        for(int i=0; i<6; ++i)
	        {
	            pf = (pFun)*q;///对指针解引用，取得函数地址
	            if((int)pf<=0) break;//
	            cout<<i+1<<"th virtual fun addr: "<<(int*)(pf)<<endl;
	            pf();///invoke
	            ++q;
	        }
	        cout<<"*q="<<(*q)<<"\n"<<endl;
	        p++;
	    }
}
int main()
{
		t6();
		return 0;
}
```
这时，如果使用基类指针去调用相关函数，那么实际运行时将根据指针指向的实际类型调用相关函数了：
```
Derive d;
    Base1 *b1 = &d;
    Base2 *b2 = &d;
    Base3 *b3 = &d;
    b1->f();///Derive::f
    b2->f();///Derive::f
    b3->f();///Derive::f
    b1->g();///Base1::f
    b2->g();///Base2::f
    b3->g();///Base3::f
```

* ### 继承与名字查找规则
 * 规则0. 名字查找在编译时发生
 * 规则1. 与基类同名的派生类成员将屏蔽对基类成员的直接访问。
   如果一定要访问，则必须使用作用域操作符限定访问基类成员。一般来说，派生类中重新定义的成员最好不要和基类中的成员同名。
 * 规则2. 基类和派生类中使用同一名字的成员函数，其行为和数据成员一样。
   即使函数原型不同，基类成员也会被屏蔽。
```
    struct Base  
    {  
        int  f();  
    };  
    struct Derived: Base  
    {  
        int f(int);  
    };  

    Derived d;  
    Base  b;  
    b.f();<span style="white-space:pre">    </span>//ok, Base::f()  
    d.f(100);  //ok  Derived::f(int)  
    d.f();//error  no Derived::f()  
    d.Base::f(); //ok Base::f()  
```
d.f()调用出错的原因是：编译器在Derived中查找名字f,一旦找到该名字，就不在继续查找了。然后进行参数类型检查，发现类型错误，于是报错。
 * 规则3. 如果在派生类中对基类的虚函数进行重写，则原型必须完全一样。否则就会出现2中的情况，并没有真正实现多态。
 * 规则4. 函数调用遵循以下四个步奏：
   1. 首先确定进行函数调用的对象、引用或指针的静态类型；
   2. 在该类中查找函数，如果找不到，就在直接基类中查找，如此沿着类的继承链往上找，直到找到该函数或者查找完最后一个类。如果不能找到该名字，则调用出错。
   3. 一旦找到该名字，就进行常规类型检查，如果匹配，则调用合法；如果类型不匹配，则报错。（停止继续查找）
   4. 假设函数调用合法，编译器就生成代码。如果是虚函数且通过引用或者指针调用，则编译器生成代码以确定根据对象的动态类型运行哪个函数版本，否则编译器生成代码直接调用函数。

* ### p.s.
	* **通过基类类型的指针访问派生类自己的虚函数**
虽然在上面的继承关系图中可以看到 Base1  的虚表中有 Derive 的虚函数，但是以下语句是非法的：
```
Base1 *b1 = new Derive();
b1->g1();/// 编译错误： Base1 没有成员 g1
```
如果一定要访问，只能通过上面的强制转换指针类型来完成了。
	* **访问非 public 成员**
把基类和派生类的成员 f 、 h 这些都改成 private 的，你会发现上述**通过指针访问成员函数没有任何问题**。
**这就是 C++!**

## 4. 重载、覆盖、隐藏三者的区别?
* **覆盖**也称**重写**，英文override
 * 特点：
   1. 不同的范围（分别位于派生类与基类）；
   2. 函数名、参数均完全相同；
   3. 基类函数必须有virtual 关键字；
 * 作用：
   基类指针和引用在调用对应方法时，根据所指对象类型实现动态绑定。
* **重载（overload）**
 * 特点：
   1. 相同的范围（在同一个类中）；
   2. 函数名相同，但是**参数类型**或者**个数**等不完全相同；
   3. virtual 关键字可有可无；
 * 作用：
   同一方法，根据传递消息的不同（类型或个数），产生不同的动作（相同方法名，实现不同）。
* **隐藏（遮蔽）**是指派生类的函数屏蔽了与其同名的基类函数
 * 特点：
   不同作用域，基类和派生之间
   分两种情形：
   1. 基类和派生类**函数名相同**，但**参数列表不同**，**此时不管有无virtual**，基类函数在派生类中被隐藏，派生类只能调用新的方法，不能调用已被隐藏的基类方法（**不同于重载，作用域不同**）
   2. 基类与派生类**函数名相同**，且**参数列表相同**，**但基类函数无virtual**，同样派生类中同样隐藏基类的同名同参函数（**不同于覆盖，无virtual**）

 分析：覆盖进行动态绑定，根据基类指针或引用指向的对象类型，调用相应的方法
隐藏进行静态绑定，取决于 调用的指针或应用类型，而非 基类指针或引用指向的对象类型
使用时，隐藏以产生混淆，应极力避免。

## 5. 虚继承是为了解决什么问题?
**作用**：为了解决从不同途径继承来的同名的数据成员在内存中有不同的拷贝造成数据不一致问题，将共同基类设置为虚基类。
这时从不同的路径继承过来的同名数据成员在内存中就只有一个拷贝，同一个函数名也只有一个映射。这样不仅解决
了二义性问题，也节省了内存，避免了数据不一致的问题。
**底层实现原理**：底层实现原理与编译器相关，一般通过虚基类指针实现，即各对象中只保存一份父类的对象，多继承时通过
虚基类指针引用该公共对象，从而避免菱形继承中的二义性问题。  
同时，虚继承与普通继承的区别有：
**假设derived 继承自base类，那么derived与base是一种“is a”的关系，即derived类是base类，而反之错误；
假设derived 虚继承自base类，那么derivd与base是一种“has a”的关系，即derived类有一个指向base类的vptr。**
因此,简单地**将虚继承理解为普通继承是有很大问题的**。接下来详细展开一下。

* ### 多重继承
首先我们考虑一个多重继承（非虚继承）的相对简单的例子。看看下面的C++类层次结构。
```
class Top
{
	public: int a;
};
class Left : public Top
{
	public: int b;
};
class Right : public Top
{
	public: int c;
};
class Bottom : public Left, public Right
{
	public: int d;
};
```
使用UML表示为：
![](http://i.imgur.com/NNQyFBO.png)
注意 **Top** 被继承了两次。这意味着类型Bottom的一个实例bottom将有两个叫做a的元素（分别为bottom.Left::a和bottom.Right::a）。
Left、Right和Bottom在内存中是如何布局的？让我们先看一个简单的例子。Left和Right拥有如下的结构：
|Left           | Right         |
| ------------- |:-------------:|
| Top::a        | Top::a        |
| Left::b       | Right::c      |
请注意第一个属性是从Top继承下来的。这意味着在下面两条语句后
```
Left* left = new Left();
Top* top = left;
```
left和top指向了同一地址，我们可以把Left Object当成Top Object来使用(很明显，Right与此也类似)。那Buttom呢？GCC的建议如下：
```
Bottom
Left::Top::a
Left::b
Right::Top::a
Right::c
Bottom::d
```
如果我们提升Bottom指针，会发生什么事呢？
```
Bottom* bottom = new Bottom();
Left* left = bottom;
```
这段代码工作正常。我们可以把一个Bottom的对象当作一个Left对象来使用，因为两个类的内存部局是一样的。那么，如果将其提升为Right呢？会发生什么事？
```
Right* right = bottom;
```
为了执行这条语句，我们需要判断指针的值以便让它指向Bottom中对应的段。
![](http://i.imgur.com/JKl6y0Y.png)
经过这一步，我们可以像操作正常Right对象一样使用right指针访问bottom。虽然，bottom与right现在指向两个不同的内存地址。出于完整性的缘故，思考一下执行下面这条语句时会出现什么状况。
```
Top* top = bottom;
```
此时，编译器将会报错。
```
error: `Top' is an ambiguous base of `Bottom'
```
两种方式可以避免这样的歧义
```
Top *topL = (Left *) bottom;
Top *topR = (Right *) bottom;
```
执行这两条语句后，topL和left会指向同样的地址，topR和right也会指向同样的地址。
* ### 虚拟继承
为了避免重复继承Top，我们必须虚拟继承Top：
```
class Top
{
	public: int a;
};
class Left : virtual public Top
{
	public: int b;
};
class Right : virtual public Top
{
	public: int c;
};
class Bottom : public Left, public Right
{
	public: int d;
};
```
这就得到了如下的层次结构
![](http://i.imgur.com/tecW37Y.png)
 虽然从程序员的角度看，这也许更加的明显和简便，但从编译器的角度看，这就变得非常的复杂。重新考虑下Bottom的布局，其中的一个（也许没有）可能是：
**Bottom**
Left::Top::a
Left::b
Right::c
Bottom::d
这个布局的优点是，布局的第一部分与Left的布局重叠了，这样我们就可以很容易的通过一个Left指针访问 Bottom类。可是我们怎么处理
```
Right *right = bottom;
```
我们将哪个地址赋给right呢? 经过这个赋值，如果right是指向一个普通的Right对象，我们应该就能使用 right了。但是这是不可能的！Right本身的内存布局是完全不同的，这样我们就无法像访问一个"真正的"Right对象一样，来访问升级的Bottom对象。而且，也没有其它（简单的）可以正常运作的Bottom布局。
解决办法是复杂的。我们先给出解决方案，之后再来解释它。
![](http://i.imgur.com/vxFBHTG.png)
你应该注意到了这个图中的两个地方。第一，字段的顺序是完全不同的（事实上，差不多是相反的）。第二，有几个vptr指针。这些属性是由编译器根据需要自动插入的（使用虚拟继承，或者使用虚拟函数的时候）。编译器也在构造器中插入了代码，来初始化这些指针。
vptr (virtual pointers)指向一个 “虚拟表”。类的每个虚拟基类都有一个vptr指针。要想知道这个虚拟表 (vtable)是怎样运用的，看看下面的C++ 代码。
```
Bottom* bottom = new Bottom();
Left* left = bottom; int p = left->a;
```
第二个赋值使left指向了bottom的所在地址（即，它指向了Bottom对象的“顶部”）。我们想想最后一条赋值语句的编译情况（稍微简化了）：
```
movl left, %eax # %eax = left
movl (%eax), %eax # %eax = left.vptr.Left
movl (%eax), %eax # %eax = virtual base offset
addl left, %eax # %eax = left + virtual base offset
movl (%eax), %eax # %eax = left.a
movl %eax, p # p = left.a
```
用语言来描述的话，就是我们用left指向虚拟表，并且由它获得了“虚拟基类偏移”(vbase)。这个偏移之后就加到了left，然后left就用来指向Bottom对象的Top部分。从这张图你可以看到Left的虚拟基类偏移是20；如果假设Bottom中的所有字段都是4个字节，那么给left加上20字节将会确实指向a字段。
经过这个设置，我们就可以同样的方法访问Right部分。按这样
```
Bottom* bottom = new Bottom();
Right* right = bottom; int p = right->a;
```
之后right将指向Bottom对象的合适的部位：
![](http://i.imgur.com/IoGxluo.png)
对top的赋值现在可以编译成像前面Left同样的方式。唯一的不同就是现在的vptr是指向了虚拟表的不同部位：取得的虚拟表偏移是12，这完全正确（确定！）。我们可以将其图示概括：
![](http://i.imgur.com/vRXA0rY.png)
当然，这个例子的目的就是要像访问真正Right对象一样访问升级的Bottom对象。因此，我们必须也要给Right（和Left）布局引入vptrs：
![](http://i.imgur.com/VmEC2Aj.png)
现在我们就可以通过一个Right指针，一点也不费事的访问Bottom对象了。不过，这是付出了相当大的代价：我们要引入虚拟表，类需要扩展一个或更多个虚拟指针，对一个对象的一个简单属性的查询现在需要两次间接的通过虚拟表（即使编译器某种程度上可以减小这个代价）。
* ### 向下转换
如我们所见，将一个派生类的指针转换为一个父类的指针(或者说，向上转换)可能涉及到给指针增添一个偏移。有人可能会想了，这样向下转换（反方向的）就可以简单的通过减去同样的偏移来实现。确实，对非虚拟继承来说是这样的。可是，虚拟继承（毫不奇怪的！）带来了另一种复杂性。
假设我们像下面这个类这样扩展继承层次。
```
class AnotherBottom : public Left, public Right
{
	public:
		int e; int f;
};
```
继承层次现在看起来是这样
![](http://i.imgur.com/rnZlqun.png)
现在考虑一下下面的代码。
```
Bottom* bottom1 = new Bottom();
AnotherBottom* bottom2 = new AnotherBottom();
Top* top1 = bottom1;
Top* top2 = bottom2;
Left* left = static_cast<Left*>(top1);
```
 下图显示了Bottom和AnotherBottom的布局，而且在最后一个赋值后面显示了指向top的指针。
![](http://i.imgur.com/2H1spc7.png)
现在考虑一下怎么去实现从top1到left的静态转换，同时要想到，我们并不知道top1是否指向一个Bottom类型的对象，或者是指向一个AnotherBottom类型的对象。所以这办不到！这个重要的偏移依赖于top1运行时的类型（Bottom则20，AnotherBottom则24）。编译器将报错：
```
error: cannot convert from base `Top' to derived type `Left'
via virtual base `Top'
```
因为我们需要运行时的信息，所以应该用一个动态转换来替代实现：
Left* left = dynamic_cast<Left*>(top1);
可是，编译器仍然不满意：
```
error: cannot dynamic_cast `top' (of type `class Top*') to type
   `class Left*' (source type is not polymorphic)
```
 （注：polymorphic多态的）

问题在于，动态转换（转换中使用到typeid）需要top1所指向对象的运行时类型信息。但是，如果你看看这张图，你就会发现，在top1指向的位置，我们仅仅只有一个integer (a)而已。编译器没有包含指向Top的虚拟指针，因为它不认为这是必需的。为了强制编译器包含进这个vptr指针，我们可以给Top增加一个虚拟的析构器：
```
class Top
{
	public:
		virtual ~Top() {}
		int a;
};
```
这个修改需要指向Top的vptr指针。Bottom的新布局是
![](http://i.imgur.com/NFiZHUm.png)
(当然类似的其它类也有一个新的指向Top的vptr指针)。现在编译器为动态转换插进了一个库调用：
```
left = __dynamic_cast(top1, typeinfo_for_Top, typeinfo_for_Left, -1);
```
这个函数__dynamic_cast定义在stdc++库中(相应的头文件是cxxabi.h)；参数为Top的类型信息，Left和Bottom(通过vptr.Top)，这个转换可以执行。 (参数 -1 标示出Left和Top之间的关系现在还是未知)。
* ### 虚拟基类的构造函数
编译器必须确保对象的所有虚指针都被正确的初始化。特别是，编译器确保了类的所有虚基类都被调用，并且只被调用一次。如果你不显示地调用虚拟父类(不管他们在继承层次结构中的距离有多远)，编译器都会自动地插入调用他们缺省构造函数。
 这样也会引来一些不可以预期的错误。以上面给出的类层次结构作为示例，并添加上构造函数的部分：
```
class Top
{
public:
	Top() { a = -1; }
	Top(int _a) { a = _a; }
	int a;
};
class Left : public Top
{
public:
	Left() { b = -2; }
	Left(int _a, int _b) : Top(_a) { b = _b; }
	int b;
};
class Right : public Top
{
public:
	Right() { c = -3; }
	Right(int _a, int _c) : Top(_a) { c = _c; }
	int c;
};
class Bottom : public Left, public Right
{
public:
	Bottom() { d = -4; }
	Bottom(int _a, int _b, int _c, int _d) : Left(_a, _b), Right(_a, _c)
	{
		d = _d;
	}
	int d;
};
```
首先考虑非虚拟的情况，下面的代码段输出什么：
```
Bottom bottom(1,2,3,4);
printf("%d %d %d %d %d\n", bottom.Left::a, bottom.Right::a, bottom.b, bottom.c, bottom.d);
```
结果是**1 1 2 3 4**，然而现在考虑虚拟的情况(虚拟继承自Top类)。并再一次运行程序，我们会得到：
**-1 -1 2 3 4**，为什么呢？通过跟踪构造函数的执行，会发现：
```
Top::Top()
Left::Left(1,2)
Right::Right(1,3)
Bottom::Bottom(1,2,3,4)
```
就像上面解释的一样，编译器在Bottom类执行其他构造函数之前已经插入并调用了缺省构造函数。 然后，**当Left去调用它自身的父类的构造函数时(Top),我们会发现Top已经被初始化了因此构造函数不会被调用**。
为了避免这种情况，你应该显示的调用虚基类的构造函数：
```
Bottom(int _a, int _b, int _c, int _d): Top(_a), Left(_a,_b), Right(_a,_c)
{
   d = _d;
}
```
* ### 观察这段代码，有什么问题
```
class Base
{
public:
    Base( int a ){ m_A = a; }
    int m_A;
};

class Derived_1: virtual public Base
{
public:
    Derived_1( void ): Base( 1 ){ }
};

class Derived_2: public Derived_1
{
public:
    Derived_2( void ): Derived_1( ){ }
};

int main( int argc, char** argv )
{
    Derived_2 derived;
}
```
**为避免构造函数重复运行，虚基类的构造是不由直接派生类负责的，改由最高派生类负责。**在这个例子中，最高派生类是derived_2，因此由它负责初始化Base，derived_1中的Base(1)实际上不起任何作用，即使在初始化列表中显式调用。
但在derived_2中没有调用Base的构造函数，于是编译器就试图调用Base的默认构造函数，不幸地，上面代码没有提供默认构造函数，而且由于我们自己提供了B的自定义构造函数，导致编译器也不会自动提供默认构造函数，因此编译结果会报找不到Base的默认构造函数的错误。
解决方法：或者在derived_2的成员初始化列表中显式调用Base的构造函数，或者在Base中提供默认构造函数，或者删除所有自定义构造函数。

## 6. 线程之间的同步机制有哪些?
* ### 什么是线程同步？
当使用多个线程来访问同一个数据时，非常容易出现线程安全问题(比如多个线程都在操作同一数据导致数据不一致),所以我们用同步机制来解决这些问题。

  **注意与进程间同步区别开来，一个是进程内通信，一个是进程间通信，大为不同。**

  **最常见的进程/线程的同步方法有 互斥锁（或称互斥量Mutex)，读写锁(rdlock)，条件变量(cond)，信号量(Semophore)等。在Windows系统中，临界区（Critical Section）和事件对象(Event)也是常用的同步方法。**
 1. **互斥锁**--保护了一个临界区，在这个临界区中，一次最多只能进入一个线程。如果有多个进程在同一个临界区内活动，就有可能产生竞态条件(race condition)导致错误，其中包含递归锁和非递归锁，（递归锁：同一个线程可以多次获得该锁，别的线程必须等该线程释放所有次数的锁才可以获得）。
 2. **读写锁**--从广义的逻辑上讲，也可以认为是一种共享版的互斥锁。可以多个线程同时进行读,但是写操作必须单独进行,不可多写和边读边写。如果对一个临界区大部分是读操作而只有少量的写操作，读写锁在一定程度上能够降低线程互斥产生的代价。
 3. **条件变量**--允许线程以一种无竞争的方式等待某个条件的发生。当该条件没有发生时，线程会一直处于休眠状态。当被其它线程通知条件已经发生时，线程才会被唤醒从而继续向下执行。条件变量是比较底层的同步原语，直接使用的情况不多，往往用于实现高层之间的线程同步。使用条件变量的一个经典的例子就是线程池(Thread Pool)了。
 4. **信号量**--通过精心设计信号量的PV操作，可以实现很复杂的进程同步情况（例如经典的哲学家就餐问题和理发店问题）。而现实的程序设计中，却极少有人使用信号量。能用信号量解决的问题似乎总能用其它更清晰更简洁的设计手段去代替信号量。
 5. **自旋锁**--当要获取一把自旋锁的时候又被别的线程持有时,不断循环的去检索是否可以获得自旋锁,一直占CPU资源。

  对于这些同步对象，有一些共同点：
   1. 每种类型的同步对象都有一个init的API，它完成该对象的初始化，在初始化过程中会分配该同步对象所需要的资源（注意是为支持这种锁而需要的资源，不包括表示同步对象的变量本身所需要的内存）
   2. 每种类型的同步对象都一个destory的API，它完成与init相反的工作
   3. 对于使用动态分配内存的同步对象，在使用它之前必须先调用init
   4. 在释放使用动态分配内存的同步对象所使用的内存时，必须先调用destory释放系统为其申请的资源
   5. 每种同步对象的默认作用范围都是进程内部的线程，但是可以通过修改其属性为PTHREAD_PROCESS_SHARED并在进程共享内存中创建它的方式使其作用范围跨越进程范围
   6. 无论是作用于进程内的线程，还是作用于不同进程间的线程，真正参与竞争的都是线程（对于不存在多个线程的进程来说就是其主线程），因而讨论都基于线程来
   7. 这些同步对象都是协作性质的，相当于一种君子协定，需要相关线程主动去使用，无法强制一个线程必须使用某个同步对象

  总体上来说，可以将它们分为两类：

     第一类是互斥锁、读写锁、自旋锁，它们主要是用来保护临界区的，也就是主要用于解决互斥问题的，当尝试上锁时大体上有两种情况下会返回：上锁成功或出错，它们不会因为出现信号而返回。另外解锁只能由锁的拥有者进行。

     第二类是条件变量和信号量，它们提供了异步通知的能力，因而可以用于同步和互斥。但是二者又有区别：
     * 信号量可以由发起P操作的线程发起V操作，也可以由其它线程发起V操作；但是条件变量一般要由其它线程发起signal（即唤醒）操作
     * 由于条件变量并没有包含任何需要检测的条件的信息，因而对这个条件需要用其它方式来保护，所以条件变量需要和互斥锁一起使用，而信号量本身就包含了相关的条件信息（一般是资源可用量），因而不需要和其它方式一起来使用
     * 类似于三种锁，信号量的P操作要么成功返回，要么失败返回，不会因而出现信号而返回；但是条件变量可能因为出现信号而返回，这也是因为它没包含相关的条件信息而导致的。
