# HotelManagementSys —— 2023数据库课程设计

## 1.	选题说明：
  在当前形式下，酒店管理系统的重要性得到了更加显著的体现。

  疫情期间，酒店需要采取各种措施降低疫情传播风险，实现客户与员工的安全保障。

  而当前疫情已逐步控制，但与之相关的安全和卫生问题仍然是许多消费者非常关注的核心问题。

  因此，酒店经营者需要重视酒店管理系统的研发与建设，提升竞争力。

  该酒店管理系统的设计方案以提高服务质量和客户满意度为中心，并在此基础上加强细节管理，提高管理效率，降低运营成本。
## 2.	需求介绍：

  前台管理：

  1）客房预订：客户可以通过系统预约需要的房间，系统需要提供相应的房型、单价、入住时间、离店时间等信息。

  2）入住登记：客户入住酒店后，需要通过系统进行登记，填写个人基本信息、房间信息等，系统需要生成入住单据，同时分配客房。

  3）客房查询：客户可通过系统查询空房间信息、已预定房间信息、已入住客房信息等。

  4）收费管理：系统需要记录客户的消费信息，包括住宿费用、房间服务费、餐饮费用等，方便客户结账。

  后台管理：

  1）信息输入：负责系统基础数据的设置、修改、删除等功能，包括客房类型、楼层、房间编号、服务项目、收费标准等。

  2）客房信息安排：可对客房进行修改、删除等操作，并随时查看客房信息的最新更新。

  3）前台操作员管理：包括操作员的设置、权限管理、修改密码、删除等功能，保证系统安全。

  4）系统日志管理：记录管理员的操作日志，方便追溯操作过程和管理。

  5）统计报表：提供各种客房预订和入住的报表，如预定分析、入住分析、客房类型分析等。

  该系统的实现需要技术：Qt、MySQL等。


## 3. 建表:

``` SQL
create table user
(
    id           int auto_increment      --id
        primary key,
    name         varchar(20) not null,   --客户名
    sex          varchar(5)  null,       --性别
    id_card      varchar(30) not null,   --身份证号
    vip          varchar(20) null,       --贵宾身份
    phone_number varchar(20) null        --手机号
);

create table room_type                   --房间类型表
(
    id       int auto_increment          --类型id
        primary key,
    name     varchar(20) not null,       --房间名
    num      int         null,           --房间数量
    capacity int         null,           --容量
    price    double      null            --定价
);

create table room                        --房间信息表
(
    id            int auto_increment     --房间id
        primary key,
    type_id       int         not null,  --类型
    room_num      int         not null,  --房间号
    status        varchar(20) not null,  --状态
    checkin_time  datetime    null,      --入住时间
    checkout_time datetime    null,      --退房时间
    constraint room_room_type
        foreign key (type_id) references room_type (id)
);

create table receipt                     --单据信息表
(
    id            int auto_increment,    --单据id
    user_id       int      not null,     --客户id
    room_id       int      not null,     --房间id
    checkin_time  datetime not null,     --入住时间
    checkout_time datetime not null,     --退房时间
    room_price    double   null,         --房间费
    service_price double   null,         --服务费
    food_price    double   null,         --餐饮费
    primary key (id, room_id),
    constraint receipt_room
        foreign key (room_id) references room (id),
    constraint receipt_user
        foreign key (user_id) references user (id)
);

create table admin                       --管理员信息表
(
    id       int auto_increment          --管理员id
        primary key,
    name     varchar(20)  not null,      --姓名
    username varchar(30)  not null,      --登陆用户名
    id_card  varchar(30)  null,          --身份证号
    position varchar(20)  null,          --职位
    password varchar(200) null,          --密码
    root          int                    not null       --root身份
);

create table log                         --日志表
(
    id         int auto_increment        --日志id
        primary key,
    admin_id   int           not null,   --管理员id
    type       int default 1 null,       --1:插入2:更新3:删除
    table_name varchar(30)   null,       --表名
    table_id   int           null,       --表id
    comment    int           null,       --备注
    dt         datetime      not null,   --时间
    constraint log_admin_null_fk
        foreign key (admin_id) references admin (id)
);

create table cost_type (				--消费类型表
    id       int auto_increment primary key,    --类型id
    name     varchar(20) not null unique        --类型名，加入唯一性约束以避免重复定义
);

create table cost (					--消费记录表
    id         int auto_increment primary key,  --消费记录id
    receipt_id int not null,                    --单据id
    type_id    int not null,                    --消费类型id，引用cost_type表中的id
    price      double not null,                 --消费金额
    dt         datetime null,                   --消费时间
    comment    varchar(50) null,                --备注
    constraint cost_receipt_fk foreign key (receipt_id) references receipt (id),
    constraint cost_type_fk foreign key (type_id) references cost_type (id)
);

```

