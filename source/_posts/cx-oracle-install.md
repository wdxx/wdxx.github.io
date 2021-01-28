---
title: 脱机安装cx_oracle
date: 2020-12-15 22:10:38
tags: [python,oracle]
categories: [python,oracle]
---
当服务器没有root权限的时候，安装某些Python模块就不方便了，该文章记录下，当没有root权限时，本地安装Python第三方模块的临时办法，以`cx_Oracle`模块进行说明。
###准备工作
在安装`cx_Oracle`之前，服务器是需要安装`basic packages`与`Development and Runtime packages`，下载网址为：https://www.oracle.com/database/technologies/instant-client/linux-x86-64-downloads.html。
当前本地环境为`python 2.7`，选择了如下的包：
instantclient-basic-linux.x64-11.2.0.4.0.zip
instantclient-sdk-linux.x64-11.2.0.4.0.zip
<!-- more -->

### 安装oracle instantclient
将上一步骤的zip包解压。
```bash
mkdir -p ~/tools/oracle
cp instantclient-basic-linux.x64-11.2.0.4.0.zip ~/tools/oracle
cp instantclient-sdk-linux.x64-11.2.0.4.0.zip ~/tools/oracle
cd ~/tools/oracle
unzip instantclient-sdk-linux.x64-11.2.0.4.0.zip
unzip instantclient-basic-linux.x64-11.2.0.4.0.zip
cd instantclient_11_2
cp -R sdk/* ./
cp -R ./sdk/include/* .
ln -s libclntsh.so.12.1 libclntsh.so
ln -s libocci.so.12.1 libocci.so
export ORACLE_HOME=~/tools/oralce/instantclient_11_2/
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$ORACLE_HOME
```

###编译安装cx_Oracle
先需要下载cx_Oracle的源码，从[github网站](https://github.com/oracle/python-cx_Oracle),可以下载到。由于我使用的`python 2.7`根据介绍，使用[5.x的版本](https://github.com/oracle/python-cx_Oracle/archive/5.3.zip)。
下载后安装步骤如下:
```bash
unzip python-cx_Oracle-5.3.zip
cd python-cx_Oracle-5.3
python setup.py build
mkdir install
export PYTHONPATH=$PYTHONPATH:~/tools/oralce/python-cx_Oracle-5.3/install
python setup.py install --prefix=~/tools/oralce/python-cx_Oracle-5.3/install
## 注意上个步骤会报错，执行下面的步骤继续
export PYTHONPATH=$PYTHONPATH:~/tools/oralce/python-cx_Oracle-5.3/installlib/python2.7/site-packages/
python setup.py install --prefix=~/tools/oralce/python-cx_Oracle-5.3/install
```
按照上面的步骤就可以安装成功了。

###cx_Oracle使用
因为当前不是在系统路径安装的，所以重新开一个窗口会导致`import cx_Oracle`失败，需要编写一个脚本来进行环境变量的设置，然后加入到`.bashrc`中也是可以的。
下面是设置环境变量的脚本(`envsetup.sh`):
```bash
export ORACLE_HOME=~/tools/oralce/instantclient_11_2/
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$ORACLE_HOME
export PYTHONPATH=$PYTHONPATH:~/tools/oralce/python-cx_Oracle-5.3/installlib/python2.7/site-packages/
```
上面的步骤执行完后，就可以在Python中导入`cx_Oracle`模块了。

###总结
其实该方法可以推广到其他模块的安装，当你需要在本地临时验证时。

### 参考
1. https://www.cnblogs.com/doctormo/p/12059738.html
2. https://blog.csdn.net/yuan_lo/article/details/48289317
3. https://stackoverflow.com/questions/25885467/cx-oracle-pip-install-fails-oci-h-no-such-file-or-directory