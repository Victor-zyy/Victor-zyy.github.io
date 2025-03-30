+++
title = "1-explicit-typefunction-object-model-cpp"
author = ["zyy"]
date = 2025-03-30T13:44:00+08:00
tags = ["c++", "language"]
categories = ["object_model"]
draft = false
showtoc = true
+++

## conversion function {#conversion-function}

form: **operator typename()**

```c++
class Fraction
{
public:
  Fraction(int num, int den=1) : m_numerator(num), m_denominator(den) {}

  operator double() const{
    return (double) (m_numerator / m_denominator);
  }

private:
  int m_numerator; //fenzi
  int m_denominator;// fenmu
};

Fraction f(3,5);
double d = 4 + f;

```

The conversion function is aimed to let compiler know. When the compiler tries to compile the line double d = 4 + f; it will see if operator + (double, Fraction) is defined, or whether f is able to convert to double.
Finally, the compiler will try the latter one, convert f to double use the operator double() function in Fraction.
Actually you can write many conversion function as long as they are meaningful to the user.


## non-explicit-one-argument-ctor {#non-explicit-one-argument-ctor}

**explicit** keyword
This means that Fraction function only needs one passing argument but actually it has two parameters. That's because Fraction given a int number is meaningful when den=1 as a default argument.

```c++

class Fraction
{
public:
  Fraction(int num, int den=1) : m_numerator(num), m_denominator(den) {}

  Fraction operator+ (const Fraction& f){
      return Fraction(...);
  }
private:
  int m_numerator; //fenzi
  int m_denominator;// fenmu
};

Fraction f(3,5);
Fraction d = f + 4;

```

Fraction d = f + 4; The compiler will call non-explicit-one-argument ctor to convet 4 to **Fraction(4,1)**. And we have operator+ overload function so Fraction adds a Fraction is ok to compile even if we don't actually fully define this function.

The non-explicit ctor is the opposite of conversion function, like if you use conversion function you convert this type to another type, if you use non-explicit ctor , you convert other type to this type(ctor).


## Conflict {#conflict}

Btw, something ambiguous might happen when conflicts between conversion function and non-explicit ctor.

```c++
class Fraction
{
public:
  Fraction(int num, int den=1) : m_numerator(num), m_denominator(den) {}
  operator double() const {
    return (double) ( m_numerator / m_denominator);
  }
  Fraction operator+ (const Fraction& f){
    return Fraction(....);
  }
private:
  int m_numerator; //fenzi
  int m_denominator;// fenmu
};

Fraction f(3,5);
Fraction d = f + 4; // which will cause ambiguous

explicit.cc:21:18: error: ambiguous overload for ‘operator+’ (operand types are ‘Fraction’ and ‘int’)
   21 |   Fraction d = f + 4; // which will cause ambiguous
      |                ~ ^ ~
      |                |   |
      |                |   int
      |                Fraction
explicit.cc:21:18: note: candidate: ‘operator+(double, int)’ (built-in)
   21 |   Fraction d = f + 4; // which will cause ambiguous
      |                ~~^~~
explicit.cc:12:12: note: candidate: ‘Fraction Fraction::operator+(const Fraction&)’
   12 |   Fraction operator+ (const Fraction& f){
      |            ^~~~~~~~
```

The ambiguity exists in that overload operator+ function. operator+(double , int) and operator+(const Fraction&amp; f);


## explicit {#explicit}

The explicit means that you have to call ctor explicitly don't do implicitly in compile time.

```c++

class Fraction
{
public:
  explicit Fraction(int num, int den=1) : m_numerator(num), m_denominator(den) {}
  operator double() const {
    return (double) ( m_numerator / m_denominator);
  }
  Fraction operator+ (const Fraction& f){
    return Fraction(....);
  }
private:
  int m_numerator; //fenzi
  int m_denominator;// fenmu
};

Fraction f(3,5);
Fraction d = f + 4;
```

If we add explicit to ctor, then the 4 can't convert to Fraction implicitly, which causes the ambigurity disappear.
But the f+ 4 is a double type , and you assign a double type to a Fraction which cause conversion type error.

```sh
explicit.cc: In function ‘int main()’:
explicit.cc:21:18: error: conversion from ‘double’ to non-scalar type ‘Fraction’ requested
   21 |   Fraction d = f + 4; // which will cause ambiguous
      |                ~~^~~
```


## STL conversion example {#stl-conversion-example}

```c++
template<class Alloc>
class vector<bool, Alloc>
{
public:
  typedef __bit_reference reference;
protected:
  reference operator[](size_type n) {
    return *(begin() + difference_type(n));
  }
};

struct __bit_reference {
  unsigned int *p;
  unsigned int mask;

public:
  operator bool() const {
    return !(!(*p & mask));
  }

};
```
