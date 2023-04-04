---
title: python subprocess
toc: true
date: 2018-09-03 14:06:24
tags:
    - python
---

Synchronous vs multiprocessing vs multithreading vs async

Concurrency vs Parralism.

asyncio & threading can run multiple I/O operations at the same time.

Async runs one block of code at a time while threading just one line of code at a time. With async, we have better control of when the execution is given to other block of code but we have to release the execution ourselves.

- **IO bound problems**: use async if your libraries support it and if not, use threading.
- **CPU bound problems**: use multi-processing.
- **None above is a problem:** you are probably just fine with synchronous code. You may still want to use async to have the feeling of responsiveness in case your code interacts with a user.

Reference: 

* [blog: subprocess](http://www.cnblogs.com/vamei/archive/2012/09/23/2698014.html)

subprocess.call()
父进程等待子进程完成
返回退出信息(returncode，相当于exit code，见[Linux进程基础](http://www.cnblogs.com/vamei/archive/2012/09/20/2694466.html))

subprocess.check_call()

父进程等待子进程完成

返回0

检查退出信息，如果returncode不为0，则举出错误subprocess.CalledProcessError，该对象包含有returncode属性，可用try...except...来检查(见[Python错误处理](http://www.cnblogs.com/vamei/archive/2012/07/10/2582787.html))。

subprocess.check_output()

父进程等待子进程完成

返回子进程向标准输出的输出结果

检查退出信息，如果returncode不为0，则举出错误subprocess.CalledProcessError，该对象包含有returncode属性和output属性，output属性为标准输出的输出结果，可用try...except...来检查。

* [official doc: subprocess](https://docs.python.org/3.7/library/subprocess.html)

```py
subprocess.run(args, *, stdin=None, input=None, stdout=None, stderr=None, capture_output=False, shell=False, cwd=None, timeout=None, check=False, encoding=None, errors=None, text=None, env=None)
```

* [config env in subprojcess](https://docs.python.org/3.7/library/subprocess.html)

```pyt
new_env = os.environ.copy()
new_env['MEGAVARIABLE'] = 'MEGAVALUE'
subprocess.Popen('path', env=new_env)
```

an example to use subprojcess:

```python
try:
                subprocess.check_output(['java',
                                         '-DCOMMAND=' + command,
                                         '-DTENANT_ID=' + tenant_id,
                                         '-DMODEL_ID=' + model_id,
                                         '-jar', 'xxx.jar',
                                         ])
            except subprocess.CalledProcessError as e:
                db_util.reopx_solver - 1.0 - SNAPSHOT - all.jarport_status(tenant_id, model_id=model_id,
                                                                           msg=f'Failed runner for "{command}"...')
                logger.exception('Could not start solver jar: ', e.output)
                raise OPXException(f'Could not start runner for "{command}".', e.stderr)
```

> the jar must be in the same directory as parent process dir
