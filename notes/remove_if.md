---
title: Remove elements in a vector
tags:
  - c++
emoji:
---

In C++17
```c++
auto p = std::remove_if(
    std::begin(a), std::end(a), [](const auto& e) { return e.seleted(); });
a.erase(p, std::end(a))
```

The complexity is $$O(n)$$

In C++20, we can do
```c++
auto p = ranges::remove_if(a, &E::selected);
a.erase(p, std::end(a))
```
