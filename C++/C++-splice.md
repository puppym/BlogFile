函数有以下三种声明：

一：void splice ( iterator position, list<T,Allocator>& x );

二：void splice ( iterator position, list<T,Allocator>& x, iterator it );

三：void splice ( iterator position, list<T,Allocator>& x, iterator first, iterator last );

解释：

position 是要操作的list对象的迭代器

list&x 被剪的对象

对于一：会在position后把list&x所有的元素到剪接到要操作的list对象
对于二：只会把it的值剪接到要操作的list对象中
对于三：把first 到 last 剪接到要操作的list对象中



* https://blog.csdn.net/qjh5606/article/details/85881680