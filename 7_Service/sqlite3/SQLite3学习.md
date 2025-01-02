# 1. SQLite3 介绍

SQLite3 是一种轻量级的关系型数据库管理系统（RDBMS），它以跨平台、零配置、服务器-less 的方式存储数据。

SQLite3 不像其他常见的数据库管理系统，如 MySQL 或 PostgreSQL 那样需要一个独立的服务器进程，在应用程序内部直接操作文件来进行数据存储和读取。

SQLite3 非常适合于嵌入式设备和单机应用程序等场景，因为它不需要占用太多资源，也允许在不同的平台上运行。此外，SQLite3 支持大多数 SQL 语法，并且还提供了一些高级功能，如触发器、存储过程等。

# 2. SQLite3 的优势和特性

- 跨平台性：SQLite3 可以在多种操作系统和编程语言下使用，包括 Windows、Linux、macOS、iOS、Android 等 
- 零配置：SQLite3 的特点之一是不要求任何服务器或网络配置。只需将数据库文件嵌入应用程序即可轻松地访问数据
- 体积小：SQLite3 的核心库非常小，通常只有几百 KB，因此非常适合在资源受限或空间受限的系统中使用
- 支持 SQL：SQLite3 支持大多数标准 SQL 查询语言，使用户能够使用大多数传统数据库管理任务
- ACID 兼容：SQLite3 支持 ACID（原子性、一致性、隔离性和持久性）事务处理，确保数据始终处于一致状态
- 高可靠性：SQLite3 对于频繁读取和少量更新的场景，表现出色。由于其自动记录更改，以防止损坏和数据丢失
- 强大的 API：SQLite3 提供了一个简单易用的 C 语言 API 来操作数据库，同时也提供了大量的接口和工具
- 可扩展性：SQLite3 允许用户创建自己的函数和存储过程，从而增加了其灵活性和可扩展性

# 3. SQLite3 安装

官网：https://www.sqlite.org/

下载网址：https://www.sqlite.org/download.html

这里我推荐的是使用源代码下载，源代码下载之后的文件主要有四个：shell.c  sqlite3.c  sqlite3ext.h  sqlite3.h

sqlite3 是 C 语言编写的，是以 .c 结尾的文件。因此它无法直接参与 g++编译，需要先编译为一个动态库。以 C 编译的动态库是可以直接参与到 g++ 的编译当中的。

SQLite3 动态库的编译命令: ` gcc -o libsqlite3.so -shared sqlite3.c -fPIC `

SQLite3 Shell 工具编译命令: ` gcc shell.c sqlite3.c -lpthread -ldl -lm -o sqlite3 `

注意：SQLite3 Shell 工具是可以直接打开并操作数据库文件 .db 文件的。

通常我们在使用 SQLite 库时，头文件仅需要 sqlite3.h 即可，如有特殊需要可查阅相关文档。

# 4. 基本使用

## 4.1 打开或关闭数据库

打开或关闭数据库是一个最基本的操作命令。只有成功生成并打开一个数据库文件，才能利用其 db 连接句柄进行应用操作。其函数使用：

```c++
// 声明一个sqlite数据库连接句柄，该句柄是一个指针类型
sqlite3 *db;

// 参数一指定生成的db文件名，参数二传入句柄指针地址
int ret = sqlite3_open("CustInfo.db", &db);
if( ret ){
    // sqlite3_errmsg函数返回最新错误信息
    fprintf(stderr, "Can't open database: %s\n", sqlite3_errmsg(db));
    // 释放db句柄资源，关闭数据库操作
    sqlite3_close(db);
}
```

## 4.2 SQL语句执行

负责数据库 SQL 语句的执行是 sqlite_exec 函数，使用为：

```c++
char* zErrMsg = nullptr;
const char* sql = "......";

int ret = sqlite3_exec(db, sql, callback, NULL, &zErrMsg);
if (ret != SQLITE_OK) {
    sqlite3_free(zErrMsg);
}
```

函数原型声明：

```c++
int sqlite3_exec(
  sqlite3*,                                  /* An open database */
  const char *sql,                           /* SQL to be evaluated */
  int (*callback)(void*,int,char**,char**),  /* Callback function */
  void *,                                    /* 1st argument to callback */
  char **errmsg                              /* Error msg written here */
);
```

要注意的是若 SQL 语句执行失败，系统会申请内存空间将错误信息写入该空间下，并由 zErrMsg 指针指向它。因此我们使用完错误信息后要及时的使用 sqlite3_free 函数释放报错资源。

同时 sqlite3_exec 接口是 sqlite3_prepare_v2、sqlite3_step 和 sqlite3_finalize 的方便封装，允许应用程序运行多条 SQL 语句，而无需使用大量 C 代码。

## 4.3 SQL 语句执行（等价函数）

这里不详细讲述函数的具体内容，只给出使用代码，使读者能够更好直观感受接口使用：

