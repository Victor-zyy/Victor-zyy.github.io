+++
title = "8-virtual-polymorphism-basic-object-oriented-cpp"
author = ["zyy"]
date = 2025-03-30T11:58:00+08:00
tags = ["c++", "language"]
categories = ["object_oriented"]
draft = false
showtoc = true
+++

## inheritance with virtual function {#inheritance-with-virtual-function}


### virtual function {#virtual-function}

The inheritance with virtual function can make the most of it.
The member function has three forms classified by **virtual**. And the virtual function needs to be **overriden** by derived class.

1.  non-virtual function (with no virtual identifier)
2.  impure virtual function (with virtual identifier and default definition)
3.  pure virtual (with virtual identifier, = 0)  // fixme

Sometimes we need a default virtual function definition so impure is necessary.

```c++
class Shape
{
public:
  virtual void draw() const = 0; // pure virtual function
  virtual void error(const std::string & msg); // impure virtual function
  int objecId() const; // non-virtual function
};


class Rectangle: public Shape{

};

class Ellipse: public Shape{

};
```


### Template Method {#template-method}

Template Method is a design pattern in oop and ood.Example, MFC, application framework.
It means delay the action until the derived class to override it.

```c++
//framework
CDocument::
OnFileOpen()
{
...
Serialize();
...
};

// application
class CMyDoc: public CDocument
{
  //override
  virtual Serialize() {...}
};

// main
main()
{
  CMyDoc myDoc;

  myDoc.OnFileopen();
}

CDocument::OnFileOpen(&myDoc);
->
this->Serialize() in OnFileOpen
```


### inheritance plus composition {#inheritance-plus-composition}

The two forms of this type are listed below.
Form one:
![](/c_plus_plus/images/8_inheritance_composition_form1.png)
I will write a piece of code to figure it out that base ctor calls first or component ctor calls first.

```c++

#include <iostream>
using namespace std;

class Base{
public:
  Base() { cout << "Base() ctor" << endl; }
  ~Base() { cout << "~Base() dtor" << endl; }
};

class Component
{
public:
  Component() { cout << "Component() ctor" << endl; }
  ~Component() { cout << "~Component() dtor" << endl; }
};

class Derived: public Base{
public:
  Derived() { cout << "Derived() ctor" << endl; }
  ~Derived() { cout << "~Derived() dtor" << endl; }
private:
  Component b;
};

int main()
{
  Derived b;
  return 0;
}
```

```sh
output:
Base() ctor
Component() ctor
Derived() ctor
~Derived() dtor
~Component() dtor
~Base() dtor
```

Form two:
![](/c_plus_plus/images/8_inheritance_composition_form2.png)
The latter ctor and dtor order is clear.


### inheritance plus delegation {#inheritance-plus-delegation}

in ppt, or word, we have the same data value but different view of this data. This is called design pattern
Obsever.

```c++
class Subject
{
  int m_value;
  vector<Obsever*> m_views;
public:
  void attach(Obsever *obs){
    m_views.push_back(obs);
  }
  void set_val(int val){
    m_value = val;
    notify;
  }
  void notify(){
    for(int i = 0; i < m_views.size(); i++)
      m_views[i]->update(this, m_value);
  }
};

class Obsever
{
public:
  virtual void update(Subject *sub, int value) = 0;
};
```

The outline of this is like,
![](/c_plus_plus/images/8_inheritance_delegation.png)
Some class inheriting from Observer can be inside the object vector, each one use update virtual function can do different things-**polymorphism**.


## design pattern {#design-pattern}

inheritance plus delegation
Example-Prototype

```c++

#include <iostream>
using namespace std;
enum imageType{
 LSAT,SPOT
};
class Image
{
public:
  virtual void draw() = 0;
  static Image* findAndClone(imageType);
protected:
  virtual imageType returnType() = 0;
  virtual Image* clone() = 0;
  //As each subclass of Image is declared, it registers its prototype
  static void addPrototype(Image *image){
    _prototypes[_nextSlot++] = image;
  }
private:
  // addPrototype() saves each registered prototype here
  static Image* _prototypes[10];
  static int _nextSlot;
};

//definition when you declare static inside class
Image *Image::_prototypes[];
int Image::_nextSlot;

//Client Calls this public static member function when it needs an instance of
//an Image subclass
Image* Image::findAndClone(imageType type){
  for(int i = 0; i < _nextSlot; i++){
    if(_prototypes[i]->returnType() == type)
      return _prototypes[i]->clone();
  }
  return NULL;
}


class LandSatImage : public Image
{
public:
  imageType returnType(){
    return LSAT;
  }
  void draw() {
    cout << "LandSatImage::draw" << _id << endl;
  }
  // when clone() is called, call the one-argument ctor with a dummy argument
  Image *clone() {
    return new LandSatImage(1);
  }
protected:
  //This is only called from clone
  LandSatImage(int dummy){
    _id = _count ++;
  }

private:
  // Mechanism for initializing an Image subclass
  // this causes the default ctor to be called, which registered the subclass's prototype

  static LandSatImage _landSatImage;
  // This only called when the private static data member is initialized
  LandSatImage(){
    addPrototype(this);
  }
  // Normal State per instance mechanism
  int _id;
  static int _count;
};
// Register the subclass's prototype
LandSatImage LandSatImage::_landSatImage;
// Initialize the "state" per instance mechanism
int LandSatImage::_count = 1;



class SpotImage:public Image
{
public:
  imageType returnType(){
    return SPOT;
  }

  void draw() {
    cout << "SpotImage::draw" << _id << endl;
  }

  Image *clone() {
    return new SpotImage(1);
  }
protected:
  SpotImage(int dummy){
    _id = _count++;
  }

private:
  SpotImage() {
    addPrototype(this);
  }
  static SpotImage _spotImage;
  int _id;
  static int _count;
};
SpotImage SpotImage::_spotImage;
int SpotImage::_count = 1;

// Simulated stream of creation requests
const int NUM_IMAGES = 8;
imageType input[NUM_IMAGES] =
{
LSAT, LSAT, LSAT, SPOT, LSAT, SPOT, SPOT, LSAT
};

int main()
{
    Image *images[NUM_IMAGES];
    // Given an image type, find the right prototype, and return a clone
    for (int i = 0; i < NUM_IMAGES; i++)
      images[i] = Image::findAndClone(input[i]);
    // Demonstrate that correct image objects have been cloned
    for (int i = 0; i < NUM_IMAGES; i++)
      images[i]->draw();
    // Free the dynamic memory
    for (int i = 0; i < NUM_IMAGES; i++)
      delete images[i];
}

```


#### Composite {#composite}

design pattern for DB(small file system)
![](/c_plus_plus/images/8_composite.png)

```c++
class Component
{
  int value; //default private
public:
  Component(int val) : value(val) {}
  virtual void add(Component *){} // should not pure-virtual for Primitive is not a dir

};

class Primitive:public Component
{
public:
  Primitive(int val): Component(val){}
};

class Composite : public Component
{
  vector<Component*> v;
public:
  Composite(int val) : Component(val) {}
  void add(Component *com) {
    v.push_back(com);
  }
};
```
