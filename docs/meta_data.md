# 元数据设计

## 元数据内容

### 文件/目录属性


key：file_name [ + chunk ]

value:  
user  
access  
lock  

文件/目录用户  
文件/目录权限数据  
文件/目录锁  

以上三种都是有限长度的数据，且只涉及本地单节点的数据。可以直接以KV进行存储。

### 目录下的文件索引


key: dir_name

value:  
[., .., file1, file2, file3]

一个目录下包含哪些文件，用于遍历目录。长度无限的值。
使用场景如下：  
1. read_dir接口返回该目录下的文件名。  
2. create_file/create_dir接口添加一个值
3. delete_file接口删除一个值
  
为了保证create和delete的效率，需要使用一种对随机增删友好的方式存储，例如使用map来记录，否则每次创建删除文件的时候都需要完全遍历整个文件索引数据。但同时持久化也是必要的，一方面保证宕机不丢数据，另一方面内存数据可能不够大。inode使用可能？

### 文件内容

key: file_name [ + chunk ]
value:
local_file_name
data

本地存储文件名
文件数据
使用场景两个：
1. write
2. read
这个设计倒是没有太多的余地，db存local_file_name，文件存data就好。可能需要考虑的，local_file的组织形式，此外直接block_read、block_write也是提升性能必要的。

### 其他信息

短期不需要实现，但任何设计都不能堵死扩展这些能力的可能性。

0. 数据共享。支持多客户端同步读写，锁的设计可以不完备，但需要有基本的功能或对客户端的要求。
1. 数据冗余（多副本、纠删码）
2. 扩缩容（一致性hash)
3. 容量管理（统计、限制）。这点比较重要，需要达到的目标是限制某个层级的数据容量，至少要能限制最根层目录的容量，这对于云原生场景非常有用。
4. POSIX标准的其他接口实现