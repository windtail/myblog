---
title: "lua简单操作sqlite3"
date: 2012-01-08T22:12:00+08:00
draft: false
---

luasql模块支持sqlite3，可以完成最基本的数据库功能，不过官方文档上写得不是很详细。扫了下源代码，外加实验了下，得出了点经验。


环境
==


Windows XP，[LuaForWindows](code.google.com/p/luaforwindows)


代码
==



```
require"luasql.sqlite3"

function enumSimpleTable(t)

         print"-------------------"

         fork,v in pairs(t) do

                   print(k, " = ", v)

         end

         print"-------------------\n"

end

function rows(cur)

         returnfunction(cur)

                   localt = {}

                   if(nil~= cur:fetch(t, 'a')) then return t

                   elsereturn nil end

         end,cur

end

env = assert(luasql.sqlite3())

db =assert(env:connect("test.db"))

 
db:setautocommit(false)

res = assert(db:execute [[CREATE TABLEpeople(name text, sex text)]])

res = assert(db:execute [[INSERT INTOpeople VALUES('程序猿','男')]])

res = assert(db:execute [[INSERT INTOpeople VALUES('程序猿老婆', '女')]])

assert(db:commit())

 

res = assert(db:execute [[SELECT * FROMpeople]])

colnames = res:getcolnames()

coltypes = res:getcoltypes()

enumSimpleTable(colnames)

enumSimpleTable(coltypes)

for r in rows(res) do
    enumSimpleTable(r)
end

res:close()

db:close()

env:close()
```

结论
==


environment对象（数据库驱动）
--------------------


### 构造


    env = luasql.sqlite3()


### 成员


    close()


        关闭环境。请先关闭所有connection对象。


connection对象（数据库连接）
-------------------


### 构造


    con = env:connect(sqlite3\_database\_file\_path)


* env：环境
* sqlite3\_database\_file\_path：数据库路径字符串
* con：数据库连接对象


### 成员


    res = execute(sql\_statement)


        执行sql语句


* sql\_statement：要执行的sql语句，一般用[[sql]]的字符串形式，因为sql语句中常含有引号等特殊字符
* res：
	+ 对于不返回记录集的sql语句（如CREATE，DELETE），res是一个数字，表示操作影响了多少记录。
	+ 对于返回记录集的sql语句（如SELECT），res是一个cursor对象（记录集）


    setautocommit(bAuto)


* bAuto
	+ 设置为true时，rollback当前事务，并忽略错误
	+ 设置为false时，开始一个新的事务


    commit()


        提交事务


    rollback()


        回滚事务


    close()


        关闭数据库连接，请先关闭所有的cursor对象


    rowed = getlastautoid()


        获取最近一次自动生成的sqlite的rowid字段值（对应sqlite3\_last\_insert\_rowid()）


### 


### 


cursor对象
--------


### 构造


    con:execute(select\_sql\_statement)


### 成员


    colnametable = getcolnames()


        以table形式返回记录集中每一列的列名


    coltypetable = getcoltypes()


        以table形式返回记录集中每一列的类型


    res = fetch([table[,modestring]])


        获取下一个记录集


* table：如果指定了table，则会将数据复制到table中
* modestring：可取值”n”或”a”，默认为”n”，
	+ “n”：table中的key是数字
	+ “a”：table中的key是字符串（列名）
* res：如果未指定table，则返回记录，如果指定了table，则返回修改后的table；如果没有下一记录了，则返回nil


    close()


        关闭cursor对象  

