### 搜索
- [使用 grep 查找所有包含指定文本的文件](https://www.linuxprobe.com/grep-find-file.html)
1. 在指定目录下,搜索包含指定字符串的文件
	- 在product目录下面搜索
		- grep -s AAA ./product/*
	- 在当前目录下面搜索
    	- grep -s AAA ./*
2. 在指定目录及其子目录下,搜索包含指定字符串的文件
	- 在product目录及其子目录下搜索
		- grep -R AAA ./product/*
	- 在当前目录及其子目录下搜索
    	- grep -R AAA ./*
3. 搜索结果仅显示文件名 l
	- grep -Rl AAA ./*
4. 大小写不敏感 i .以下两条是同样效果,大小写不敏感.
	- grep -Ri AAA ./*
    - grep -Ri aaa ./*
5. 显示结果包含要搜索的字符串在文件中的行号
	- grep -Rni AAA ./*
        
### 删除
1. 删除当前目录下所有文件,文件夹及其子文件夹,递归删除
```
rm -rf ./*
```
2. 删除指定文件夹
> 删除文件夹 d
```
rm -rf d/
```

### 查看当前所在目录
```
pwd

***:~/LinuxTest/d1$ pwd
/work/用户名/LinuxTest/d1
```

### 创建文件
1. 创建1个文件
```
touch f1.txt
```
2. 创建多个文件
```
touch 1.txt 2.txt 3.txt
```

### 创建文件夹
1. 创建1个文件夹
```
mkdir d1
```
2. 创建多个文件夹
```
mkdir d1 d2 d3
```
