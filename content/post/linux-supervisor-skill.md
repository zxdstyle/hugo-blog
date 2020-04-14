---
title: "生产环境中使用 supervisor 运行laravel队列任务"
date: 2019-04-01
lastmod: 2019-04-01
draft: false
tags: ["Linux", "Supervisor", "Laravel"]
categories: ["Linux", "Laravel"]
author: "zxdstyle"
contentCopyright: '本站所有文章无特殊说明都是原创，转载请注明出处<a rel="license noopener" href="https://www.zxdstyle.cn" target="_blank">www.zxdstyle.cn</a>'

---

# 安装 supervisor
首先肯定需要安装 `supervisor` 这个包，我的服务器是 `centos 7`，使用如下命令装：
```bash
yum install supervisor
```

# 调整配置
在 `centos 7` 系统中，supervisor 的配置文件默认存放路径是 `/etc/supervisord.conf` 

修改配置文件：
```bash
[program:laravel-worker]
process_name=%(program_name)s_%(process_num)02d
command=php /path/to/your-project/artisan queue:listen --tries=3
autostart=true
autorestart=true
user=nobody
numprocs=8
redirect_stderr=true
stdout_logfile=/path/to/your-project/worker.log
```

# 启动服务
然后启动 `supervisord` 服务，使用 `systemctl start supervisord`

进程起来之后，再执行以下命令：
```bash
supervisorctl start laravel-worker:*
```


# 常用命令

## Laravel
重启队列：
```bash
php artisan queue:restart
```

运行队列监听器（执行队列）
```bash
php artisan queue:listen
```

## supervisord
更新新的配置到 `supervisord`
```bash
supervisorctl update
```

重新启动配置中的所有程序
```bash
supervisorctl reload
```
启动某个进程(program_name=你配置中写的程序名称)
```bash
supervisorctl start program_name
```

查看正在守候的进程
```bash
supervisorctl
```

停止某一进程 (program_name=你配置中写的程序名称)
```bash
pervisorctl stop program_name
```

重启某一进程 (program_name=你配置中写的程序名称)
```bash
supervisorctl restart program_name
```

停止全部进程
```bash
supervisorctl stop all
```
