## 设计模式 ##  
 - 单例模式 
	一个类只有一个对象被创建  
	1. 构造函数私有化，禁止外界创建实例    
	2. 线程安全  
	3. 禁止拷贝操作和移动操作  
	4. 一个公有静态成员函数访问它的唯一实例

 1.  懒汉式---配置文件  
		静态成员对象指针变量---线程安全问题  
		首次被访问时创建实例  
  
```C++  
#include <iostream>  
using namespace std;  
  
/*  
 4. 版本1 SingletonPattern_V1 存在以下两个问题  
 5.  
 6. 1. 线程不安全, 非线程安全版本  
 7. 2. 内存泄露  
*/  
class SingletonPattern_V1  
{  
private:  
SingletonPattern_V1() {  
cout << "constructor called!" << endl;  
}  
SingletonPattern_V1(SingletonPattern_V1&) = delete;  
SingletonPattern_V1& operator=(const SingletonPattern_V1&) = delete;  
static SingletonPattern_V1* m_pInstance;  
  
public:  
~SingletonPattern_V1() {  
cout << "destructor called!" << endl;  
}  
//在这里实例化  
static SingletonPattern_V1* Instance() {  
if (!m_pInstance) {  
m_pInstance = new SingletonPattern_V1();  
}  
return m_pInstance;  
}  
void use() const { cout << "in use" << endl; }  
};  
  
//在类外初始化静态变量  
SingletonPattern_V1* SingletonPattern_V1::m_pInstance = nullptr;  
  
//函数入口  
int main()  
{  
//测试  
SingletonPattern_V1* p1 = SingletonPattern_V1::Instance();  
SingletonPattern_V1* p2 = SingletonPattern_V1::Instance();  
  
system("pause");  
return 0;  
}  
```
```C++
#include <iostream>  
using namespace std;  
#include <memory> // C++11 shared_ptr头文件  
#include <mutex> // C++11 mutex头文件  
/*  
 8. 版本2 SingletonPattern_V2 解决了V1中的问题  
 9.  
 10. 1. 通过加锁让线程安全了  
 11. 2. 通过智能指针(shareptr 基于引用计数)内存没有泄露了  
*/  
class SingletonPattern_V2  
{  
public:  
~SingletonPattern_V2() {  
std::cout << "destructor called!" << std::endl;  
}  
SingletonPattern_V2(SingletonPattern_V2&) = delete;  
SingletonPattern_V2& operator=(const SingletonPattern_V2&) = delete;  
  
//在这里实例化  
static std::shared_ptr<SingletonPattern_V2> Instance()  
{  
//双重检查锁  
if (m_pInstance == nullptr) {  
std::lock_guard<std::mutex> lk(m_mutex);  
if (m_pInstance == nullptr) {  
m_pInstance = std::shared_ptr<SingletonPattern_V2>(new SingletonPattern_V2());  
}  
}  
return m_pInstance;  
}  
  
private:  
SingletonPattern_V2() {  
std::cout << "constructor called!" << std::endl;  
}  
static std::shared_ptr<SingletonPattern_V2> m_pInstance;  
static std::mutex m_mutex;  
};  
  
//在类外初始化静态变量  
std::shared_ptr<SingletonPattern_V2> SingletonPattern_V2::m_pInstance = nullptr;  
std::mutex SingletonPattern_V2::m_mutex;  
  
int main()  
{  
std::shared_ptr<SingletonPattern_V2> p1 = SingletonPattern_V2::Instance();  
std::shared_ptr<SingletonPattern_V2> p2 = SingletonPattern_V2::Instance();  
  
system("pause");  
return 0;  
}  
```  

2. 饿汉式  
  
	静态成员对象变量  
  
	实例开始即创建  
  
	C++11 Magic Static 如果当变量在初始化的时候，并发同时进入声明语句，并发线程将会阻塞等待初始化结束。  
  
