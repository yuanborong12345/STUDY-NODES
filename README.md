# 学习笔记

日常学习笔记，涵盖 Java 后端开发、数据库、微服务、微信小程序及算法等内容。

---

## 目录

- [Java](#java)
  - [Java 基础](#java-基础)
  - [Java 并发](#java-并发)
  - [Java 集合](#java-集合)
  - [Java IO](#java-io)
- [数据库](#数据库)
  - [MySQL](#mysql)
  - [Redis](#redis)
- [SpringCloud](#springcloud)
- [微信小程序开发](#微信小程序开发)
- [LeetCode 热题 100 道](#leetcode-热题-100-道)
- [项目](#项目)

---

## Java

### Java 基础

1. **概念**
   - Java 特点、优势与劣势
   - 跨平台原理（JVM）
   - JVM、JDK、JRE 的关系
   - 编译与解释
   - 值传递与引用传递
   - 标识符、关键字、运算符
   - 数组的初始化与常见方法
2. **数据类型**
   - 8 种基本数据类型
   - 类型转换（自动提升 & 强制转换）
   - 浮点数精度丢失问题
   - 装箱与拆箱、包装类缓存机制
   - 成员变量 vs 局部变量 vs 静态变量
   - 重载与重写、可变参数
3. **面向对象基础**
   - 面向对象三大特征：封装、继承、多态
   - 接口与抽象类的区别
   - 深拷贝与浅拷贝
   - Object 类：equals / hashCode / toString
   - String / StringBuffer / StringBuilder
   - 字符串常量池与 intern 方法
4. **异常**
   - Exception 与 Error
   - try-catch-finally 执行顺序
5. **泛型**
   - 泛型类、泛型接口、泛型方法
6. **反射**
   - 获取 Class 对象的四种方式
   - 反射操作类（字段、方法、构造器）
7. **代理**
   - 静态代理
   - JDK 动态代理 & CGLIB 动态代理
8. **序列化与反序列化**
   - transient 关键字
   - JDK 自带序列化
9. **注解**
   - 元注解（@Target、@Retention）
   - 自定义注解

### Java 并发

1. **创建线程的方式**
   - 继承 Thread 类
   - 实现 Runnable 接口
   - 实现 Callable 接口（可获取返回值）
2. **线程基础**
   - start() 与 run() 的区别
   - 线程状态（NEW / RUNNABLE / BLOCKED / WAITING / TIMED_WAITING / TERMINATED）
   - 并行与并发
   - 线程优先级
   - synchronized 可重入性
3. **线程池**
   - 为什么不推荐使用 Executors 工具类
   - 线程池状态与关闭（shutdown / shutdownNow）
   - ThreadPoolExecutor 七大核心参数
   - 核心线程数设置、工作队列选型
   - 四种拒绝策略
4. **CountDownLatch** — 同步工具类
5. **多线程通信协作 & 同步互斥**
   - 基于 wait / notify 的生产者-消费者模型
   - Condition 接口（基于 Lock 锁）

### Java 集合

1. **集合基础**
   - Collection vs Collections
   - 集合遍历方式（for / for-each / Iterator）
   - 快速失败（Fail-Fast）& 安全失败（Fail-Safe）
   - 循环中删除元素的正确姿势
   - List 与 Set 的区别
2. **List**
   - ArrayList：底层数组、扩容机制、线程安全
   - LinkedList：双向链表、常用方法
   - CopyOnWriteArrayList：写时复制、适用场景
3. **Map**
   - Map 遍历方式
   - HashMap：数组+链表+红黑树、put 流程、哈希冲突解决
   - Hashtable：全方法 synchronized、与 HashMap 的区别
   - ConcurrentHashMap：JDK 7 → JDK 8 的变化、读不加锁原理、协助扩容
4. **Set**
   - HashSet / TreeSet / LinkedHashSet
   - 去重原理（hashCode + equals / CompareTo）

### Java IO

1. **IO 流概念** — 输入流/输出流、字节流/字符流
2. **字节流**
   - InputStream：FileInputStream / DataInputStream / ObjectInputStream
   - OutputStream：FileOutputStream / DataOutputStream / ObjectOutputStream
3. **字符流**
   - Reader：FileReader
   - Writer：FileWriter
4. **字节缓冲流** — BufferedInputStream / BufferedOutputStream
5. **字符缓冲流** — BufferedReader / BufferedWriter
6. **IO 模型**
   - BIO（同步阻塞）
   - NIO（同步非阻塞）
   - IO 多路复用
   - AIO（异步非阻塞）

---

## 数据库

### MySQL

1. **连接与基本操作**
2. **数据库操作** — 创建、删除、查看
3. **表操作**
   - 创建表（字段类型、约束）
   - 修改表（增删改列、修改字符集）
   - CRUD（增删改查）
4. **运算符**
5. **关键字** — DISTINCT、ORDER BY、GROUP BY、HAVING、LIMIT 等
6. **函数** — 统计数学、字符串、时间日期
7. **事务**
   - 隔离级别（脏读、不可重复读、幻读）
   - 基础语法（开启事务、保存点、回滚、提交）
   - ACID 四大特性

### Redis

1. **基础知识** — 基于内存的键值型 NoSQL 数据库
2. **数据类型与命令**
   - String、Hash、List、Set、SortedSet
   - 通用命令（DEL / EXISTS / EXPIRE / TTL）
3. **Jedis & SpringDataRedis** — Java 客户端操作 Redis
4. **实战案例**
   - 基于 Redis 实现共享 Session 短信登录
   - 查询缓存（缓存更新策略、穿透/雪崩/击穿）
   - 全局唯一 ID 生成
   - 分布式锁（超时释放、原子性问题、Redisson）
   - Redis 消息队列实现异步秒杀（List / PubSub / Stream）
   - 基于 SortedSet 实现点赞排行榜
   - Feed 流推送（Timeline 模式、滚动分页）
5. **分布式缓存**
   - 持久化（RDB & AOF）
   - 主从复制（全量同步 & 增量同步）

---

## SpringCloud

1. **认识微服务**
   - 单体架构 → 分布式架构 → 微服务架构演进
   - SpringCloud 组件生态
2. **服务拆分与远程调用**
   - 服务拆分原则
   - RestTemplate 远程调用
3. **Eureka 注册中心**
   - 搭建 Eureka Server
   - 服务注册 & 服务发现
4. **Ribbon 负载均衡**
   - 负载均衡原理与源码分析
   - 负载均衡策略（轮询、随机、权重等）
   - 饥饿加载
5. **Nacos 注册中心**
   - 服务分级存储模型（集群优先负载均衡）
   - 权重配置 & 环境隔离（namespace）
   - Nacos 与 Eureka 的区别
6. **Nacos 配置管理**
   - 统一配置管理 & 配置热更新
   - 配置共享 & 优先级
   - Nacos 集群搭建
7. **Feign 远程调用**
   - 替代 RestTemplate、声明式 HTTP 客户端
   - 自定义配置 & 性能优化
   - 最佳实践（抽取方式）
8. **Gateway 服务网关**
   - 路由配置 & 断言工厂
   - 过滤器工厂（路由过滤器、默认过滤器）
   - 全局过滤器 & 过滤器执行顺序
   - 跨域问题解决

---

## 微信小程序开发

1. **测试号**
   - 申请微信小程序测试号
   - 项目创建与运行
   - 小程序结构说明（pages / utils / app.json / app.wxss / project.config.json）
   - 前后端联调测试
2. **项目迭代开发** — 运行流程
3. **微信登录**
   - 登录基本流程
   - 环境准备 & 真机调试

---

## LeetCode 热题 100 道

1. **哈希**
   - 两数之和
   - 字母异位词分组
   - 最长连续序列
2. **双指针**
   - 移动零
   - 盛水最多的容器
3. 持续更新中...

---

## 项目

- **AI 数据解析与智能分析平台** — 项目笔记（待完善）
