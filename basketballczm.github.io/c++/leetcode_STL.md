## 最近在刷leetcode发现一些常用的标准STL老是弄混淆，特此记录 www.cpluslpus.com

### stack
empty()
size()
top()
push()
emplace() c++11 emplace(参数) 将参数传递给该元素的构造函数，进行生成一个新的对象
pop()
swap() C++

### vector
Iterators
begin()
end()
rbegin() reverse
rend()
cbegin() const
cend()

Capacity
size()
empty()
reserve()
resize()
max_size() return maximum size

Element access
operator[]
front()
back()

Modifiers
erase()
push_back()
pop_back()
swap()
emplace() C++11
emplace_back()

### queue
empty()
size()
front()
back()
push()
emplace() c++ 11
pop()
swap() c++ 11 foo.swap(bar) 对两个队列对象进行交换

### deque 底层是双向队列
empty()
size()
front()
back()
push_back()
pop_front()