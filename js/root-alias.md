[root]
语法：root path
默认值：root html
配置段：http、server、location、if

[alias]
语法：alias path
配置段：location

root实例：

 

```
location ^~ /t/ {
     root /www/root/html/;
}
```

如果一个请求的URI是/t/a.html时，web服务器将会返回服务器上的/www/root/html/t/a.html的文件。

 

alias实例：

 

```
location ^~ /t/ {
 alias /www/root/html/new_t/;
}
```

如果一个请求的URI是/t/a.html时，web服务器将会返回服务器上的/www/root/html/new_t/a.html的文件。注意这里是new_t，因为alias会把location后面配置的路径丢弃掉，把当前匹配到的目录指向到指定的目录。

 

注意：

1. 使用alias时，目录名后面一定要加"/"。
2. alias在使用正则匹配时，必须捕捉要匹配的内容并在指定的内容处使用。
3. alias只能位于location块中。（root可以不放在location中）。