```c++
inline sqlite3_stmt* preIns(sqlite3* db) {
    std::string sql("insert into CustInfo values (?, ?, ?, ?, ?);");
    sqlite3_stmt* pStmt;
    int ret = sqlite3_prepare_v2(db, sql.c_str(), -1, &pStmt, NULL);
    if (ret != SQLITE_OK) {
        std::cout << "ret = " << ret << ":" << std::string(sqlite3_errmsg(db)) << std::endl;
        return nullptr;
    }
    return pStmt;
}
```

- sqlite3_stmt 类型主要存放用于字节序的 SQL 代码
- sqlite3_prepare_v2 函数会将字符串形式的 SQL 语句翻译为字节序的 SQL 代码。

```c++
//---------------------------------------------- 数据占位处理 --------------------------------------------

inline void insert(sqlite3* db, sqlite3_stmt* pStmt, CustMsg& msg) {
    sqlite3_bind_int(pStmt, 1, msg.custid);
    sqlite3_bind_text(pStmt, 2, msg.name.c_str(), -1, SQLITE_STATIC);
    sqlite3_bind_int(pStmt, 3, msg.sex);
    sqlite3_bind_text(pStmt, 4, msg.text.c_str(), -1, SQLITE_STATIC);
    sqlite3_bind_int(pStmt, 5, msg.flag);

    int ret = sqlite3_step(pStmt);
    std::cout << "ret = " << ret << ":" << __FUNCTION__ << std::endl;

    ret = sqlite3_reset(pStmt);
    std::cout << "ret = " << ret << ":" << __FUNCTION__ << std::endl;
}
```

- sqlite3_bind_* 函数会检测到字节序 SQL 代码中的占位符 ？符号（不只有这一种），并将其占位符替换为指定的属性值。不过在使用时得注意属性值得类型是什么，选择对应得类型的 sqlite3_bind 函数。
- sqlite3_step 函数将执行字节序 SQL 代码内容，进行相应的数据库的操作。
- 经过 sqlite_bind 处理过的 sqlite3_stmt 命令是已经写死了的，如果想重复使用该预处理的字节序 SQL 代码，可以调用 sqlite3_reset 函数，将写死的 SQL 命令进行充值，重新需要我们用 sqlite3_bind 函数进行占位替换。
- 由于 sqlite3_stmt 存有预处理过的字节序 SQL 代码，并且 sqlite3_step 函数仅负责语句执行，因此当我们不需要预处理的 SQL 命令时需要手动调用 sqlite3_finalize 函数对其 pStmt 进行资源释放。

# 5. 数据库配置

SQLite3 也提供了一系列的函数供我们对数据库进行配置：

## 5.1 多线程配置

Serialzed 模式下，SQLite 启用了所有的互斥锁，包括对数据库连接和预编译语句对象的递归互斥锁

```c++
// 数据库的多线程配置
int ret = sqlite3_config(SQLITE_CONFIG_SERIALIZED);
```

SQLite 中三种不同的线程模式配置选项：SQLITE_CONFIG_MULTITHREAD 和 SQLITE_CONFIG_SERIALIZED 与 SQLITE_CONFIG_SINGLETHREAD

- SQLITE_CONFIG_MULTITHREAD：将 SQLite 的线程模式设置为 Multi-thread，即多线程模式。
- SQLITE_CONFIG_SERIALIZED：将 SQLite 的线程模式设置为 Serialized，即序列化模式。
- SQLITE_CONFIG_SINGLETHREAD：单线程模式

SQLITE_CONFIG_MULTITHREAD：在这种模式下，SQLite 禁用了对数据库连接和预编译语句对象的互斥锁（mutex），即不会自动处理多线程访问同一个数据库连接或预编译语句的情况。应用程序必须自己来保证对数据库连接和预编译语句的访问是序列化的，即在任意时刻只有一个线程能够使用特定的数据库连接或预编译语句。其他的互斥锁仍然启用，因此 SQLite 在其他方面仍然是安全的，只要不同线程不同时访问同一个数据库连接即可。

SQLITE_CONFIG_SERIALIZED：在该模式下，SQLite 启用了所有的互斥锁，包括对数据库连接和预编译语句对象的递归互斥锁。SQLite 会自己处理对数据库连接和预编译语句的访问序列化，这样应用程序可以自由地在多个线程中同时使用同一个数据库连接或预编译语句，而不需要额外的同步措施。这是当 SQLite 编译时指定了 SQLITE_THREADSAFE=1 时的默认模式。

总结：
- 如果应用程序选择使用 SQLITE_CONFIG_MULTITHREAD，则必须确保在多个线程中正确地序列化对数据库连接和预编译语句的访问。
- 如果选择 SQLITE_CONFIG_SERIALIZED，SQLite 将自动处理并序列化对数据库连接和预编译语句的访问，使得应用程序能更方便地在多线程环境中使用 SQLite。

选择适当的线程模式取决于应用程序的具体需求和实现方式，以确保在多线程环境中使用 SQLite 时的安全性和性能。
## 5.2 回滚日志模式配置

