+++
title = "5-new-delete-basic-object-oriented-cpp"
author = ["zyy"]
date = 2025-03-30T11:28:00+08:00
tags = ["c++", "language"]
categories = ["object_oriented"]
draft = false
showtoc = true
+++

## stack and heap {#stack-and-heap}

stack memory is used for local object, local var, argument passed, return val.
heap memory is dynamically allocated using new, this memory is managed by OS.


### life span {#life-span}


#### heap object {#heap-object}

```c++
{
  complex *p = new complex(2,1);
}
```

when the program goes out of the scope, the object pointed by p is still alive but we can't access because the p is local var. In this case, it will cause memory leak , so we need to release when not use it anymore.


#### global object {#global-object}

the whole program


#### static local object {#static-local-object}

the whole program


## new {#new}

new expression memory firstly and then evoke the ctor, it will be divided three ops like that.

-   allocate memory ( using c-malloc )
-   convert type
-   ctor

<!--listend-->

```c++
complex *pc = new complex(2,1);
--->

void *mem = operator new (sizeof(complex)); // operator new function is c++ function which will use malloc.
pc = static_cast<complex*>(mem);
pc->complex::complex(2,1); // Complex::Complex(pc, 2, 1);// this
```


## delete {#delete}

call dtor first and then release memory.

-   dtor
-   free memory using free function

<!--listend-->

```c++
complex *pc = new complex(2,1);
...
delete pc;

complex::~complex(pc);// dtor
operator delete(pc); // operator delete is c++ inside function which will call free to release memory.

complex *ps = new String("hello");
...
delete ps;
     2.
ps-->|-----|           1.
     |     |---------->|-------------|
     |-----|           |hello        |
                       |-------------|

```

step1. will call String::~String() dtor, delete m_data, release allocate memory storing hello string (actually release the pointer storage).
step2. will call delete operator which free(ps) the memory storing pointer to String.


## detail of allocated memory in c++ {#detail-of-allocated-memory-in-c-plus-plus}

```nil
one element         array
|-------|     |-------|
|malloc |     |malloc |
|cookie |     |cookie |
|-------|     |-------|
|       |     | size  |
|       |     |-------|
|Complex|     |Complex|
|       |     |-------|
|       |     |Complex|
|-------|     |-------|
| pad   |     | pad   |
|-------|     |-------|
|malloc |     |malloc |
|cookie |     |cookie |
|-------|     |-------|
```

When you don't use new [] ( new array) and delete [] (delete array) in pair, it will cause memory leak which doesn in your diet.

```nil
String *p = new String [3]; |   String *p = new String [3];
delete[] p;                 |   delete p;

wake up 3 dtor,another one just wake up one time.
|------|
|      |---->|------|
|      |     |      |
|      |     |      |
|      |     |------|
|------|
|      |---->|------|
|      |     |      |
|      |     |      |
|      |     |------|
|------|
|      |
|      |---->|------|
|      |     |      |
|------|     |      |
             |------|
```

{{< figure src="/c_plus_plus/images/5_delete_array.png" >}}

And the string "world" and string "zhh" is still in memory without release in one time dtor.
