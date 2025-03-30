+++
title = "6-new-delete-object-model-cpp"
author = ["zyy"]
date = 2025-03-30T14:28:00+08:00
tags = ["c++", "language"]
categories = ["object_model"]
draft = false
showtoc = true
+++

## operator new/delete {#operator-new-delete}


### introduction {#introduction}

In the last sections, we've already talked about new and delete. new expression is divided three parts

-   allocate memory using operator new
-   static_cast
-   ctor

and the delete expression is divided two parts

-   dtor
-   operator delete to release memory

However, expression is not allowed to override, but operator new and operator delete are allowed to override inside class or outside class(global).


### override ::operator new/::delete/::new[]/::delete[] {#override-operator-new-delete-new-delete}

here is an example to override the global operator new/delete function

```c++
void *myalloc(size_t size) { return malloc(size); }
void myfree(void *ptr) { return free(ptr); }

// they cannot be declared in a namespace
void* operator new(size_t size){
cout << "global new()\n"; return myalloc(size);
}

void* operator new[](size_t size){
cout << "global new[]()\n"; return myalloc(size);
}

void operator delete(void *ptr){
cout << "global delete() \n"; return free(ptr);
}

void operator delete[](void *ptr){
cout << "global delete[]() \n"; return free(ptr);
}
```

There is one thing to note that, replacement function can not be declared inline.
Actually the parameter is passed by the compiler, you don't have to calculate the size.


### override member operator new/delete {#override-member-operator-new-delete}

```c++
class Foo{
  public:
    void *operator new[](size_t);
    void operator delete[](void*, size_t); // the second arg is optional
};

Foo *p = new Foo[N];
// can be converted to
try {
  void *mem = operator new(sizeof(Foo) * N + 4);
  p = static_cast<Foo*>(mem);
  p->Foo:Foo(); // N times
}
delete[] p;
// can be converted to
p->~Foo(); // N times
operator delete(p);
```

The memory operator new/delete override example is shown below

```c++
class Foo
{
public:
  int _id;
  long _data;
  string _str;

public:
Foo() :_id(0) { cout <<"default ctor this " << this << "id=" << _id << endl; }
Foo(int i) : _id(i) { cout << "ctor this " << this << "id=" << _id << endl;}
~Foo() { cout << "dtor this " << this << "id=" << _id << endl; }
static void *operator new(size_t size);
static void *operator new[](size_t size);
static void operator delete(void *pdead, size_t size);
static void operator delete[](void *pdead, size_t size);
};

void* Foo::operator new(size_t size){
Foo* p = (Foo*)malloc(size);
return p;
}

void* Foo::operator new[](size_t size){
Foo* p = (Foo*)malloc(size);
cout << "size new[] " << size << endl;
return p;
}

void Foo::operator delete(void *pdead, size_t size){
free(pdead);
}

void Foo::operator delete[](void *pdead, size_t size){
free(pdead);
}

int main(){
cout << sizeof(Foo) << endl;
Foo *p = new Foo(7);
delete p;

Foo *parray = new Foo[5];
delete []parray;
}
```

After I run this program it will print out something like this below.

```shell
48
ctor this 0x5586af6376c0id=7
dtor this 0x5586af6376c0id=7
size new[] 248
default ctor this 0x5586af637708id=0
default ctor this 0x5586af637738id=0
default ctor this 0x5586af637768id=0
default ctor this 0x5586af637798id=0
default ctor this 0x5586af6377c8id=0
dtor this 0x5586af6377c8id=0
dtor this 0x5586af637798id=0
dtor this 0x5586af637768id=0
dtor this 0x5586af637738id=0
dtor this 0x5586af637708id=0
```

The sizefo Foo is 48 bytes, but when I allocate 5 object with new[] it turns out that it will allocate more 8 bytes that I think, That is the compiler use this 8 bytes to store the size of the array which can ctor quickly in 64bit machine.


### new/delete pair {#new-delete-pair}

In such a condition, if we use new[] to allocate memory, but delete without [], what will happened?
Here is the example to illustrate.

