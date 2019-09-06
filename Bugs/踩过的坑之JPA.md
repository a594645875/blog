#### 报错:Can not issue data manipulation statements with executeQuery().
- 报错:Can not issue data manipulation statements with executeQuery().

- 分析:一开始以为缺少继承JpaSpecificationExecutor<T>类,实际不需要继承该类
实际问题是没有添加@Modifying注解,

- 解决办法:在方法上添加 @Modifying
#### 报错:保存方法xxxDao.save(xxx);报红Inferred type 'S' for type parameter 'S' is not within
报错:保存方法`xxxDao.save(xxx);`报红`Inferred type 'S' for type parameter 'S' is not within its bound;`

分析: dao层泛型写错

办法:把Dao层的泛型
```
public interface NoFriendDao extends JpaRepository<NoFriendDao,String> {}
```
中的
```
JpaRepository<NoFriendDao,String>改为JpaRepository<NoFriend,String>
```
#### Jpa dao层报错
报错:
```
com.mysql.jdbc.exceptions.jdbc4.MySQLSyntaxErrorException: Unknown column 'friend0_.user_id' in 'field list
```

分析: dao层query语言中用了userId,friendId,表中实际是userid,friendid,h语言中列名和表中列名不堆成

办法:把dao层中的userId和friendId改成userid,和friendid

