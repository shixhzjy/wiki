### 系统环境
rocketmq-4.2.0</br>
Mac OS

### 解决启动NameServer时报缺少JAVA_HOME异常
在练习Rocket mq启动NameServer时，报报`ERROR: Please set the JAVA_HOME variable in your environment, We need java(x64)! !!`异常。</br>
查资料，该异常信息是在runserver.sh抛出的，看runserver.sh脚本如下：</br>
````shell
#!/bin/sh

error_exit ()
{
    echo "ERROR: $1 !!"
    exit 1
}

[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=$HOME/jdk/java
[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=/usr/java
[ ! -e "$JAVA_HOME/bin/java" ] && error_exit "Please set the JAVA_HOME variable in your environment, We need java(x64)!"

export JAVA_HOME
export JAVA="$JAVA_HOME/bin/java"
export BASE_DIR=$(dirname $0)/..
export CLASSPATH=.:${BASE_DIR}/conf:${CLASSPATH}

#===========================================================================================
# JVM Configuration
#===========================================================================================
JAVA_OPT="${JAVA_OPT} -server -Xms4g -Xmx4g -Xmn2g -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=320m"
JAVA_OPT="${JAVA_OPT} -XX:+UseConcMarkSweepGC -XX:+UseCMSCompactAtFullCollection -XX:CMSInitiatingOccupancyFraction=70 -XX:+CMSParallelRemarkEnabled -XX:SoftRefLRUPolicyMSPerMB=0 -XX:+CMSClassUnloadingEnabled -XX:SurvivorRatio=8  -XX:-UseParNewGC"
JAVA_OPT="${JAVA_OPT} -verbose:gc -Xloggc:/dev/shm/rmq_srv_gc.log -XX:+PrintGCDetails"
JAVA_OPT="${JAVA_OPT} -XX:-OmitStackTraceInFastThrow"
JAVA_OPT="${JAVA_OPT}  -XX:-UseLargePages"
JAVA_OPT="${JAVA_OPT} -Djava.ext.dirs=${JAVA_HOME}/jre/lib/ext:${BASE_DIR}/lib"
#JAVA_OPT="${JAVA_OPT} -Xdebug -Xrunjdwp:transport=dt_socket,address=9555,server=y,suspend=n"
JAVA_OPT="${JAVA_OPT} ${JAVA_OPT_EXT}"
JAVA_OPT="${JAVA_OPT} -cp ${CLASSPATH}"

$JAVA ${JAVA_OPT} $@
````
脚本中`[ ! -e "$JAVA_HOME/bin/java" ] && error_exit "Please set the JAVA_HOME variable in your environment, We need java(x64)!"`抛出的信息。</br>

`[ ! -e "$JAVA_HOME/bin/java" ]`是判断在$JAVA_HOME路径下是否有`bin/java`。若没有，则抛异常。</br>

根据异常信息知道缺少JAVA_HOME环境变量，在.bash_profile添加JAVA_HOME变量，如下：</br>
````shell
JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_121.jdk/Contents/Home
````
再打开一个shell窗口启动nameserver，还是报同样的异常，说明还是没找到`bin/java`。可是已经在`.bash_profile`中添加`JAVA_HOME`变量了，
难道`runserver.sh`脚本里就没读到`$JAVA_HOME`呢？
在`runserver.sh`脚本里打印一下`$JAVA_HOME`的值，部分脚本如下：</br>
````shell
echo $JAVA_HOME

[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=$HOME/jdk/java
[ ! -e "$JAVA_HOME/bin/java" ] && JAVA_HOME=/usr/java
[ ! -e "$JAVA_HOME/bin/java" ] && error_exit "Please set the JAVA_HOME variable in your environment, We need java(x64)!"
````
再启动nameserver，发现$JAVA_HOME是空值，验证了自己的猜想。
继续找资料，需要用`export`将JAVA_HOME设置成系统变量，然后其他shell窗口或脚本才可以直接用`$JAVA_HOME`。修改`.bash_profile`如下：</br>
````shell
JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_121.jdk/Contents/Home
export JAVA_HOME
````


参考：</br>
[消息队列学习 一 ------ rocketmq启动nameserver异常解决](https://blog.csdn.net/mingtian625/article/details/49307189)</br>
[linux中shell脚本设置环境变量](http://blog.sina.com.cn/s/blog_623630d50102vdyk.html)</br>
[Linux中profile、bashrc、bash_profile之间的区别和联系](https://blog.csdn.net/chenchong08/article/details/7833242)
