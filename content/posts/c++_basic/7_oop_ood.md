+++
title = "7-oop-ood-object-oriented-cpp"
author = ["zyy"]
date = 2025-03-30T11:48:00+08:00
tags = ["c++", "language"]
categories = ["object_oriented"]
draft = false
showtoc = true
+++

## OOP and OOD {#oop-and-ood}

object oriented programming, object oriented design
We have three tools to design a good program, inheritance, composition, and delegation.


### Inheritance {#inheritance}


#### **is-a** {#is-a}

The public inheritance way is "is-a", like person is a creature. dog is a animal. etc.


#### outline {#outline}

![](/c_plus_plus/images/7_inheritance.png)
The same outline as composition.


#### ctor and dtor {#ctor-and-dtor}

The order of calling ctor when construct the class is from inner to outter, the dtor is the opposite.

```c++
Derived::Derived(...) : Base() {};
Derived::~Derived(...) : {... ~Base();};
```

Besides, the compiler will automatically add the ctor and dtor, if ctor does ease your tastes then you need to change you like.
Note: the base dtor must using **virtual** identifier, otherwise it will cause undefined behavior.


### composition {#composition}


#### **has-a** {#has-a}

```c++
template <class T>
class queue{
...
protected:
  deque<T> c;
public:
  bool empty() const { return c.empty;}
  size_type size() const { return c.size(); }
  reference front() { return c.front(); }
  reference back() { return c.back(); }

  void push( const value_type&x) { c.push_back(x);}
  void pop() { c.pop_front();}
};
```

The outline of the queue class is like,
![](/c_plus_plus/images/7_queue.png)


#### adapter design pattern {#adapter-design-pattern}

Also, it is a design pattern named Adapter, which means I have a great and more sophisticated class , and i wanna implement a simpler class, then I use the more complicated one to do it.


#### ctor and dtor {#ctor-and-dtor}

![](/c_plus_plus/images/7_composition.png)
The order of calling ctor when construct the class is from inner to outter, the dtor is the opposite.

```c++
Container::Container(...) : Component() {};
Container::~Container(...) : {... ~Component();};
```

Besides, the compiler will automatically add the ctor and dtor, if ctor does ease your tastes then you need to change you like.


### delegation {#delegation}


#### **composition by reference** {#composition-by-reference}

Handle / Body

```c++
// file String.hpp
class StringRep;
class String
{
public:
  String();
  String(const char* s);
  String(const String& str);
  ~String();
private:
  StringRep *rep;
};
// file String.cpp
#include "String.hpp"
namespace {
class StringRep
{
  friend class String;
  StringRep(const char *s);
  ~StringRep();
  int count;// reference counting
  char *rep;
};
}
String::String() {...}
```


#### outline {#outline}

The outline of delegation is like.
![](/c_plus_plus/images/7_delegation.png)
As you can see, a, b and c are sharing the same string "hello", if each of them wants to change , it will cause **COW** technique.
Another thing to notice is that, reference counting is necessary because of the sharing. That's we can know how many object are instanced before call dtor.
