---
title: airflow
toc: true
tags:
  - schedule
  - biodata
date: 2020-12-18 15:33:14
---


# install

[quickstart](https://airflow.apache.org/docs/apache-airflow/stable/start.html)

> Airflow is published as `apache-airflow` package in PyPI. Installing it however might be sometimes tricky because Airflow is a bit of both a library and application. Libraries usually keep their dependencies open and applications usually pin them, but we should do neither and both at the same time. We decided to keep our dependencies as open as possible (in `setup.cfg` and `setup.py`) so users can install different version of libraries if needed. This means that from time to time plain `pip install apache-airflow` will not work or will produce unusable Airflow installation.
>
> In order to have repeatable installation, however, starting from **Airflow 1.10.10** and updated in **Airflow 1.10.13** we also keep a set of "known-to-be-working" constraint files in the `constraints-master` and `constraints-1-10` orphan branches. Those "known-to-be-working" constraints are per major/minor python version. You can use them as constraint files when installing Airflow from PyPI. Note that you have to specify correct Airflow version and python versions in the URL.

```sh
pip3 install --use-deprecated legacy-resolver "apache-airflow==1.10.14" --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-1.10.14/constraints-3.8.txt" 

pip3 install "apache-airflow==1.10.14" --constraint "https://raw.githubusercontent.com/apache/airflow/constraints-1.10.14/constraints-3.8.txt" 
```

> On November 2020, new version of PIP (20.3) has been released with a new, 2020 resolver. This resolver does not yet work with Apache Airflow and might leads to errors in installation - depends on your choice of extras. In order to install Airflow you need to either downgrade pip to version 20.2.4 `pip upgrade --pip==20.2.4` or, in case you use Pip 20.3, you need to add option `--use-deprecated legacy-resolver` to your pip install command.

```sh
# airflow needs a home, ~/airflow is the default,
# but you can lay foundation somewhere else if you prefer
# (optional)
export AIRFLOW_HOME=~/airflow

# install from pypi using pip
pip install apache-airflow

# initialize the database
airflow db init

airflow users create \
    --username admin \
    --firstname petrina \
    --lastname zheng \
    --role Admin \
    --email spiderman@superhero.org

# start the web server, default port is 8080
airflow webserver --port 8080

# start the scheduler
# open a new terminal or else run webserver with ``-D`` option to run it as a daemon
airflow scheduler

# visit localhost:8080 in the browser and use the admin account you just
# created to login. Enable the example_bash_operator dag in the home page
```