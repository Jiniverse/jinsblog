---
toc: true
layout: post
description: 总结下在超算上跑 jupyter notebook 的处理办法
categories: [server]
title: 如何在超算上跑 jupyter notebook
author: Shan Jin
comments: true
---

# 在超算上跑 jupyter notebook

## 在登录节点跑 jupyter notebook
先在本地配置好ssh登录信息，例如
```shell
Host example
    Hostname ssh.example.com
    User someone
    Port 22
```
然后用ssh连接远程服务器后者超算，并做本地的端口映射，例如
```shell
ssh example -L 8000:127.0.0.1:8890
```
其中 `8000` 对应的是本地端口，`8890` 是远程端口，在登录节点上运行
```shell
jupyter-notebook --no-browser --port=8890
```
注意端口要与ssh连接的端口保持一致。最后在本地浏览器里访问 `127.0.0.1:8000` 即可打开登录节点运行的 jupyter notebook.

{% include alert.html text="在登录节点跑 jupyter 一定程度上能节省计算资源，但仅适用于内存占用较少的程序，数据量比较大的时候应该用远程节点跑" %}

## 用远程节点跑 jupyter notebook
在远程服务器创建如下脚本（假设命名为 `jupyter.job` ）
```shell
#!/bin/bash
#SBATCH --nodes 1
#SBATCH --time 12:00:00
#SBATCH --job-name jupyter-notebook
#SBATCH -o /path/to/logfile/jupyter-output.log
#SBATCH -e /path/to/logfile/jupyter-error.log

# get tunneling info
XDG_RUNTIME_DIR=""
port=$(shuf -i8000-9999 -n1)
node=$(hostname -s)
user=$(whoami)

# print tunneling instructions jupyter-log
echo -e "
   MacOS or linux terminal command to create your ssh tunnel
   ssh example -N -L ${port}:${node}:${port} 

   Use a Browser on your local machine to go to:
   localhost:${port}  (prefix w/ https:// if using password)
   "

jupyter-notebook --no-browser --port=${port} --ip=${node}
```
将其中的 `/path/to/logfile/` 替换为你想要的路径，将 `example` 改为你定义的 Host. 用 `sbatch jupyter.job` 提交任务，而后在输出文件 `jupyter-output.log` 中复制ssh的连接信息，在本地终端中运行。最后在本地浏览器中访问 `127.0.0.1:${port}`，其中 `${port}` 是输出文件中相应的端口号，大功告成！