首先memcpy主要考虑以下情况：

1. dest>(src+size) || dest<src   对于这种情况下dest和src不存在重叠的情况，因此可以直接进行字符串的拷贝工作。
2. 另外当在上面的两种情况之外的情况下，需要考虑重叠部分，也就是从源字符串的高地址往低地址进行字符串的拷贝工作。

```C++

void *memcpy(void *dest, const void *src, size_t count)
{
 char *d;
 const char *s;
 
 if (dest > (src+size)) || (dest < src))
    {
    d = dest;
    s = src;
    while (count--)
        *d++ = *s++;        
    }
 else /* overlap */
    {
    d = (char *)(dest + count - 1); /* offset of pointer is from 0 */
    s = (char *)(src + count -1);
    while (count --)
        *d-- = *s--;
    }
  
 return dest;
}
```

