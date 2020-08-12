本文是掘金小册子《MySQL 是怎样运行的：从根儿上理解 MySQL》总结的内容
- [MySQL 是怎样运行的：从根儿上理解 MySQL](https://juejin.im/book/5bffcbc9f265da614b11b731/section/5bffda656fb9a049b13deba8)

### 字符集和比较规则
1. 字符集指的是某个字符范围的编码规则。
2. 比较规则是针对某个字符集中的字符比较大小的一种规则
3. 在MySQL中，一个字符集可以有若干种比较规则，其中有一个默认的比较规则，一个比较规则必须对应一个字符集。
4. 查看MySQL中查看支持的字符集和比较规则的语句如下：
   ```php
   SHOW (CHARACTER SET|CHARSET) [LIKE 匹配的模式];
   SHOW COLLATION [LIKE 匹配的模式];
   ```
5. MySQL有四个级别的字符集和比较规则
   1. 服务器级别: character_set_server表示服务器级别的字符集，collation_server表示服务器级别的比较规则。
   2. 数据库级别, 创建和修改数据时候可以指定字符集和比较规则
   3. 表级别, 创建和修改表的时候指定表的字符集和比较规则
   4. 列级别，创建和修改列定义的时候可以指定该列的字符集和比较规则
6. 从发送请求到接收结果过程中发生的字符集转换
   1. 客户端使用操作系统的字符集编码请求字符串，向服务器发送的是经过编码的一个字节串。
   2. 服务器将客户端发送来的字节串采用character_set_client代表的字符集进行解码，将解码后的字符串再按照character_set_connection代表的字符集进行编码。
   3. 如果character_set_connection代表的字符集和具体操作的列使用的字符集一致，则直接进行相应操作，否则的话需要将请求中的字符串从character_set_connection代表的字符集转换为具体操作的列使用的字符集之后再进行操作。
   4. 将从某个列获取到的字节串从该列使用的字符集转换为character_set_results代表的字符集后发送到客户端。
   5. 客户端使用操作系统的字符集解析收到的结果集字节串。
    系统变量| 描述
    ---|---
    character_set_client | 服务器解码请求时使用的字符集
    character_set_connection | 服务器处理请求时会把请求字符串从character_set_client转为character_set_connection
    character_set_results | 服务器向客户端返回数据时使用的字符集

7. 比较规则的作用通常体现比较字符串大小的表达式以及对某个字符串列进行排序中