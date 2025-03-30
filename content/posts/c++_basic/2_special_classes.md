+++
title = "2-special-class-object-model-cpp"
author = ["zyy"]
date = 2025-03-30T13:55:00+08:00
tags = ["c++", "language"]
categories = ["object_model"]
draft = false
showtoc = true
+++

## pointer-like classes {#pointer-like-classes}

we say pointer like classes are smart pointers.
overload operator\* and operator-&gt; function


### simple in class {#simple-in-class}

```c++

template<class T>
class shared_ptr
{
public:
   T& operator*() const { return *px; }
   T*  operator->() const { return px; }
   shared_ptr(T* p) : px(p) {}
private:
   T* px;
   long *pn;
};


  //usage example
struct Foo
{
  ...
  void method(void) { ... }

};

shared_ptr<Foo> sp(new Foo);
Foo f(*sp);
sp->method();
px->method();

```

I am gonna to clarify that **sp-&gt;method**, the sp-&gt; will call the operator-&gt; function, it will return the px pointer but why it will call method function. Note: remember that -&gt; will never disappear when called the operator-&gt; function.


### iterator in STL {#iterator-in-stl}

we use linked-list as an example

```c++
template<class T>
struct __list_node {
  void *prev;
  void *next;
  T data;
};
template<class T, class Ref, class Ptr>
struct __list_iterator {
  typedef __list_iterator<T, Ref, Ptr> self;
  typedef Ptr pointer;
  typedef Ref reference;
  typedef __list_node<T>* link_type;
  link_type node;
  ...
  reference operator*() const { return (*node).data; }
  pointer operator->() const { return &(operator*()); }
};
```

The outline of this iterator is like this.
![](/c_plus_plus/images/2_function_like_class.png)

```c++
//example
list<Foo>::iterator ite;
*ite; //get the Foo object
ite->method();

```

Let's clarify this example, the user didn't know prev and next, the only know the Foo object, which means that the operator\* and operator-&gt; much behaves like linked list Foo object.
So  ite-&gt;method is the same as (\*ite).method, and &amp;(\*ite)-&gt;method

No matter how smarter the smart_ptr is , the operator\* and operator-&gt; didn't change except for iterator which has to resolve the real object node like Foo object.


## function-like classes {#function-like-classes}

overload operator() function


### example {#example}

Example:

```c++
template <class T1, class T2>
struct pair{
  typedef T1 first_type;
  typedef T2 second_type;
  T1 first;
  T2 second;
  pair() :first(T1()), second(T2()) {}
  pair(const T1& a, const T2& b) :first(a), second(b) {}

};
template <class T>
struct identiy .. {
  const T&
  operator() (const T& x) const { return x; }
};
template <class Pair>
struct select1st .. {
  const typename Pair::first_type&
  operator() (const Pair& x) const { return x.first; }
};

template <class Pair>
struct select2nd .. {
  const typename Pair::second_type&
  operator() (const Pair& x) const { return x.second; }
};
```

Note: The Pair is not a type in template quote line, it is just a reminder to let the caller to pass the Pair class.
Note: typename inside a struct is really a type not a random name.
They all inherit from unary_function

```c++

template <class T>
struct identiy : public unary_function<T,T> {
  const T&
  operator() (const T& x) const { return x; }
};
template <class Pair>
struct select1st : public unary_function<Pair, typename Pair::first_type>{
  const typename Pair::first_type&
  operator() (const Pair& x) const { return x.first; }
};

template <class Pair>
struct select2nd : public unary_function<Pair, typename Pair::second_type>{
  const typename Pair::second_type&
  operator() (const Pair& x) const { return x.second; }
};
```


### function-like class in STL {#function-like-class-in-stl}

```c++
template <class T>
struct plus : public binary_function<T, T, T> {
  T operator() (const T& x, const T& y) const { return x + y ; }
};

template <class T>
struct minus : public binary_function<T, T, T> {
  T operator() (const T& x, const T& y) const { return x - y ; }
};

template <class T>
struct equal_to : public binary_function<T, T, bool> {
  bool operator() (const T& x, const T& y) const { return x == y ; }
};

template <class T>
struct less : public binary_function<T, T, bool> {
  bool operator() (const T& x, const T& y) const { return x < y ; }
};
```


### base class in function like class {#base-class-in-function-like-class}

```c++
template <class Arg, class Result>
struct unary_function {
  typedef Arg argument_type;
  typedef Result result_type;
};

template <class Arg1, class Arg2, class Result>
struct binary_function {
  typedef Arg1 first_argument_type;
  typedef Arg2 second_argument_type;
  typedef Result result_type;
};
```

Actually there are only typedefs in the struct, in theory there may be size 0 in memory, but size probably equals to 1.
