---
title: Day02 Linux 目录与文件常用命令(必背)
date: 2026-04-22 12:51:55
tags: [Linux]
categories: [自学记录]
---
# Day02｜Linux 目录与文件常用命令（必背版）

今天是 180 天运维自学计划的第二天，我们正式开始 Linux 最核心的内容：**文件与目录操作**。
这部分是 Linux 的基础中的基础，必须熟练到肌肉记忆。
<!-- more -->
## 一、Linux目录结构
Linux 一切皆文件，目录结构如下：
- `/`: 根目录,所有文件和目录的根（起点），也叫根目录。
- `/root`: 根目录的子目录及root用户目录
- `/home`: 普通用户目录
- `/etc`: 系统配置文件
- `/bin`: 系统命令目录
- `/dev`: 设备文件目录
- `/var`: 运行时数据目录及日志文件、数据文件
- `/tmp`: 临时文件目录
- `/usr`: 用户软件安装目录
- `/lib`: 系统库文件目录
- 等等

## 二、Linux 目录操作

### 1.切换目录 `cd` （change directory）
```bash
cd /root # 切换到/root目录
cd ~     # 切换到当前用户目录
cd -     # 退回上一次所在目录
cd ..    # 退回上一级目录
```

### 2.查看目录 `ls` `pwd`（list directory & present working directory）
```bash
pwd     # 查看当前所在目录
ls      # 查看当前目录下所有文件及目录
ls -l   # 查看当前目录下所有文件及目录的详细信息(简写ll)
ls -a   # 查看当前目录下所有文件及目录包括隐藏文件(简写aa)
> -a = all 
ls -lh  # 查看当前目录下所有文件及目录的详细信息，并显示文件大小
> -h = human readable  作用：把文件大小变成 GB/MB/KB，方便人看
```

### 3.创建目录 `mkdir` （make directory）
```bash
mkdir dir1                  # 创建目录dir1
mkdir dir1 dir2 dir3        # 创建多个目录
mkdir -p dir1/dir2/dir3     # 创建多级目录
> -p = parents（父目录）
> 作用：自动创建路径中缺失的父目录  自上往下创建缺失的目录
> 发现dir1不存在则创建，再看dir1下的dir2不存在则创建dir2，再看dir2下的dir3不存在则创建dir3
mkdir -m 755 dir1           # 创建目录并设置权限
> 755：所有者可读写执行，其他人只能读和执行
> 第一位 7：文件 / 目录所有者的权限 → 读 (4)+ 写 (2)+ 执行 (1) = 7
> 第二位 5：文件 / 目录所属用户组的权限 → 读 (4)+ 执行 (1) = 5
> 第三位 5：文件 / 目录其他用户的权限 → 读 (4)+ 执行 (1) = 5
```

### 4.删除目录 `rmdir` （remove directory）
```bash
rmdir dir1                  # 删除目录dir1
rmdir dir1 dir2 dir3        # 删除多个目录
rmdir -v dir1               # 删除目录并显示删除信息
> -v = verbose 详细输出 / 显示执行过程
rmdir -p dir1/dir2/dir3     # 删除多级目录
rmdir -p -v dir1/dir2/dir3  # 删除多级目录并显示删除信息
> rmdir -p → 不是递归，是 “反向清理空父目录”
> 作用：自动删除变空的父目录  自下而上删除空的父目录 (前提：所有目录必须是空的) 
> 先删除dir3，发现dir2空了再删除dir2，再发现dir1空了再删除dir1
```

### 5.修改目录权限 `chmod` （change mode）
```bash
chmod 755 dir1             # 修改目录dir1的权限
> 755：所有者可读写执行，其他人只能读和执行
chmod -R 755 dir1          # 递归修改目录及子目录权限
> -R = recursive 递归
chmod u+x dir1             # 为当前用户添加执行权限
chmod g+x dir1             # 为所属用户组添加执行权限
chmod o+x dir1             # 为其他用户添加执行权限
chmod a+x dir1             # 为所有用户添加执行权限
chmod ugo+x dir1           # 为所有用户添加执行权限
chmod u=rwx,g=rx,o=rx dir1 # 为当前用户、所属用户组、其他用户分别添加权限
> u：所有者  g：所属用户组  o：其他用户  a：所有用户
> x：执行  w：写入  r：读  -：取消权限  +：添加权限  =：设置权限     
```

