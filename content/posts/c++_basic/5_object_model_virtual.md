+++
title = "5-vptr-vbtl-object-model-cpp"
author = ["zyy"]
date = 2025-03-30T14:27:00+08:00
tags = ["c++", "language"]
categories = ["object_model"]
draft = false
showtoc = true
+++

## vptr and vtbl {#vptr-and-vtbl}


### outline {#outline}

Let's use an example to illustrate the c++ object model (aka vptr and vbtl).

```c++
class A{
  public:
      virtual void vfunc1();
      virtual void vfunc2();
      void func1();
      void func2();
 private:
      int m_data1, m_data2;
}


class B: public A{
  public:
      virtual void vfunc1();
      void func2();

 private:
      int m_data3;

}
class C: public B{
  public:
      virtual void vfunc1();
      void func1();
  private:
      int m_data1, m_data4;
}
```

If we create an object, the sizeof the object is member data variable. But if the class has at least one virtual function it will add another variable called vptr in object.
In this example above, the inheritance of class and memory is shown below.
![](/c_plus_plus/images/5_vptr_vbtl.png)
In each class B and class C, all override the virtual function 1, so each one is really different in memory. But B and C inherit from A, so all objects of B and C have vbtl entry pointing to A::func2. The picture shown above is really clear to see that.
The vptr in another side means that inheritance not only inherits data but also inherits access of function calls.


### dynamic binding {#dynamic-binding}

The dynamic binding or polymorphism is effective when meeting three conditions

-   pinter use
-   satisfy up-cast (aka pointed to derived class)
-   virtual function

And the final call of the dynamic binding is like, the compiler will do to generate code like this when the dynamic binding effects. n is computed by compiler and p is pointer.

```c++

(*(p->vptr)[n])(p);
// or
(* p->vptr[n] )(p);
```

Here is the example to illustrate that.
Conditions, C derived from B, B derived from A.

```c++
B b;
A a = (A)b;
a.vfunc1(); // static binding  in asm tail / call instruction

A *pa = new B;
pa->vfunc1(); // dynamic binding in asm using vbtl

// (* pa->vptr[n])(pa)
```

Using object to call virtual function is not dynamic binding.


## const {#const}

|                            | const object | non-const object |
|----------------------------|--------------|------------------|
| const member funcitons     |              |                  |
| (guarantee data members    | +            | +                |
| non-const member functions | -            | +                |
| (cannot guarantee0         |              |                  |

Note, when the non-const and const member functions occur at the same time in the class, the const object cannot access the non-const member function only const member function.


### example in STL {#example-in-stl}

```c++
charT
operator [](size_type pos) const
{ ... }

reference
operator [](size_type pos) const
{ ... }
```

s[5] = 'A' may trigger of COW(copy on write), so we have to handle that.
The basic_string adopts the shared-method design pattern, which means sometimes you may don't have to change the value of string, if you do so when offer that and copy another one for you. This is reference counting method.
