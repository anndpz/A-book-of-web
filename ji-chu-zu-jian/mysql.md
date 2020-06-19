---
description: mysql2
---

# Mysql

## mysql2

mysql 作为一个数据库，本身就是来存储数据的。废话。`SQL structured Query language` 也就是结构性查询语言，这里有一个关键字就是，结构性。y？ 因为我的表的结构是固定的，每一列的类型，字段都是已经确定了的。可以类比与 Excel，后面可能回接触到 nosql，这个就是不结构性的了。

### 快速开始

在mysql 开始使用的时候，可能有点逻辑上的东西，我们这里需要来进行理解一下，就是这里的异步查询来怎么进使用呢？这里来给一个例子来理解这个东西：

这里是DB里面使用的函数，如果使用了一个异步的方法怎么办呢？就要使用异步的调用方式。实例代码如下面：

```javascript
var mysql = require('mysql2');
let config = {
    host     : 'localhost',
    user     : 'root',
    password : 'toor',
    database : 'work',
    port:3306,
    multipleStatements: true//允许多条sql同时执行
};
let pool = mysql.createPool(config);
let query = (sql, values) => {
    return new Promise((resolve, reject) => {
          // 这里是一个异步回调的过程
        pool.getConnection((err, connection) => {
            if (err) {
                reject(err)
            } else {
                // 这里又是一个异步回调的过程
                connection.query(sql, values, (err, rows) => {
                    if (err) {
                        reject(err)
                    } else {
                        resolve(rows)
                    }
                    connection.release()
                })
            }
        })
    })
};
```

### 同步异步

异步回调这里的思想最大的不一样的是，函数本身是不会返回一个值的，而是把值传入了回调函数中，为了拿到这个回调函数的值，我们这里只能使用 Promise 结构。上面的代码看不懂？没关系，如果我们假定是同步的过程，我们会得到下面的的代码（当然这段代码是无法正常工作的）

```javascript
// 如果这里的db是同步的
function Q() {
  conn = pool.getConnection()
  result = connection.query('select * from tb0')
    return result
}
```

所以在真实的代码里面是使用 resolve来实现异步的值传递，上面的代码如果我们简化一下可以得到：

```javascript
let query = (sql, values) => {
  return new Promise((resolve, reject) => {
    // 这里是一个异步回调的过程
      pool.getConnection((err, connection) => {
      // 这里又是一个异步回调的过程
        connection.query(sql, values, (err, rows) => {
            // 这里使用 resolve把值，返回给返回的promise
          resolve(rows)
          connection.release()
        })
      }
    })
  })
};
```

### 如何使用

有了上面的基础函数了这里来看如何使用，由于我们知道，query是一个返回promise的函数，如果希望这些promise 是同步的执行，就需要使用到 await的结构，正如下面的，就可以用来得到 sql 查询的结果了。

```text
getDataById: async (id) => {
  a = await query('SELECT data from rand where id = ?',[id])
  if (a.length != 0) {
        return a[0];
  } else {
      return null;
  }
}
```

## 

