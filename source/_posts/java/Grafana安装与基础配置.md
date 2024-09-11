### 一、安装

下载`Grafana` [https://grafana.com/grafana/download](https://grafana.com/grafana/download)，一般下载压缩包即可，直接解压可用

默认配置：

* 端口：3000，访问地址 `http://localhost:3000`
* 用户名、密码：`admin/admin`



### 二、配置

#### 1、自定义配置信息

默认配置文件：`$WORKING_DIR/conf/defaults.ini`，其中 `WORKING_DIR`指`Grafana`所在目录，**不要修改`defaults.ini`文件**，如果需要自定义配置，在 ``$WORKING_DIR/conf/`目录下新增文件 `custom.ini`(此种方式适用于使用压缩包形式安装`Grafana`，如果使用其他方式安装，需要参考：[https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/](https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/))



配置文件支持读取环境变量和外部文件，使用方式如下：

* 环境变量读取：格式： `$__env{KEY}` 或 `${KEY}`
* 外部文件读取：格式：`$__file{FILE_PATH}`



| 配置                                        | 描述                                                         | 默认值                    |
| ------------------------------------------- | ------------------------------------------------------------ | ------------------------- |
| `instance_name`                             | `grafana-server`实例名称                                     | `${HOSTNAME}`             |
| `[paths].logs`                              | `Grafana`日志存储路径，可以通过启动参数 `cfg:default.paths.logs=xxx`修改日志路径 | `${WORKING_DIR}/data/log` |
| `[server].protocol`                         | 协议，`http、https、socket`                                  | `http`                    |
| `[server].http_port`                        | `grafana-server`访问端口                                     | `3000`                    |
| `[server].cert_file`                        | `certificate file`路径（协议为`https`时）                    |                           |
| `[server].cert_key`                         | `certificate key`路径（协议为`https`时）                     |                           |
| `[database].type`                           | 数据库类型(`mysql`,`postgres`,`sqlite3`)，`grafana`需要数据库存储用户和`dashboards`等信息 | `sqlite3`                 |
| `[database].host`                           | 数据库连接`host`，包含端口，`host = 127.0.0.1:3306`          |                           |
| `[database].name`                           | 数据库名                                                     |                           |
| `[database].user`                           | 数据库连接用户名                                             |                           |
| `[database].password`                       | 数据库连接密码，如果密码包含`#`或`;`，需要使用三引号包括，例如`"""#password;"""` |                           |
| `[database].max_idel_conn`                  | 最大空闲连接数                                               |                           |
| `[database].max_open_conn`                  | 允许创建的最大连接数                                         |                           |
| `[database].conn_max_lifetime`              | 连接最大存活时间                                             | `14400s`                  |
| `[database].log_queries`                    | 打印`SQL`调用和执行时间                                      |                           |
| `[security].disable_initial_admin_creation` | 禁止在第一次启动时创建管理员用户                             | `false`                   |
| `[security].admin_user`                     | 默认的管理员账号，管理员拥有全部权限                         | `admin`                   |
| `[security].admin_password`                 | 默认的管理员密码                                             | `admin`                   |
| `[log].mode`                                | 日志输出位置，`console` `file` `syslog`,支持多个模式的时候用空格分割 | `console file`            |
| `[log].level`                               | 日志级别，`debug` `info` `warn` `error` `critical`           | `info`                    |
| `[log.file].level`                          | 指定文件日志级别                                             | 继承自`[log].level`       |
| `[log.file].format`                         | 日志输出格式，`text` `console` `json`                        | `text`                    |
| `[log.file].log_rotate`                     | 日志滚动                                                     | `true`                    |
| `[log.file].max_lines`                      | 每个文件允许的最大行                                         | `1_000_000`               |
| `[log.file].max_size_shift`                 | 文件大小                                                     | `28` `1 << 28 = 256MB`    |
| `[log.file].daily_rotate`                   | 允许按日滚动，`false` `true`                                 | `true`                    |
| `[log.file].max_days`                       | 日志文件最大保留天数                                         | `7`                       |



 完整配置参考：[https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/](https://grafana.com/docs/grafana/latest/setup-grafana/configure-grafana/)



详细使用手册参考：[https://grafana.com/docs/grafana/latest/](https://grafana.com/docs/grafana/latest/)

