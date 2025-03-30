+++
title = "1-basic-ctor-object-oriented-cpp"
author = ["zyy"]
date = 2025-03-30T10:09:00+08:00
tags = ["c++", "language"]
categories = ["object_oriented"]
draft = false
showtoc = true
+++

## C-plus-plus profile {#c-plus-plus-profile}


### object {#object}

-   object based
    -class without pointer members
     -Complex
    -class with pointer members
     -String
-   object oriented
    -inheritance
    -composition
    -delegation


### data and functions {#data-and-functions}

-   C type[data] [functions]----------------&gt;variable in memory
-   C++ class/struct [data, functions]------&gt;object in memory

Note: the function of class only has one copy which means when you create class only data occupies memory and vptr (if has virtual function inside class).


### C-plus-plus coding program {#c-plus-plus-coding-program}

.h/.cpp/.hpp never mind


### gurad declaration {#gurad-declaration}

```c++
#progam once
//
#ifndef __COMPLEX__
#define __COMPLEX__


#endif
```


### Header Layout {#header-layout}

1.  class-delcaration

<!--listend-->

```c++
class complex
{

}
```

1.  class-definition

<!--listend-->

```c++
complex::function ... ...
```

1.  forward-declarations

<!--listend-->

```c++
class complex;

```


### template {#template}

complex not only double real and image, maybe float or int

```c++
template<typename T>
class complex
{

private:
  T re, ra;
}

complex<int> c1;
complex<double> c1;
```


### inline {#inline}

when you define a function in class body

```c++

class complex
{
public:
  complex (double r= 0, double i= 0)
    : re(r), im(i)
    {}

  complex& operator+=(const complex&);
  double real() const{return re;}
  double imag() const{return im;}

private:
  double re, ra;

  friend complex& __doapl (complex*, const complex&);
}
```

The real/imag function will be canditate for inline functions automatically.Btw, the way it becomes inline function depends on the complier and the complexity of the function itself.


## Constructor function (ctor) {#constructor-function--ctor}


### access level {#access-level}

public
protected ( in which I don't give it to public interface and differs from private function , I can use it)
protected example in 8_virtual_polymorphism.org file.
private are not allowed by outside of the class

```c++

class complex
{
public:
  complex (double r= 0, double i= 0)
    : re(r), im(i)
    {}

  complex& operator+=(const complex&);
  double real() const{return re;}
  double imag() const{return im;}

private:
  double re, im;

  friend complex& __doapl (complex*, const complex&);
}

....
complex c1(2,1);
cout << c1.re; //not allowed
cout << c1.im; //not allowed

complex c1(2,1);
cout << c1.real();
cout << c1.imag();
```


### constructor (ctor) {#constructor--ctor}

1.  function name is class no return type
2.  default argument
3.  initialization list ( assignment not advised)

<!--listend-->

```c++
public:
  complex (double r= 0, double i= 0)
    : re(r), im(i)
    {}

```

initialization stage:

-   initialization **:re(r), im(i)**
-   assignment

when you use assignment to initialize private variable, it is allowed but low-efficiency

```c++
public:
  complex (double r= 0, double i= 0)
    { re = r; im = i;}
```


### ctor function overloading {#ctor-function-overloading}

That means ctor has multiple forms, actually the overloading function has different name viewed by compiler.

```c++

void real(double r ) {re = r;}

viewed by compiler,

?real@Complex@@QBENXZ
?real@Complex@@QAENABN@Z
```

however, when you define two ctor functions.

```c++

class complex
{
public:
  complex (double r= 0, double i= 0)
    : re(r), im(i)
    {}
  complex () : re(0), im(0) {}

...
complex c1;
complex c2();
```

These two functions are not allowed, that is because the compiler doesn't deccide which one to call when construct c1 and c2, i mean both the ctor functions are ok to called. It is defined ok but called error, and it will print messages like that below.

```shell
  complex_test.cpp:14:11: error: call of overloaded ‘complex()’ is ambiguous
   14 |   complex c1;
      |           ^~
In file included from complex_test.cpp:2:
complex.h:17:3: note: candidate: ‘complex::complex()’
   17 |   complex (): re (0), im (0) { }
      |   ^~~~~~~
complex.h:16:3: note: candidate: ‘complex::complex(double, double)’
   16 |   complex (double r = 0, double i = 0): re (r), im (i) { }
```


### brace initialization {#brace-initialization}

In the last example above, if you use **complex c2();** to declare an object, the compiler will think it might be a function declaration or object declaration which confuses the compiler. So, if you wanna use default ctor you can remove the **()**, or change it to **{}** braces.
complex c2{}; complex c2;

```shell
complex_test.cpp:14:13: warning: empty parentheses were disambiguated as a function declaration [-Wvexing-parse]
   14 |   complex c2();
      |             ^~
complex_test.cpp:14:13: note: remove parentheses to default-initialize a variable
   14 |   complex c2();
      |             ^~
      |             --
complex_test.cpp:14:13: note: or replace parentheses with braces to value-initialize a variable
```


### private ctor {#private-ctor}

Singleton - design pattern, means just only one instance of a class.

```c++
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


### const member function {#const-member-function}

Advice: member function

1.  might change the private value
2.  might not change the private value

The function which not change the private value need to add **const** identifier.
The position of const need to add after the **)** before the **{** symbol

```c++
double real() const{return re;}
double imag() const{return im;}

double real() {return re;}
double imag() {return im;}

// example
{
 const complex c1(2,1)
 cout << c1.real(); // error conflict!
 cout << c1.imag();
}
```
