# 基本命令

```postgresql
-- 启动postgresql
pg_ctl start
```



# debug步骤

1. 运行PGDev下的compile.sh

```bash
source ./env-debug

cd ./postgresql-12.5

make distclean

./configure --prefix=/home/suhan_2020152064/PGDev/target-debug \
        --enable-debug \
        CCFLAG="-g -O0" CC='gcc'

make -j 1 world

make install-world

cd ..

echo 'Run successful.'
```



2. 在tutorial目录下执行make clean，之后执行make install

3. 进入到~/PGDev/pghome/share/postgresql/contrib目录下
4. 修改sql目录的删除类型操作，G移动到文件末尾，dd删除一行

5. 创建数据库createdb testdb
6. 将sql文件导入db，psql -f complex.sql testdb



7. 查看postgresql进程 ps -ef | grep postgres
8. 进入~/PGDev/pghome/bin文件下，使用gdb
9. 执行file postgres

![image-20230327220252104](C:\Users\ALIENWARE\AppData\Roaming\Typora\typora-user-images\image-20230327220252104.png)

10. attach （进程id），在步骤7中查看

11. 在postgres中执行操作语句
12. 在gdb下执行continue



加断点调试

1. 在.c文件中查看需要加断点的行数

![image-20230327220714737](C:\Users\ALIENWARE\AppData\Roaming\Typora\typora-user-images\image-20230327220714737.png)





# Join

实现连接的三种方法：

- 嵌套循环
- 排序合并
- 哈希



## 哈希连接

当join的谓词形式为：$T1.attr1 = T2.attr2$ 时，可以使用哈希连接进行运算，其中T1与T2分别为连接的两个关系，其中一个被指定为inner relation（T1），另一个被指定为outer relation（T2）



> 连接过程

常规哈希连接的阶段：

- 构建阶段
- 探测阶段



构建阶段：

对inner relation进行扫描，使用哈希函数f计算每个元组的连接属性的哈希值。随后元组插入到哈希表中



探测阶段：

扫描outer relation，寻找与inner relation匹配的元组。匹配过程分为两步，一、先使用相同的哈希函数f计算outer relation的连接属性的哈希值；二、如果哈希值对应桶不为空，匹配桶内的属性值，若真实值匹配成功











# 索引























