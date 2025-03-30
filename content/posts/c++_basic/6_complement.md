+++
title = "6-complement-objcet-oriented-cpp"
author = ["zyy"]
date = 2025-03-30T11:41:00+08:00
tags = ["c++", "language"]
categories = ["object_oriented"]
draft = false
showtoc = true
+++

## static {#static}


### static member data {#static-member-data}

only has one slice in memory, and need to be definition.


### static member function {#static-member-function}

The only way to access static member data is using static member function.Besides, there is no **this** pointer in static member function.

```c++
complex c1;
complex c2(2,1);
cout << c1.real();
cout << c2.imag();
--->this pointer.
cout << Complex::real(&c1);
cout << Complex::imag(&c2);
```

The member function only has one slice, it needs different address of object to access data.
![](/c_plus_plus/images/6_static_object.png)

Example using static:

```c++

class A{
public:
  static A& getInstance() { return a; }

  setup() { ... }

private:
  A();
  A(const A& rhs);
  static A a;
};

// good example
class A{
public:
  static A& getInstance();
  setup() { ... }

private:
  A();
  A(const A& rhs);
  ...
};
static A& A::getInstance(){
  static A a;
  return a;
}

...
A::getInstance().setup();
```

The first example is not good because no matter when you need a class named A, it is still be in memory. So instead of still in memory, we choose another one to create when you need one using static local object.

Example static,
two ways to access static function

1.  object
2.  class name

<!--listend-->

```c++
class Account
{
public:
  static double m_rate;
  //static double m_rate = 9;
  static void set_rate(const double& rate) { m_rate = rate;}

};
double Account::m_rate = 8;

int main(){
  Account::set_rate(8);

  Account a;
  a.set_rate(5);
}
```

Note, definition out of class is necessary because in class this is shown of declaration. but it's not allowed to define in class which will expose error.


## cout {#cout}

```c++
class _IO_ostream_withassign : public ostream{

};

extern _IO_ostream_withassign cout;
class ostream: virtual public ios
{
public:
  ostream& operator<< (int n);
  ostream& operator<< (unsigned int n);
  ostream& operator<< (char c);

...
};
```

cout is kind of ostream, and there are many **operator&lt;&lt;** overload function.


## template {#template}

The template will be compiled successfully, but when called by user, it will be compiled again which might cause error. And the most important thing is operator overload function. The aim of template is generalization.


### template class {#template-class}

```c++
template<typename T>
class complex
{

private:
  T re, ra;
}

....
complex<int> c1;
complex<double> c1;
```

When the compiler sees the Complex&lt;int&gt; or Complex&lt;double&gt; it will generate two slices of Complex has different type. This is kind of wasteful of memory but it's necessary.


### template function {#template-function}

```c++
stone r1(2,3), r2(2,0), r3;
r3 = min(r1, r2);
-->
template <class T>
inline
const T& min(const T& a, const T& b)
{
  return a < b ? a : b;
}
-->
class stone
{
public:
  ...

  bool operator < (const stone& t2) {
    return _weight < t2._weight;
  }
private:
  double _weight;
  double _height;
};
```

The compiler will do **argument deduction** to function template, which we don't have to add &lt;int&gt; or &lt;stone&gt; after the min function. After that, the min function will call the operator &lt; function. So the class designer must overload this function.

And the more details about template is in 3_template.org in object_mode directory


## namespace {#namespace}

```c++
namespace std{

}

using namespace std;
using std::cout;
```

Two ways to use it.

1.  using directive (valid all)
2.  using delcaration(valid some case)

Example

```c++
namespace jj01
{
  void test_member_template();
};

namespace jj02
{
  void test_member_template();
};

int main()
{
  jj01::test_member_template();
  jj02::test_member_template();
};
```

When using namespace with different departments, the conflict disappears which is really cooperative.
