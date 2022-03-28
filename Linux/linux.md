# 命令

#### /etc/rc.d/rc.local

并非所有的发行版都使用了rc.local，通常它是一个留给用户修改的shell脚本。一般会在init进程结束的时候运行它，所以你可以在这里放一些想要运行的额外脚本，而不用再创建自己的init脚本。

## ps -ef | grep activemq | grep -v grep

grep -v ：排除掉含有grep关键字的信息





### 编写脚本自动部署打包

* 需要安装putty，winscp

----------------------bat脚本

```bash
@echo off
set project_path=D:\todo\todo-timing
set local_file=%project_path%\target\task-service.jar
set deploy-shell=C:\Users\liyang\Desktop\autoDevelop.txt

set user=root
set passwd=q123456.
set ip=172.16.53.193

set server_path=/

echo ---------------------------------------------- execute mvn clean install
D:
cd %project_path%
call mvn clean install -D maven.test.skip=true

echo ---------------------------------------------- upload war file to server
call pscp -l %user% -pw %passwd% -r %local_file% %ip%:%server_path%

putty -ssh %user%@%ip% -pw %passwd% -m %deploy-shell%

pause
```

----------------------------------shell脚本

```shell
source /etc/profile
\#!/bin/sh

\# NAME = $1
\# echo NAME
\# awk '{print $2'} ----- read the file by oneline and oneline，use space as delimiters，print the second word
ID=`ps aux|grep task-service.jar|grep -v "$0"|grep -v "grep"|awk '{print $2'}`
\# print the assign process ID
echo $ID
for id in $ID
do
kill -9 $id
done

cd /home/PSRMDEV/pdf

rm -rf /home/PSRMDEV/pdf/logs /home/PSRMDEV/pdf/nohup.out /home/PSRMDEV/pdf/task-service.jar

mv /task-service.jar /home/PSRMDEV/pdf/

nohup java -jar -Djavax.xml.parsers.SAXParserFactory=[com.sun.org](http://com.sun.org/).apache.xerces.internal.jaxp.SAXParserFactoryImpl -Djavax.xml.parsers.DocumentBuilderFactory=[com.sun.org](http://com.sun.org/).apache.xerces.internal.jaxp.DocumentBuilderFactoryImpl -Djavax.xml.transform.TransformerFactory=[com.sun.org](http://com.sun.org/).apache.xalan.internal.xsltc.trax.TransformerFactoryImpl task-service.jar &

echo ==================
```
