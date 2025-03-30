+++
title = "4-syntactic-sugar-object-model-cpp"
author = ["zyy"]
date = 2025-03-30T14:26:00+08:00
tags = ["c++", "language"]
categories = ["object_model"]
draft = false
showtoc = true
+++

## auto (since c++ 11) {#auto--since-c-plus-plus-11}

Using this keyword in c++ since c11, the compiler can deduce in the compilling time. If the typename is too long to write you can use it and let compiler to deduce.

```c++
list<string> c;
list<string>::iterator ite;
ite = find(c.begin(), c.end(), target);

auto ite = find(c.begin(), c.end(), target);
```


## range-base for( since c++11) {#range-base-for--since-c-plus-plus-11}

In normal c++, we can use for-loop of type iterator or container.size to iterate the container. In c++11, it provides us a new for-loop.

```c++
for( decl : coll) {


}
```

The first must be **declaration** and the second one must be a **container**. We can use in this example.

```c++
vector<double> vec;

for(auto elem : vec ) // pass be value
{
  cout << elem << endl;
}

for(auto &elem : vec ) // pass be reference
{
  elem *= 2; // you can change
  cout << elem << endl;
}
```

When passing by value it needs to copy for every element, but passing by reference is more efficient. And you can change the actual value of elem in the vector.


## Reference {#reference}

The reference in bottom is a smart pointer than pointer, but it will pretend to show the same thing as value, which means that the size equals and the address equals.

-   sizeof(reference) ==sizeof( value)
-   &amp;reference == &amp;value


### Usage {#usage}

Usually, reference is used in argument passing not in the declaration types.
passing by value and passing by reference can be use in the same way only differs in function declaration.And the reference is more efficient. Passing by pointer is confused with the reference, and reference is pointer at some point.

```c++
void func1(Cls* pobj) { pobj->.... }
void func1(Cls& pobj) { pobj.... }
void func1(Cls pobj)  { pobj.... }

Cls obj;
func1(&obj);
func2(obj);
func3(obj);
```

signature: without return type, the function name and argument and const suffix including.
if you define two functions like that it will cause compiler ambiguity.

```c++
double imag(const double& im) {}
double imag(const double im) {}
```