### 6.修改目录所有者 `chown` （change owner）
```bash
chown user:group dir1       # 修改目录dir1的所有者和用户组
chown user dir1             # 改变目录dir1的所有者
chown :group dir1           # 改变目录dir1的用户组
chown -R user:group dir1    # 递归修改目录及子目录的所有者和用户组
chown -R user:group *.txt   # 批量修改文件所有者及子目录下的文件所有者
```

### 7.查看目录权限 `stat` （statistics 统计）
```bash
stat dir1                         # 查看目录dir1的权限
stat -c "%a %n" *                 # 批量查看文件权限
> %a：以数字形式显示权限(如 644、755)  %A:看符号权限(如rw r w)
> %s：文件大小  %n：文件名
```

## 三、Linux 文件操作

### 1.创建文件 `touch` （touch 接触触摸）
```bash
touch file1                                 # 创建文件file1
touch file1 file2 file3                     # 创建多个文件(.txt .md 等)
touch file{1..10}.txt                       # 创建带序号的10个文件(file1.txt ~ file10.txt)
touch -t 202505201314 test.txt              # 创建文件并指定时间格式(格式：YYYYMMDDhhmm)
touch -d "2025-05-20 13:14:15" test.txt     # 创建文件并指定自然语言时间(格式：YYYY-MM-DD hh:mm:ss)
touch -a test.txt                           # 修改访问时间
touch -m test.txt                           # 修改修改时间
touch -c test.txt                           # 创建文件并忽略已存在的文件
touch -r file1 file2                        # 创建文件并复制文件属性
> -a：修改访问时间  -m：修改修改时间  -c：创建文件并忽略已存在的文件  -r：创建文件并复制文件属性
```

### 2.查看文件属性 `file` （file）
```bash
file file1              # 查看文件属性
```

### 3.查看文件内容 `cat` （concatenate 连接）
```bash
cat file1                   # 查看输出文件内容
cat -n file1                # 查看输出文件内容并显示行号
cat -s file1                # 删除多余空行
```

### 4.复制文件 `cp` （copy 复制）
```bash
cp file1 file2              # 复制文件file1为file2
cp -r dir1 dir2             # 递归复制目录dir1为dir
cp file1 dir1               # 复制文件file1到目录dir1
cp -r dir1 /root            # 递归复制目录dir1到/root目录
> -i：覆盖前提示确认  -r：递归  -p：复制属性  -f：强制覆盖  -a：复制所有属完整保留原文件
```

### 5.移动/重命名文件 `mv` （move 移动）
```bash
mv name1 name2               # 重命名文件name1为name2
mv dir1 dir2                 # 重命名目录dir1为dir2
mv file1 dir1                # 移动文件file1到目录dir1
mv -i file1 dir1             # 移动文件file1到目录dir1并覆盖前提示确认
mv -f file1 dir1             # 移动文件file1到目录dir1并强制覆盖
mv -n file1 dir1             # 移动文件file1到目录dir1并忽略已存在的文件
mv -s file1 dir1             # 移动文件file1到目录dir1并删除源文件
```

### 6.删除文件 `rm` （remove 移除）
```bash
rm file1                    # 删除文件file1
rm -f file1                 # 强制删除文件file1
rm -i file1                 # 删除前提示确认
rm -r dir1                  # 递归删除目录dir1
rm -rf dir1                 # 强制删除目录dir1
rm -d dir1                  # 删除空目录dir1
> -i：删除前提示确认  -r：递归  -f：强制删除  -d：删除空目录
```