## 4. 视图：

``` SQL

-- 查询客户和对应房间的视图

create view user_room_info as 
select u.name as user_name, u.phone_number, r.room_num, r.status, r.checkin_time, r.checkout_time, rt.name as room_type, rt.capacity, rt.price
from user u 
join receipt rc on u.id = rc.user_id
join room r on rc.room_id = r.id
join room_type rt on r.type_id = rt.id;

-- 查询管理员和对应的日志信息的视图：

create view admin_log_info as
select a.name as admin_name, a.username, l.type, l.table_name, l.dt
from admin a
join log l on a.id = l.admin_id;

-- 查询每个房间最近一次入住记录的视图：

create view recent_checkin_info as
select r.room_num, u.name as user_name, rc.checkin_time, rc.checkout_time
from room r
left join (
    select room_id, max(checkin_time) as max_checkin_time
    from receipt
    group by room_id
) latest_rc on r.id = latest_rc.room_id
join receipt rc on r.id = rc.room_id and rc.checkin_time = latest_rc.max_checkin_time
join user u on rc.user_id = u.id;

```


## 5. 安全性设计

### 为了确保数据库的安全性，以下是一些可以实施的安全性保证措施：

  1. 数据备份和恢复计划：定期备份数据以防止数据丢失，制定可行的恢复计划以应对紧急情况。

  2. 访问控制：控制用户对数据库的访问权限，使用安全的身份验证方法，确保只有授权的用户可以访问数据库。

  3. 密码保护：使用强密码策略，确保所有用户都使用安全的密码，并定期更改密码以防止未经授权的访问。

  4. 安全审计：跟踪数据库的活动并记录安全事件，以及保留足够的信息以追踪数据库中的问题。

  5. 加密：使用加密技术来保护敏感信息，以确保只有授权的用户可以访问它们。

  6. 防火墙：使用防火墙保护数据库服务器，以确保只有授权的用户可以访问数据库服务器。

  7. 数据访问监控：实时监控对数据库的访问并对异常访问进行警报和响应。

  8. 安全补丁：定期检查并更新数据库和操作系统中的安全补丁，以确保安全漏洞得到及时修补。

  9. 安全培训：为数据库用户和管理员提供必要的安全培训，以提高安全意识和防范能力。

### 操作

   1. 用户权限控制：创建用户，授权用户

    创建用户：

    ```
    CREATE USER 'username'@'localhost' IDENTIFIED BY 'password';
    ```

    授权用户：

    ```
    GRANT SELECT, INSERT, UPDATE, DELETE ON database_name.table_name TO 'username'@'localhost';
    ```

   2. 数据备份：使用 mysqldump 命令进行备份

    ```
    mysqldump -u username -p database_name > backup_file.sql
    ```

   3. 数据库加密：使用数据库加密工具进行加密

    例如使用 MySQL Enterprise Transparent Data Encryption 进行加密：

    ```
    ALTER TABLE table_name ENCRYPTED=YES, ENCRYPTION_KEY_ID=1;
    ```

   4. 数据库审计：开启 MySQL 的审计日志功能

    开启审计日志：

    ```
    SET GLOBAL audit_log_file = '/var/log/mysql/audit.log';
    SET GLOBAL audit_log_format = 'JSON';
    SET GLOBAL audit_log_policy = ALL;
    ```

   5. 防止拒绝服务攻击：限制连接数、限制每个连接的资源使用量

    限制连接数：

    ```
    SET GLOBAL max_connections = 100;
    ```

    限制每个连接的资源使用量：

    ```
    SET GLOBAL read_rnd_buffer_size = 262144;
    SET GLOBAL sort_buffer_size = 262144;
    ```
