---
title: svn hotcopy实现全量备份
tags: [svn]
---

参考：http://blog.csdn.net/windone0109/article/details/4040756

### 备份一个svn仓库

将svn下的project1项目仓库备份到d:/svnrootbackup/project1目录下。

```
svnadmin hotcopy d:/svnroot/project1 d:/svnrootbackup/project1
```

注：第一个参数为待备份的仓库，第二个参数为备份到的路径。

如果我们目录下有很多版本库，需要为每个版本库写这样一条语句备份，会比较麻烦。

### 使用脚本备份多个库

1）simplebackup.bat：

```
@echo 正在备份版本库%1......
@"%SVN_HOME%\bin\svnadmin" hotcopy %1 %BACKUP_DIRECTORY%/%2
@echo 版本库%1成功备份到了%2！
```

注：这里的%1，%2是为该批处理文件提供的参数。%SVN_HOME%等字段为设置的环境变量。

2）backup.bat

```
echo off
rem Subversion的安装目录
set SVN_HOME=E:\SVN\VisualSVN Server
rem 所有版本库的父目录
set SVN_REPOSITORY_ROOT=E:\SVN\Repositories
rem 备份的目录
set BACKUP_SVN_ROOT=F:\svnback
set BACKUP_DIRECTORY=%BACKUP_SVN_ROOT%\%date:~0,4%\%date:~5,2%\%date:~8,2%
if exist %BACKUP_DIRECTORY% goto checkBack
echo 建立备份目录%BACKUP_DIRECTORY%>>%SVN_ROOT%\backup.log
mkdir %BACKUP_DIRECTORY%
rem 验证目录是否为版本库，如果是则取出名称备份
for /r %SVN_REPOSITORY_ROOT% %%I in (.) do @if exist "%%I/conf/svnserve.conf" "%SVN_REPOSITORY_ROOT%/simplebackup.bat" "%%~fI" %%~nI
goto end
:checkBack
echo 备份目录%BACKUP_DIRECTORY%已经存在，请清空。
goto end
:end
```

注：如果是项目的版本库，则目录下一定会存在conf/svnserve.com文件。

注2：命令都是有双引号括起来了，这是为了防止命令路径中有空格。

分析for循环语法：

```
for /r %SVN_REPOSITORY_ROOT% %%I in (.) do @if exist "%%I/conf/svnserve.conf" "%SVN_REPOSITORY_ROOT%/simplebackup.bat" "%%~fI" %%~nI
```

/r指定遍历的目录，并使用%%I来引用变量的值。

@if判断目录下的文件是否存在，如果存在则执行simplebackup.bat文件。

%%~fI获取文件/目录的全路径（f理解为full）

%%~nI获取文件的文件名（n理解为name）

字符串的截取：

```
%date:~0,4%\%date:~5,2%\%date:~8,2%
```

截取date命令输出内容的前4个字符，运行echo命令查看输出结果。

```
echo %date:~0,4%
```

注：第一个数值为从那个下标开始，第二个数值为截取的长度。

3）实际运行命令

配置目录SVN_HOME，SVN_REPOSITORY_ROOT和BACKUP_SVN_ROOT，配置完成后运行backup.bat命令。