### 7.查找文件 `find` （find 寻找）
```bash
# name    匹配文件名
find /home -name "file1"            # 在 /home 下找 file1
find / -name "*.log"                # 全盘找所有 .log 结尾的文件
find . -name "test*"                # 当前目录找 test 开头的文件
# type    匹配文件类型
find / -type f -name "*.log"        # 全盘找所有 .log 结尾的文件
find / -type d -name "test*"        # 全盘找 test 开头的目录
# size    匹配文件大小
find / -size +100M -name "*.log"    # 全盘找 大于 100M 的 .log 结尾的文件
find / -size -100M -name "*.log"    # 全盘找 小于 100M 的 .log 结尾的文件
find / -size 100M -name "*.log"     # 全盘找 100M 的 .log 结尾的文件
# time    匹配文件时间
find / -mtime +1 -name "*.log"      # 全盘找 1 天前内容修改过的 .log 结尾的文件
find / -mtime -1 -name "*.log"      # 全盘找 1 天内内容修改过的 .log 结尾的文件
find / -atime +1 -name "*.log"      # 全盘找 1 天前访问过的 .log 结尾的文件
find / -atime -1 -name "*.log"      # 全盘找 1 天内访问过的 .log 结尾的文件
find / -ctime +1 -name "*.log"      # 全盘找 1 天前状态改过的 .log 结尾的文件
> 文件属性 / 权限 / 所有者 / 文件名被修改的时间(chmod、chown、mv 都会变 ctime)
find / -ctime -1 -name "*.log"      # 全盘找 1 天内状态改过的 .log 结尾的文件
```

### 8.查找文件里内容 `grep` （grep 搜索）
```bash
grep "test" file1                   # 在file1文件中查找test
grep -i "test" file1                # 在file1文件中忽略大小写查找test
grep -r "test" dir1                 # 在dir1目录下递归查找test
grep -v "test" file1                # 在file1文件中反向查找test(排除)
grep -n "test" file1                # 在file1文件中查找test并显示行号
grep -c "test" file1                # 在file1文件中查找test并统计个数
grep -w "test" file1                # 在file1文件中精确查找test(全词匹配)
grep -l "test" dir1                 # 在dir1目录下查找test并显示文件名
grep -E "test|test1" file1          # 在file1文件中使用正则表达式搜索test或者test1
grep "test" file1 | grep -v "#"     # 搜索包含关键字但不包含注释的行
```

### 9.查看文件内容`less` （less 浏览）`tail` `head` （head 显示文件开头，tail 显示文件结尾）
```bash
less file1                          # 浏览文件file1
less -N file1                       # 浏览文件file1并显示行号
> less查看文件可以 1.上下箭头：逐行滚动 2.PageUp / PageDown：翻页 3.G：跳到文件末尾 4.g：跳到文件开头 5./ 关键词：向下搜索搜到后按 n 下一个，N 上一个 ? 关键词：向上搜索 6.q：退出 less
tail -f file1                       # 持续(实时)查看文件file1内容
head -n 10 file1                    # 显示文件file1的前10行
tail -n 10 file1                    # 显示文件file1的最后10行
head -n 20 file1 | tail -n 10       # 显示文件file1前20行中的后10行(11-20行)
```

### 10.压缩/解压文件 `zip` `tar`    （zip 压缩，tar  tape archive 磁带归档）
```bash
zip file1.zip file2 file3                   # 压缩file2和file3为file1.zip
zip -r dir1.zip dir1                        # 递归压缩dir1为dir1.zip
unzip file1.zip                             # 解压file1.zip
unzip file1.zip -d dir1                     # 指定解压file1.zip到dir1
tar -zcvf file1.tar.gz/.tgz file2 file3     # 压缩file2和file3为file1.tar.gz
tar -zcvf file1.tar.gz/.tgz dir1            # 递归压缩dir1为file1.tar.gz
> -z：使用gzip压缩  -c：create 创建包  -v：显示压缩进度  -f：指定压缩文件名  -r：递归压缩 -u：更新已存在的文件 -o：覆盖已存在的文件  -k：保留已存在的文件
tar -zxvf file1.tar.gz                      # 解压file1.tar.gz
tar -zxvf file1.tar.gz -C dir1              # 指定解压file1.tar.gz到dir1
> -z：使用gzip解压  -x：extract 解压  -v：显示解压进度  -f：指定解压文件名  -C：指定解压目录
```

## 四、Day02 合格标准
1. 能熟练使用简单的Linux命令如 `pwd` `ls` `cd`  `mkdir` `rmdir` `touch` `mv` `rm`
2. 能清晰的描述文件权限 `chmod` `chown`
3. 能简单使用 `find` `grep` `less` `tail` `head` `zip` `tar`


# Linux目录与文件命令  目标：形成肌肉记忆
# 以上就是Day02的全部内容，主要是多练，多敲一敲
































