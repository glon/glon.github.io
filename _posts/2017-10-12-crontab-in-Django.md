---
layout: post
title:  "crontab in Django"
date:   2017-10-12
tags: Python Django
---

## 需求

> 有个小需求，需要定时统计每天 JIRA 发布过的任务。由于当前的 Web 框架用的是 Django，对这个成熟的框架而言，轮子多，解决方案自然不少，期间也有考虑过使用 Celery，但明显还用不着这个 "牛刀"，于是使用了 django-crontab 来解决。

## 安装

使用 pip 来安装：
```
pip install django-crontab
```

## 使用方法

```
vagrant@glon:~$ python manage.py crontab -h
usage: manage.py crontab [-h] [--version] [-v {0,1,2,3}] [--settings SETTINGS]
                         [--pythonpath PYTHONPATH] [--traceback] [--no-color]
                         {add,show,remove,run} [jobhash]


run this command to add, show or remove the jobs defined in CRONJOBS setting
from/to crontab

positional arguments:
  {add,show,remove,run}
  jobhash

optional arguments:
  -h, --help            show this help message and exit
  --version             show program's version number and exit
  -v {0,1,2,3}, --verbosity {0,1,2,3}
                        Verbosity level; 0=minimal output, 1=normal output,
                        2=verbose output, 3=very verbose output
  --settings SETTINGS   The Python path to a settings module, e.g.
                        "myproject.settings.main". If this isn't provided, the
                        DJANGO_SETTINGS_MODULE environment variable will be
                        used.
  --pythonpath PYTHONPATH
                        A directory to add to the Python path, e.g.
                        "/home/djangoprojects/myproject".
  --traceback           Raise on CommandError exceptions
  --no-color            Don't colorize the command output.
```

## 添加 django-crontab 到 INSTALLED_APPS

```
INSTALLED_APPS = [
    ...
    'jira',
    'django_crontab',
]
```
至此，基本工作完成。

## 自定义函数

接下来，编写要定时执行的业务函数。

在某一个 APP 中（与 views.py 同级）创建一个 cron.py 文件，对定时任务集中管理。当然，这不是必须的，只是建议。

在 cron.py 中编写函数：
```
...
def daily_statistics():
    ...
    # 筛选符合条件的 issues
    issues = get_jira_issues(jql_str)

    for issue in issues:
        ...

    # 入库
    ....
```

## 配置定时任务

好了，定时业务编写完成，接下来要配置定时任务。

在 settings.py 最后添加：
```
CRONJOBS = [
    ('0 7 * * *', 'jira_workload.cron.daily_statistics','>>/tmp/jira_daily_statistics.log'), # 每天 7 点开始统计昨天发布的 JIRA 任务
]
```
多个定时任务，都可以以元组的方式定义在 CRONJOBS 中。

聪明的你肯定也注意到了，没错，定义时间的方式和 Linux 中的 crontab 是一模一样的！

元组的第一个元素是时间；第二个元素是业务函数的路径，以 appname.cron.func 的形式描述；第三个元素是额外参数，可以定义函数的输出结果文件，这可以用作做日志，方便追踪。

当然，这只是其中的一种表示方式。

### 激活定时任务

执行下面的命令，添加并激活定时任务：
```
python manage.py crontab add
```

可以使用 `python manage.py show` 和 `crontab -l` 来检查，然后使用 `python manage.py remove` 来删除定时任务。

---

最后，django-crontab 模块并不适用于 Windows 平台，毕竟是对 Linux 底层的调用。

更多信息请参考：[https://github.com/kraiz/django-crontab](https://github.com/kraiz/django-crontab)