```c++
#include <iostream>
#include <cstdlib>

using namespace std;
void *myalloc(size_t size) { return malloc(size); }
void myfree(void *ptr) { return free(ptr); }

// they cannot be declared in a namespace
void* operator new(size_t size){
  cout << "global new() " << size << endl; return myalloc(size);
}

void* operator new[](size_t size){
cout << "global new[]()\n"; return myalloc(size);
}

void operator delete(void *ptr){
cout << "global delete() \n"; myfree(ptr);
}

void operator delete[](void *ptr){
cout << "global delete[]() \n"; myfree(ptr);
}

class A{
public:
  int data;
public:
  ~A() { cout << "dtor A " <<endl; }
};

int main(){
  A *pa = new A[4];
  pa->data = 4;
  delete pa;
}
```

The result is that it will core dumped. But what happened in detail.

```shell
$./a.out
global new[]()
dtor A
global delete()
segmentation fault (core dumped)

using valgrind to check


==692147== 24 bytes in 1 blocks are definitely lost in loss record 1 of 1
==692147==    at 0x484A2F3: operator new[](unsigned long) (in /usr/libexec/valgrind/vgpreload_memcheck-amd64-linux.so)
==692147==    by 0x1093B1: main (new.cc:34)
==692147==
==692147== LEAK SUMMARY:
==692147==    definitely lost: 24 bytes in 1 blocks
==692147==    indirectly lost: 0 bytes in 0 blocks
==692147==      possibly lost: 0 bytes in 0 blocks
==692147==    still reachable: 0 bytes in 0 blocks
==692147==         suppressed: 0 bytes in 0 blocks
```

We should use new/delete in pair.


### placement new/delete {#placement-new-delete}

We can override class member operator new() with lots of versions, the beforhand of this overridding is that every declaration must have special argument list, and the first argument is size_t, when happens new(), and the argument in the () is placement arguments.
**Foo \*pf = new(300, 'c') Foo;**
Also, we can override class member operator delete() with lots of versions, and it will not be called by delete until throw an exception in ctor. It is mainly used in measure memory in object.

```c++
class Bad{};
class Foo{
public:
  Foo() { cout << "Foo::Foo()" << endl; }
  Foo(int) { cout << "Foo::Foo(int)" <<endl; throw Bad(); }

  void *operator new(size_t size){
      return malloc(size); // ordinary malloc
  }

  void *operator new(size_t size, void *start){
      return start;
  }

  void *operator new(size_t size, long extra){
      return malloc(size + extra);
  }

  void *operator new(size_t size, long extra, char init){
      return malloc(size + extra);
  }

  // ordinary delete
  void operator delete(void *pdead, size_t size){
      cout << "operator delete(void , size_t) " << endl;
  }

  void operator delete(void *pdead, void *){
      cout << "operator delete(void , void) " << endl;
  }

  void operator delete(void *pdead, long){
      cout << "operator delete(void , long) " << endl;
  }

  void operator delete(void *pdead, long, char){
      cout << "operator delete(void , long, char)" << endl;
  }
private:
  int m_;
};
```

If you give up dealing with ctor exception, there is no need to correspond new/delete one by one.
But in my version of g++(13.0) there is no placement operator called.

```shell
Foo::Foo()
Foo::Foo(int)
terminate called after throwing an instance of 'Bad'
aborted (core dumped)
```


### basic_string {#basic-string}

Basic_string use member placement operator new to store reference counting.

```c++
template <>
class basic_string{
private:
  struct Rep{
    void release() { if( -- ref== 0) delete this; }
    static void *operator new(size_t, size_t);
    static void operator delete(void *);
    static Rep* create(size_t);
  };
};

template<class charT, class traits, class Allocator>
inline basic_string<charT, traits, Allocator>::Rep*
basic_string <charT, traits, Allocator>::Rep::
create(size_t extra)
{
  extra = frob_size(extra + 1);
  Rep * p = new(extra) Rep;
  return p;
}

...
operator new(size_t s, size_t extra){

  return Allocator::allocate(s + extra * sizeof(charT));
}
```
