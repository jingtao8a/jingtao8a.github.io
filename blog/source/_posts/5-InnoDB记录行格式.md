---
title: 5.InnoDB记录行格式
date: 2023-05-27 14:17:56
tags: MySQL
categories: 数据库
---
#InnoDB页简介
>InnoDB是一个将表中的数据存储到磁盘上的存储引擎。由于磁盘IO和内存IO速度差了几个量级，InnoDB采取的方式是：**将数据划分为若干个页，以页作为磁盘和内存之间的交互的基本单位，InnoDB中页的大小一般为16KB**。

#InnoDB行格式
>我们平时是以记录为单位来向表中插入数据的，这些记录在磁盘上的存放方式也被称为**行格式**或者**记录格式**
###指定行格式的语法
```
CREATE TABLE 表名 (列的信息) ROW_FORMAT=行格式名称
    
ALTER TABLE 表名 ROW_FORMAT=行格式名称

```
###COMPACT行格式
![QQ截图20221205160357.png](https://upload-images.jianshu.io/upload_images/28458225-3383c0dce9fc46f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

一条完整的记录其实可以被分为记录的额外信息和记录的真实数据两大部分。
- 记录的额外信息：服务器为了描述这条记录而不得不添加的一些信息，分为3类，变长字段长度列表、NULL值列表、记录头信息
1.变长字段长度列表：MySQL支持一些变长的数据类型，比如VARCHAR(M)、VARBINARY(M)、各种TEXT类型，各种BLOB类型，这些变长字段占用的存储空间分为两部分（真正的数据内容和占用的字节数），对于CHAR(M）类型的列来说，当列采用的是定长字符集时，该列占用的字节数不会被加到变长字段长度列表，而如果采用变长字符集时，该列占用的字节数也会被加到变长字段长度列表
**在COMPACT行格式中，把所有的变长字段的真实数据占用的字节长度都存放在记录的开头部位，从而形成一个变长字段长度列表，按逆序排放**
2.NULL值列表：处理过程是这样的，先统计表中哪些列允许存储NULL值(主键列、被NOT NULL修饰的列都是不可以存储NULL值的)，所以在统计的时候不会把这些列算进去，接着如果有的列可以存储NULL值，那么就需要NULL值列表，将每个允许存储NULL的列对应一个二进制位（为1代表该列值为NULL，为0代表不为NULL），其次MySQL规定NULL值列表必须用整数个字节的位表示，如果使用的二进制位个数不是整数个字节，则在字节的高位补0
3.记录头信息：它是由固定的5个字节组成，不同位代表不同意思
![QQ截图20221205161836.png](https://upload-images.jianshu.io/upload_images/28458225-900c799de838c944.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 记录的真实数据
MySQL会为每个记录默认的添加一些列（也称为隐藏列），具体的列如下：
![QQ截图20221205162406.png](https://upload-images.jianshu.io/upload_images/28458225-21846d1bd89aaaa8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
InnoDB表对主键的生成策略:优先使用用户自定义主键作为主键，如果用户没有定义主键，则选取一个Unique键作为主键，如果表中连Unique键都没有定义的化，则InnoDB会为表默认添加一个名为row_id的隐藏列作为主键

###Redundant行格式
![QQ截图20221207112201.png](https://upload-images.jianshu.io/upload_images/28458225-b02bf154c412a842.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
- 记录的额外信息
1.字段长度偏移列表：与compact行格式相比，没有了变长两个字，多了偏移两个字，Redundant的行格式会把该条记录中所有（包括隐藏列）的长度信息都按照逆序存储到字段长度偏移列表。同时Redundant的行格式是按照两个相邻数值的差值来计算各个列值的长度。
![QQ截图20221207115628.png](https://upload-images.jianshu.io/upload_images/28458225-1952905a3f74dca0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

- 记录头信息
![QQ截图20221207113712.png](https://upload-images.jianshu.io/upload_images/28458225-753e88f896831a7b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![QQ截图20221207113735.png](https://upload-images.jianshu.io/upload_images/28458225-df9ca63de546584f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

对于Compact和Reduntant行格式来说，如果某一列中的数据非常多的话，在本记录的真实数据处只会存储该列的前768个字节的数据和一个指向其它页的地址，然后把剩下的数据存放到其它页中，这个过程叫做**行溢出，存储超出768字节的那些页也被称为溢出页**