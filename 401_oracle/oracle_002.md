# oracle中斜线(/)和分号(;)问题


# 一、问题描述

公司最近有个项目要上线，数据库用的是oracle11g，整理的sql脚本时发现，有些sql单个执行时都可以正常执行，放到sql文件中批量执行时就报错。

经过排查，发现是由于触发器结尾的分号(;)并不能执行创建该触发器，需要添加斜线来执行。

# 二、解决方法

出现上面问题的原因是：

1、**对于sql语句(如insert、update等),`;`结尾表示sql结束，并且会执行sql。**如：

```
insert into skd_score_card_cycle values (1,1,1,0,sysdate,0,sysdate);
insert into skd_score_card_cycle values (2,2,1,0,sysdate,0,sysdate);
```


2、**对于sql语句块、pl块，`;`表示sql结束，但运行sql，需要用`/`来执行。`/`作用是让服务器开始执行前面所写的sql脚本。**如下面触发器结尾需要加个`/`来执行。

```
create or replace trigger tr_data_score
before insert on skd_data_score
for each row
begin
  select seq_data_score.nextval into :new.data_id from dual;
end;  
/

create or replace trigger tr_data
before insert on skd_data
for each row
begin
  select seq_data.nextval into :new.data_id from dual;
end; 
/

```

