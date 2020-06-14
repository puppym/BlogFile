## python 递归遍历文件夹

1. 使用递归遍历文件夹

```python
def dirlist(path, allfile):
    filelist = os.listdir(path)

    for filename in filelist:
        filepath = os.path.join(path, filename)
        if os.path.isdir(filepath):
            dirlist(filepath, allfile)
        else:
            allfile.append(filepath)
    return allfile
```

2. 使用`os.walk`

```python
import os

def dirlist(path, allfile):
    g = os.walk(path)
    for path, d, filelist in g:
        print (d)
        for filename in filelist:
            allfile.append(os.path.join(path, filename))
    return allfile

allFileName = dirlist("/home/czm/csv_data", allfile=[])

print (allFileName)
```