```C++  
#include <iostream>  
using namespace std;  
/*  
 13. 版本3 SingletonPattern_V3 使用局部静态变量 解决了V2中使用智能指针和锁的问题  
 14.  
 15. 1. 代码简洁 无智能指针调用  
 16. 2. 也没有双重检查锁定模式的风险  
*/  
class SingletonPattern_V3  
{  
public:  
~SingletonPattern_V3() {  
std::cout << "destructor called!" << std::endl;  
}  
SingletonPattern_V3(const SingletonPattern_V3&) = delete;  
SingletonPattern_V3& operator=(const SingletonPattern_V3&) = delete;  
static SingletonPattern_V3& Instance() {  
static SingletonPattern_V3 m_pInstance;  
return m_pInstance;  
  
}  
private:  
SingletonPattern_V3() {  
std::cout << "constructor called!" << std::endl;  
}  
};  
  
int main()  
{  
SingletonPattern_V3& instance_1 = SingletonPattern_V3::Instance();  
SingletonPattern_V3& instance_2 = SingletonPattern_V3::Instance();  
  
system("pause");  
return 0;  
}  
```  
  
- 工厂模式
	生产者模型

1. 简单工厂模式
		直接生产产品，工厂类中做判断
		
```C++
enum CTYPE {COREA, COREB};   
class SingleCore  
{  
public:  
    virtual void Show() = 0;
};  
//单核A  
class SingleCoreA: public SingleCore  
{  
public:  
    void Show() { cout<<"SingleCore A"<<endl; }  
};  
//单核B  
class SingleCoreB: public SingleCore  
{  
public:  
    void Show() { cout<<"SingleCore B"<<endl; }  
};  
//唯一的工厂，可以生产两种型号的处理器核，在内部判断  
class Factory  
{  
public:   
    SingleCore* CreateSingleCore(enum CTYPE ctype)  
    {  
        if(ctype == COREA) //工厂内部判断  
            return new SingleCoreA(); //生产核A  
        else if(ctype == COREB)  
            return new SingleCoreB(); //生产核B  
        else  
            return NULL;  
    }  
};  
```

2. 工厂方法模式
		定义一个用于创建对象的接口，让子类决定实例化对象
		
```C++
class SingleCore  
{  
public:  
    virtual void Show() = 0;
};  
//单核A  
class SingleCoreA: public SingleCore  
{  
public:  
    void Show() { cout<<"SingleCore A"<<endl; }  
};  
//单核B  
class SingleCoreB: public SingleCore  
{  
public:  
    void Show() { cout<<"SingleCore B"<<endl; }  
};  
class Factory  
{  
public:  
    virtual SingleCore* CreateSingleCore() = 0;
};  
//生产A核的工厂  
class FactoryA: public Factory  
{  
public:  
    SingleCoreA* CreateSingleCore() { return new SingleCoreA; }  
};  
//生产B核的工厂  
class FactoryB: public Factory  
{  
public:  
    SingleCoreB* CreateSingleCore() { return new SingleCoreB; }  
};  
```

3.  抽象工厂模式

```C++
//单核  
class SingleCore   
{  
public:  
    virtual void Show() = 0;
};  
class SingleCoreA: public SingleCore    
{  
public:  
    void Show() { cout<<"Single Core A"<<endl; }  
};  
class SingleCoreB :public SingleCore  
{  
public:  
    void Show() { cout<<"Single Core B"<<endl; }  
};  
//多核  
class MultiCore    
{  
public:  
    virtual void Show() = 0;
};  
class MultiCoreA : public MultiCore    
{  
public:  
    void Show() { cout<<"Multi Core A"<<endl; }  
  
};  
class MultiCoreB : public MultiCore    
{  
public:  
    void Show() { cout<<"Multi Core B"<<endl; }  
};  
//工厂  
class CoreFactory    
{  
public:  
    virtual SingleCore* CreateSingleCore() = 0;
    virtual MultiCore* CreateMultiCore() = 0;
};  
//工厂A，专门用来生产A型号的处理器  
class FactoryA :public CoreFactory  
{  
public:  
    SingleCore* CreateSingleCore() { return new SingleCoreA(); }  
    MultiCore* CreateMultiCore() { return new MultiCoreA(); }  
};  
//工厂B，专门用来生产B型号的处理器  
class FactoryB : public CoreFactory  
{  
public:  
    SingleCore* CreateSingleCore() { return new SingleCoreB(); }  
    MultiCore* CreateMultiCore() { return new MultiCoreB(); }  
};
```

- 观察者模式
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTcyMjQ0NDc0NF19
-->