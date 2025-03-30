+++
title = "4-big-three-object-oriented-cpp"
author = ["zyy"]
date = 2025-03-30T11:15:00+08:00
tags = ["c++", "language"]
categories = ["object_oriented"]
draft = false
showtoc = true
+++

## Big three {#big-three}

why we need big there, and what conditions do we need big three?
The class contains pointer must include these three functions.
Note: if we don't write copy ctor ,copy op= and dtor, the compiler will generate automatically.
For copy ctor it will copy bit by bit directly.
For dtor it will do nothing.

-   copy ctor
-   copy op= (copy assignment function)
-   dtor (destructor function)

String Class with pointer.


### String Class {#string-class}

```c++
class String
{

};

String::function ...

Global-function

int main(){
  String s1();
  String s2("hello");

  String s3(s1); // copy ctor
  cout << s3 << endl;
  s3 = s2;   // copy op=

  cout << s3 << endl;
}
```


### String Class Outline {#string-class-outline}

```c++
class String
{
public:
  String (const char* cstr = 0);
  String (const String& str);
  String& operator=(const String& str);

  ~String();
  char *get_c_str() const { return m_data; }

private:
  char *m_data;
}
```


### ctor {#ctor}

The string has two formats like, one is lens + string, another one is string end with **'\\0'**. C and C++ choose the latter one.

```c++
inline
String::String(const char* cstr = 0)
{
  if(cstr){
    m_data = new char[ strlen(cstr) + 1 ];
    strcpy(m_data, cstr);
  }else{
    m_data = new char[1];
    *m_data = '\0';
  }

}

inline
String::~String()
{
  delete [] m_data;
}
```


### copy op= {#copy-op}

If we don't provide a deep copy function of copy assignment, the compiler will generate a shadow copy version, which will causes memory leak in the future.

I will show you a img to illustrate this danger and the difference of shadow copy and deep copy.
![](/c_plus_plus/images/4_copy_op=.png)

And the alias in c++ is dangerous.

```c++
inline String&
String::operator =(const String& str)
{
// check self assignment
  if(this == &str)
    return *this;

  delete[] m_data;
  m_data = new char [strlen(str.m_data) +  1];
  strcpy(m_data, str.m_data);
  return *this;
}

//
String s1("hello");
String s2(s1);
s2 = s1;
```

Don't forget self assignment check, it will cause undefined error when you don't.

{{< figure src="/c_plus_plus/images/4_copy_op=_self_check.png" >}}

After deletion, the strcpy will access the memory which just deleted, will cause UB( undefined behavior).


### copy ctor {#copy-ctor}

Note: brothers between each are friends, so it can access private data like str.m_data.

```c++
inline
String::String (const String & str)
{
  m_data = new char[ strlen(str.m_data) + 1 ];
  strcpy(m_data, str.m_data);
}

{
  String s1("hello");
  String s2(s1);
//String s2 = s1; has the same meaning of String s2(s1);

}
```


### output {#output}

```c++
#include <iostream>
ostream& operator<<(ostream& os, const String& s)
{
  return os << s.get_c_str();
}

String s1("hello");
cout << s1;
```
