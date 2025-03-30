+++
title = "3-operator-overload-object-oriented-cpp"
author = ["zyy"]
date = 2025-03-30T11:06:00+08:00
tags = ["c++", "language"]
categories = ["object_oriented"]
draft = false
showtoc = true
+++

## operator overload {#operator-overload}


### member function {#member-function}

the member function overload contains this pointer implicitly, but you don't have to pass the argument. you can use it freely.

```c++
inline complex&
__doapl(complex* ths, const complex& r)
{
  ths->re += r.re;
  ths->im += r.im;
  return *ths;
}

inline complex&
complex::operator += (const complex& r)
{
  return __doapl(this, r);
}


//...
complex c1(2,1);
complex c2(5);
c2 += c1;
c3 += c2 += c1;

```

why we need return by reference when we define **+=** overload function? If you just use c2 += c1; you can define void return type, but when there is a consecutive assignment which means the expression must return a type that can be passed to another +=. So **complex &amp;** matters. **c3 += c2 += c1**


### non-member function (global function) {#non-member-function--global-function}

There is no **this** pointer, it belongs to global function.

```c++
inline complex operator + (const complex &x, const complex &y)
{
  return complex (real(x) + real(y), imag(x) + imag(y));
}

inline complex operator + (const complex &x, double y)
{
  return complex (real(x) + y, imag(x));
}

inline complex operator + (double x, const complex &y)
{
  return complex (x + real(y) , imag(y));
}

//...
complex c1(2,1);
complex c2;

c2 = c1 + c2;
c2 = c1 + 5;
c2 = 7 + c1;
```

Besides, it used **temp object**--- **typename()**.There is no allocated storage provided, you have to return by value by creating a local object.

Other definitions outside the class.

```c++
inline complex  //inline complex& ok
operator + (const complex &x)
{
  return x;
}
inline complex
operator - (const complex &x)
{
  return complex(-real(x), -imag(x));
}
```

There is another question remaining that in +(positive function), there isn't newer things created, you can return by reference.

Another example:

```c++
inline complex
conj (const complex & x)
{
  return complex( real(x), -imag(x));
}

#include <iostream>
ostream&
operator << (ostream &os, const complex& x)
{
  return os << '(' << real(x) << ','
            << imag(x) << ')';
}
complex c1(2,1)
cout << conj(c1)
cout << c1 << conj(c1);
```

The operator&lt;&lt; overload function must be non-member function, because the complex is new than ostream and the first parameter is always an ostream object. It would have to be a member of the ostream class but we can not change).
And &lt;&lt; left operand is cout, right operand is complex x; return by reference is for sure in case of consecutive usage.