```c++
// 设置回滚日志模式为WAL，用于提高多线程并发量
int ret = sqlite3_exec(db, "PRAGMA journal_mode = WAL", NULL, NULL, NULL);
```

## 5.3 设置文件锁定模式

```c++
// 设置文件独占锁，只允许一个进程访问
int ret = sqlite3_exec(db, "PRAGMA locking_mode = EXCLUSIVE ;", NULL, NULL, NULL);
```


## 5.4 本地同步配置

```c++
int ret = sqlite3_exec(db, "PRAGMA synchronous = NORMAL;", NULL, NULL, NULL);
```

数据库本地同步配置命令（查询或更改同步标志的设置）：

```mysql
# 1、查询形式：将以整数形式返回同步设置的结果
PRAGMA schema.synchronous;

# 2、更改形式：更改同步设置
PRAGMA schema.synchronous = 0 | OFF | 1 | NORMAL | 2 | FULL | 3 | EXTRA;
```

各种同步设置的含义如下：

- OFF (0)：设置为此时，SQLite 会在将数据移交给操作系统后继续运行而不同步。如果运行 SQLite 的应用程序崩溃，数据将是安全的，但如果在数据写入磁盘表面之前操作系统崩溃或计算机断电，数据库可能会损坏。另一方面，该设置会让事务的提交速度可以快上几个数量级。
- NORMAL (1)：设置为此时，SQLite 数据库引擎仍会在最关键时刻同步，但频率低于 FULL 模式。在较旧的文件系统上，在 journal_mode=DELETE 下，在错误的时间断电可能会损坏数据库，这种可能性非常小（虽然不是零）。在 synchronous=NORMAL 的情况下，WAL mode 不会损坏数据库，在现代文件系统上，DELETE 模式可能也是安全的。在 synchronous=NORMAL 的情况下，WAL 模式始终保持一致，但 WAL 模式会失去持久性。在 synchronous=NORMAL 的 WAL 模式下提交的事务可能会在断电或系统崩溃后回滚。无论同步设置或日志模式如何，事务在应用程序崩溃时都是持久的。对于大多数在 WAL 模式下运行的应用程序来说，同步=NORMAL 设置是个不错的选择。
- FULL (2)：设置为此时，SQLite 数据库引擎将使用 VFS 的 xSync 方法，确保在继续之前将所有内容安全写入磁盘表面。这可确保操作系统崩溃或断电不会损坏数据库。完全同步非常安全，但速度也较慢。在不使用 WAL 模式时，FULL 是最常用的同步设置。
- EXTRA (3)：EXTRA 同步与 FULL 同步类似，但在 DELETE 模式下提交事务时，包含回滚日志的目录会在取消日志链接后同步。EXTRA 可在提交后紧接着断电的情况下提供额外的耐用性。

# 6. 使用例子

```c++
#include <iostream>
#include <sqlite3.h>
 
int main() {
    sqlite3* db;
    char* errMsg = nullptr;
    int rc;
 
    // 打开数据库连接
    rc = sqlite3_open(":memory:", &db);
    if (rc != SQLITE_OK) {
        std::cerr << "Cannot open database: " << sqlite3_errmsg(db) << std::endl;
        sqlite3_close(db);
        return 1;
    }
 
    // 创建一个表
    const char* createTableSQL = "CREATE TABLE IF NOT EXISTS users (id INTEGER PRIMARY KEY, name TEXT, email TEXT);";
    rc = sqlite3_exec(db, createTableSQL, nullptr, nullptr, &errMsg);
    if (rc != SQLITE_OK) {
        std::cerr << "SQL error: " << errMsg << std::endl;
        sqlite3_free(errMsg);
        sqlite3_close(db);
        return 1;
    }
 
    // 插入数据
    const char* insertSQL = "INSERT INTO users (name, email) VALUES ('Alice', 'alice@example.com');";
    rc = sqlite3_exec(db, insertSQL, nullptr, nullptr, &errMsg);
    if (rc != SQLITE_OK) {
        std::cerr << "SQL error: " << errMsg << std::endl;
        sqlite3_free(errMsg);
        sqlite3_close(db);
        return 1;
    }
 
    // 查询数据
    sqlite3_stmt* stmt;
    rc = sqlite3_prepare_v2(db, "SELECT id, name, email FROM users;", -1, &stmt, nullptr);
    if (rc != SQLITE_OK) {
        std::cerr << "Failed to prepare statement: " << sqlite3_errmsg(db) << std::endl;
        sqlite3_close(db);
        return 1;
    }
 
    while ((rc = sqlite3_step(stmt)) == SQLITE_ROW) {
        int id = sqlite3_column_int(stmt, 0);
        const unsigned char* name = sqlite3_column_text(stmt, 1);
        const unsigned char* email = sqlite3_column_text(stmt, 2);
        std::cout << "ID: " << id << ", Name: " << name << ", Email: " << email << std::endl;
    }
 
    sqlite3_finalize(stmt);
 
    // 关闭数据库连接
    sqlite3_close(db);
 
    return 0;
}
```
