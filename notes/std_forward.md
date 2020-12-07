---
title: std::forward
tags:
  - c++
  - stl
emoji:
---

In this note, I'm trying to demonstrate the usage of `std::forward` where without it, there is no easy workaround. To understand the usage, one has to know some basic concept of lvalue and rvalue C++. (Basic cases should be sufficient).

From a high level, `std::forward` helps us fowarding a type information of a variable "perfectly". Let's start with some example to explain the task.

# Perfect Forwarding

Let's start with a super simple class `Cat` with an overloaded method `meow`

```c++
class Cat {
 public:
  void meow() { std::cout << "meow()" << std::endl; }
  void meow() const { std::cout << "meow() const" << std::endl; }
};
```

We overloaded `meow` function to distinguish the cases whether the object is a constant. A simple test run:

```c++
Cat cat1;
const Cat cat2;
cat1.meow(); // meow()
cat2.meow(); // meow() const
```

For some reason, we need a (trivial) wrapper, so we have to **forward** the function call.

```c++
void callMeow(const Cat &cat) { cat.meow(); }
```

Note that, we are trying to foward, so we pass by reference to avoid copying. In that case, we **must**  `const` the argument. Otherwise, the compiler won't let us pass in a constant object as it will loss the qualifier.

When we invoke the wrapper:

```c++
callMeow(cat1); // meow() const
callMeow(cat2); // meow() const
```

which is not surprising. But it is not we want if we'd like to have **perfect forwarding**. An obvious solution is to overload the functions and implement the wrapper twice with different arguments. As a programmer, we often want to avoid code copying. Therefore, *template* comes for the resecue!

```c++
template <typename T_Cat>
void callMeow(T_Cat &cat) {
  cat.purr();
}
```

The above template function can forward the type information -- whether it is a constant perfectly:

```c++
callMeow(cat1); // meow()
callMeow(cat2); // meow() const
```
Neat! This behavior is what we call "perfect forwarding". With template, we can forward the `const` information. However, without extra trick, some type information that we cannot forward easily.

### The problem
Now let's overload another function `purr` for lvalue and rvalue objects. (It can combine with a `const` qualifier. See the next section with full code examples)
```c++
class Cat {
 public:
  void purr() & { std::cout << "purr() lvalue" << std::endl; }
  void purr() && { std::cout << "purr() rvalue" << std::endl; }
};
```

Can we still do the perect forwarding with template?

```c++
template <typename T_Cat>
void callPurr(T_Cat &cat) {
  cat.purr();
}

Cat cat;
callPurr(cat); // purr() lvalue
callPurr(Cat{}); // COMPILE ERROR
```

It turns out that when we pass in an rvalue, it doesn't even compile! Because it's passing by reference, in which case, the arguemnt cannot be an rvalue.

# Solution

 To handle lvalue and rvalue at the same time, the *universal reference* `&&` was introduced in C++11.

With `&&`, `T_Cat` will become `Cat&` when the argument is a lvalue as usual; when it's an rvalue, it becomes `Cat`, which works fine:

```c++
template <typename T_Cat>
void callPurr(T_Cat &&cat) {
  cat.purr();
}

Cat cat;
callPurr(cat); // purr() lvalue
callPurr(Cat{}); // purr() lvalue
```

It compiles now. However, it is not what we expected! After second thought with the above interpretation of `&&`, it's actually understandable. When we passed in a rvalue, it consider it as the type `Cat`, so it's a new (lvalue) copy.

Finally, along all the way, we reach to the topic of this note, `std::forward`. It helps us forward the "rvalue information". The usage is as follows.

```c++
template <typename T_Cat>
void callPurr(T_Cat &&cat) {
  std::forward<T_Cat>(cat).purr();
}

Cat cat;
callPurr(cat); // purr() lvalue
callPurr(Cat{}); // purr() rvalue
```

Perfect! And that's it. Hopefully, it will be helpful for understanding `std::forward` or even dealing with rvalue for the sake of efficiency.


# Full code examples

### source code


```c++
#include <iostream>
#include <utility>

class Cat {
 public:
  void meow() { std::cout << "meow()" << std::endl; }
  void meow() const { std::cout << "meow() const" << std::endl; }

  void purr() & { std::cout << "purr() &" << std::endl; }
  void purr() && { std::cout << "purr() &&" << std::endl; }
  void purr() const & { std::cout << "purr() const &" << std::endl; }
  void purr() const && { std::cout << "purr() const &&" << std::endl; }
};

void callMeow(const Cat &cat) { cat.meow(); }

template <typename T_Cat>
void callMeowTemplate(T_Cat &cat) {
  cat.meow();
}

template <typename T_Cat>
void callPurrTemplate(T_Cat &cat) {
  cat.purr();
}

template <typename T_Cat>
void callPurrTemplateUniversalRef(T_Cat &&cat) {
  cat.purr();
}

template <typename T_Cat>
void callPurrTemplateUniversalRefForward(T_Cat &&cat) {
  std::forward<T_Cat>(cat).purr();
}

int main() {
  Cat cat1;
  const Cat cat2;
  std::cout << "### Call Meow directly" << std::endl;
  cat1.meow();
  cat2.meow();
  std::cout << std::endl;

  std::cout << "### Call Meow" << std::endl;
  callMeow(cat1);
  callMeow(cat2);
  std::cout << std::endl;

  std::cout << "### Call Meow template" << std::endl;
  callMeowTemplate(cat1);
  callMeowTemplate(cat2);
  std::cout << std::endl;

  std::cout << "### Call Purr directly" << std::endl;
  cat1.purr();
  cat2.purr();
  Cat{}.purr();
  {
    const Cat tmpCat;
    std::move(tmpCat).purr();
  }
  std::cout << std::endl;

  std::cout << "### Call Purr Template (Ref)" << std::endl;
  callPurrTemplate(cat1);
  callPurrTemplate(cat2);
  // callPurrTemplate(Cat{}); // It expects an lvalue
  std::cout << std::endl;

  std::cout << "### Call Purr Template (Universal reference)" << std::endl;
  callPurrTemplateUniversalRef(cat1);
  callPurrTemplateUniversalRef(cat2);
  callPurrTemplateUniversalRef(Cat{});
  {
    const Cat tmpCat;
    callPurrTemplateUniversalRef(std::move(tmpCat));
  }
  std::cout << std::endl;

  std::cout << "### Call Purr Template (Universal reference and Forward)"
            << std::endl;
  callPurrTemplateUniversalRefForward(cat1);
  callPurrTemplateUniversalRefForward(cat2);
  callPurrTemplateUniversalRefForward(Cat{});
  {
    const Cat tmpCat;
    callPurrTemplateUniversalRefForward(std::move(tmpCat));
  }
  std::cout << std::endl;
  return 0;
}
```

### Result

```
### Call Meow directly
meow()
meow() const

### Call Meow
meow() const
meow() const

### Call Meow template
meow()
meow() const

### Call Purr directly
purr() &
purr() const &
purr() &&
purr() const &&

### Call Purr Template (Ref)
purr() &
purr() const &

### Call Purr Template (Universal reference)
purr() &
purr() const &
purr() &
purr() const &

### Call Purr Template (Universal reference and Forward)
purr() &
purr() const &
purr() &&
purr() const &&


```



# Reference

Mainly from this [blog post](http://kksnote.logdown.com/posts/1653018-c-forward-11-perfect-forwarding-are-what) and with various helps from stackover flow and the [c++ reference](https://en.cppreference.com/w/).