---
title: cmu15445-project0
date: 2024-02-22 02:15:27
tags: cmu15445—2023
categories: cmu15445-2023
---

## TASK 1 Copy-On-Write Trie
COW Trie在每次插入和删除时不会改变原有节点，而是对该节点的副本进行修改后，依次为其父节点创建修改后的副本，最后返回一个新的根节点。
***
插入("ad", 2),创建了一个新的Node2
![img](../images/cmu15445-project0/2.png)

***

插入("b", 3)
![img](../images/cmu15445-project0/1.png)

***

插入("a", "abc") 删除("ab", 1)<br>
注意删除操作后需要清除所有不需要的节点

![img](../images/cmu15445-project0/3.png)


Get函数实现
> 从root节点遍历Tire树，
> 如果key不存在返回nullptr，
> 如果key存在，但是对应的Node无value或者value的类型不匹配，返回nullptr
> 其它情况，返回value
```cpp
// Get the value associated with the given key.
// 1. If the key is not in the trie, return nullptr.
// 2. If the key is in the trie but the type is mismatched, return nullptr.
// 3. Otherwise, return the value.
template <class T>
auto Trie::Get(std::string_view key) const -> const T * {
  if (!root_) {
    return nullptr;
  }
  std::shared_ptr<const TrieNode> ptr(root_);
  for (char ch : key) {
    if (ptr->children_.count(ch) == 0) {
      return nullptr;
    }
    ptr = ptr->children_.at(ch);
  }
  if (!ptr->is_value_node_) {
    return nullptr;
  }
  auto p = std::dynamic_pointer_cast<const TrieNodeWithValue<T>>(ptr);
  if (!p) {
    return nullptr;
  }
  return p->value_.get();
}
```

``` cpp
template <class T>
auto Trie::Put(std::string_view key, T value) const -> Trie {
  // Note that `T` might be a non-copyable type. Always use `std::move` when creating `shared_ptr` on that value.

  // You should walk through the trie and create new nodes if necessary. If the node corresponding to the key already
  // exists, you should create a new `TrieNodeWithValue`.
  std::shared_ptr<const TrieNode> new_root(nullptr);
  std::map<char, std::shared_ptr<const TrieNode>> children;
  if (key.length() == 0) {//key长度为0，表示在root节点put value
    if (root_) {
      children = root_->children_;
    }
    new_root = std::make_shared<const TrieNodeWithValue<T>>(children, std::make_shared<T>(std::move(value)));//创建一个新的root节点
    return Trie(new_root);
  }

  std::vector<std::unique_ptr<TrieNode>> stack;
  if (root_) {
    stack.push_back(root_->Clone());
  } else {
    stack.push_back(std::make_unique<TrieNode>());
  }
  auto ptr(root_);

  for (int64_t i = 0; i < static_cast<int64_t>(key.length() - 1); ++i) {
    std::unique_ptr<TrieNode> tmp_ptr(nullptr);
    if (ptr && ptr->children_.count(key[i]) == 1) {
      ptr = ptr->children_.at(key[i]);
      tmp_ptr = ptr->Clone();
    } else {
      tmp_ptr = std::make_unique<TrieNode>();
      ptr = nullptr;
    }

    stack.push_back(std::move(tmp_ptr));
  }
  auto value_ptr = std::make_shared<T>(std::move(value));
  if (ptr && ptr->children_.count(key.back())) {
    ptr = ptr->children_.at(key.back());
    children = ptr->children_;
  }
  auto value_node = std::make_unique<TrieNodeWithValue<T>>(children, std::move(value_ptr));
  stack.push_back(std::move(value_node));

  for (int64_t i = key.length() - 1; i >= 0; i--) {
    auto tmp_ptr = std::move(stack.back());
    stack.pop_back();
    stack.back()->children_[key[i]] = std::move(tmp_ptr);
  }
  new_root = std::move(stack.back());
  return Trie(new_root);
}
```


## TASK 2 Concurrent Key-Value Store




## TASK 3 Debugging




## TASK 4 SQL String Functions



