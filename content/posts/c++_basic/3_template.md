+++
title = "3-template-object-model-cpp"
author = ["zyy"]
date = 2025-03-30T14:05:00+08:00
tags = ["c++", "language"]
categories = ["object_model"]
draft = false
showtoc = true
+++

## Template {#template}

The template will be compiled successfully, but when called by user, it will be compiled again which might cause error. And the most important thing is operator overload function.


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

The compiler will compile the template, when you call this template function it will compile the code again.


### member template {#member-template}

in a simple way to clarify that a template in template.

```c++
template<class T1, class T2>
struct pair{
  typedef T1 first_type;
  typedef T2 second_type;
  T1 first;
  T2 second;
  pair() :first(T1()), second(T2()) {}
  pair(const T1& a, const T2& b) :first(a), second(b) {}

  // ctor overload
  template<class U1, class U2>
  pair(const pair<U1, U2>& p) : first(p.first), second(p.second) {};
};
```

Here is an example like we have two types of class and their derived classes. The question is that can we assign to it ?

```c++
class Base1 {};
class Derived1: public Base1{};
class Base2 {};
class Derived2: public Base2{};

     //T1         T2
pair<Derived1, Derived2> p;
pair<Base1, Base2> p2(p);

     //T1         T2        U1        U2
pair<Base1, Base2> p2(pair<Derived1, Derived2>());
```

 **pair(const pair&lt;U1, U2&gt;&amp; p) : first(p.first), second(p.second) {}** indicates that U1 is assigned to T1 also for U2 and T2.
Can the assignment succeed or not?
Let's use an example in life. The sparrow derived from bird, and sparrow is a bird (inheritance). Fine! From this example we know that derived class can be assigned to base class which means the derived class can be converted to base class. But on the opposite it's not allowed.

The reason in STL using member template is to make the STL more elastic.


### In STL member template {#in-stl-member-template}

```c++
template<typename _Tp>
class shared_ptr : public __shared_ptr<_Tp>
{

  template<typename _Tp1>
  explicit shared_ptr(_Tp1 *__p)
    :__shared_ptr<_Tp>(__p) {}
};

// smart pointer
Base *ptr = new Derived1; // up-cast
shared_ptr<Base> sptr(new Derived); // simulate up-cast
```

Base class pointer can point to Derived class which is easy and rational to understand. So the shared_ptr must have this attribute so member template matters.


## Specialization {#specialization}


### template specialization (full specialization) {#template-specialization--full-specialization}

The template specialization is used for special design of template, we all know that the template is used for generalization. Okay, here is an example!

```c++
template<class Key>
struct hash{};

template<>
struct hash<char>{
  size_t operator() (char x) const { return x; }
}


template<>
struct hash<int>{
  size_t operator() (int x) const { return x; }
}

template<>
struct hash<long>{
  size_t operator() (long x) const { return x; }
}

cout << hash<long>()(100);
```

In the example above, we have not only generalization but also specialization, when we instance a new object passing type like( char, int, long), the compiler will compile the specialization template related, if passed other types, the compiler will use the generalization template. It's useful in some cases.


### partial specialization {#partial-specialization}

The partial specialization includes two types like numeric partial specialization and scope partial specialization.
Firstly let's talk about the numeric partial specialization. Here is the example.

```c++
template<typename T, typename Alloc=...>
class vector
{


}

template<typename Alloc=...>
class vector<bool, Alloc>
{

  ..

}
```

template argument of the vector is two, T and Alloc, numeric partial specialization means that I use only one argument Alloc in this example. That's beacause that we all know that to store the bool type we can use bit of char or something else, it's not necessary to store the bool type in a generalization in case of reduce memory usage. Note, the template argument cannot be only two may be more but when you choose the numeric partial specialization you have to follow the sequences of the argument which means you cannot jump one of the two.

Another type of partial specialization is scope partial specialization, here is the example.

```c++
template<typename T>
class C
{

};

template<typename T>       ->  template <typename U>
class C<T*>                    class C<U*>
{                              {

};                             };

C<string> obj1;
C<string*> obj1;
```

The scope in this example means that a pointer pointing an instance and an instance.The pointer means narrow the scope actually.


## template template parameter {#template-template-parameter}

tempate template parameter means that among the template arguments there is one template like argument.Which in my opinoin is hard to understand.Here is an example.

```c++
template<typename T,
      template <typename T>
      class Container
      >
class XCls
{
  private:
  Container<T> c;
};

template<typename T>
using Lst = list<T, allocator<T>>;

XCLs<string, list> mylist1; // error
XCLs<string, Lst> mylist2;
```

In this example above, I wanna use linked list to store the string, so we need another argument container is template. However the template template argument needs special sanity to pass **using.** The container need two arguments, but the smartptr needs only one argument.
Note, only in this condition, the **class** and **typename** is the same meaning.


## variadic templates( since c++ 11) {#variadic-templates--since-c-plus-plus-11}

the template argument number maybe be unsure, so since c++11, a new feature comes that you can divide the template arguments into one argument and the other is one packet using the **...**. Here is the example.

```c++
void print()
{
}
template<typename T, typename... Types>
void print(const T& fristarg, const Types&... args)
{
  cout << firstarg << endl;
  print(args...);
}
```

pack three ways to add **...**

-   template argument
-   function parameter types
-   function arguments

Don't forget the ... in the template argument list and function parameters, inside of the function, if you use sizeof()... you can access the number of arguments. As you can see, this example use the print recusively so you need to prepare an empty print function to end the calling.
