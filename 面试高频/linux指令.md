### 显示linux的某一行

1. tail和head组合使用

从第1000行开始，显示2000行。即显示1000~2999行

cat input_file | tail -n +1000 | head -n 2000

2. 

显示 1000行到3000行

cat input_file | head -n 3000 | tail -n +1001



tail -n 1000：显示最后1000行

tail -n +1000：从1000行开始显示，显示1000行以后的

head -n 1000：显示前面1000行

3.  sed -n '5,10p' input_file这样你就可以只查看文件的第5行到第10行。

4. 用awk处理

   awk 'NR==2, NR==11{print}'  input_file

   或者

   awk 'NR>2 && NR<11 {print $0}'  input_file

https://blog.csdn.net/wu8439512/article/details/78642892