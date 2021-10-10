---
title: hdp superset upgrade
toc: true
date: 2021-07-13 17:25:34
tags:
  - big data
---

# Mac 安装 superset

```sh
# 创建 python 虚拟环境，在虚拟环境中安装 superset
python3 -m venv venv
# 启动虚拟环境
. venv/bin/activate

# 更新 setuptools, pip
pip install --upgrade setuptools pip

# Install superset
pip install apache-superset

# Initialize the database
superset db upgrade

# Create an admin user (you will be prompted to set a username, first and last name before setting a password)
$ export FLASK_APP=superset
superset fab create-admin

# Load some data to play with
superset load_examples

# Create default roles and permissions
superset init

# To start a development web server on port 8088, use -p to bind to another port
superset run -p 8088 --with-threads --reload --debugger


# production start
./venv/bin/gunicorn -D --workers 4 -p ./run/superset/superset.pid --log-file ./log/superset/superset.log -t 300 -b 0.0.0.0:9088 --limit-request-line 0 --limit-request-field_size 0 "superset.app:create_app()"


gunicorn \
      -w 2 \
      -k gevent \
      --timeout 120 \
      -b  0.0.0.0:8088 \
      --limit-request-line 0 \
      --limit-request-field_size 0 \
      --statsd-host localhost:8125 \
      "superset.app:create_app()"
```

# Ambari customerized service - superset

[ambari customerized service explained](https://programmer.group/ambari-custom-service-1-introduction.html)

[official doc](https://cwiki.apache.org/confluence/display/AMBARI/Custom+Services)

[ambari resource_management Script](https://github.com/apache/ambari/blob/24dbed27b97f4c82835273758f0bceb3334b4ef2/ambari-common/src/main/python/resource_management/libraries/script/script.py)

```
```

