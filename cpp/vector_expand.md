# c++ vector 扩容问题

问：C++ vector扩容方式

* 容量满了之后申请一块2倍大小的新内存，将之前旧地址的内容复制进去

问：复制的时间复杂度是

* vector 扩容内存拷贝使用的是uninitialized_copy，时间复杂度是O（n）

```
扩容相关代码

      const size_type old_size = size();    
      const size_type len = old_size + max(old_size, n);    
      iterator new_start = data_allocator::allocate(len);    
      iterator new_finish = new_start;    
      __STL_TRY {    
        new_finish = uninitialized_copy(start, position, new_start);    
        new_finish = uninitialized_copy(first, last, new_finish);    
        new_finish = uninitialized_copy(position, finish, new_finish);    
      }    

```

```
uninitialized_copy

template<class InputIterator, class ForwardIterator>
  ForwardIterator uninitialized_copy ( InputIterator first, InputIterator last,
                                       ForwardIterator result )
{
  for (; first!=last; ++result, ++first)
    new (static_cast<void*>(&*result))
      typename iterator_traits<ForwardIterator>::value_type(*first);
  return result;
}
```

