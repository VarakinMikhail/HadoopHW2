
Задание:
Развернуть YARN и опубликовать веб-интерфейсы основных и вспомогательных демонов кластера для внешнего использования.
(Прошлое задание выполнено и считается, что выполнение начинается после выполнение предыдущего задания, репозиторий с предыдущим заданием https://github.com/VarakinMikhail/HadoopHW1)

Данные:

узел для входа 176.109.81.242 

jn 192.168.1.106 

nn 192.168.1.107 

dn-00 192.168.1.109 

dn-01 192.168.1.108

Подключаемся к ноде:
```
ssh team@176.109.81.242
```

Переходим в папку с дистрибутивом hadoop
```
cd hadoop-3.4.0/etc/hadoop
```

И редактируем конфиг
```
vim yarn-site.xml
```
В configuration добавляем


```
<property>
	<name>yarn.nodemanager.aux-services</name>
	<value>mapreduce_shuffle</value>
  </property>
  <property>
	<name>yarn.nodemanager.env-whitelist</name>
	<value>JAVA_HOME, HADOOP_COMMON_HOME, HADOOP_HDFS_HOME, HADOOP_CONF_DIR, CLASSPATH_PREPEND_DISTCACHE, HADOOP_YARN_HOME, HADOOP_HOME, PATH, LANG, TZ, HADOOP_MAPRED_HOME</value>
  </property>
<property>
  <name>yarn.resourcemanager.hostname</name>
  <value>tmpl-nn</value>
</property>
<property>
  <name>yarn.resourcemanager.address</name>
  <value>tmpl-nn:8032</value>
</property>
<property>
  <name>yarn.resourcemanager.resource-tracker.address</name>
  <value>tmpl-nn:8031</value>
</property>
```

Потом редактируем следующий конфиг:
```
vim mapred-site.xml
```

Добавляя в configuration следующее:
```
<property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
</property>
<property>
    <name>mapreduce.application.classpath</name>
    <value>$HADOOP_HOME/share/hadoop/mapreduce/*:$HADOOP_HOME/share/hadoop/mapreduce/lib/*</value>
</property>
```
Теперь скопируем конфиги на остальные ноды:
```
scp mapred-site.xml tmpl-dn-00:/home/hadoop/hadoop-3.4.0/etc/hadoop
```
```
scp mapred-site.xml tmpl-dn-01:/home/hadoop/hadoop-3.4.0/etc/hadoop
```
```
scp mapred-site.xml tmpl-nn:/home/hadoop/hadoop-3.4.0/etc/hadoop
```
```
scp yarn-site.xml tmpl-dn-00:/home/hadoop/hadoop-3.4.0/etc/hadoop
```
```
scp yarn-site.xml tmpl-dn-01:/home/hadoop/hadoop-3.4.0/etc/hadoop
```
```
scp yarn-site.xml tmpl-nn:/home/hadoop/hadoop-3.4.0/etc/hadoop
```
Переходим на Name Node:
```
ssh tmpl-nn
```
И запустим YARN:
```
hadoop-3.4.0/sbin/start-yarn.sh
```
Можно проверить, что YARN запустился, с помощью jps

Теперь можно опубликовать веб-интерфейсы

Перейдем обратно на NameNode:
```
exit
```
Запустим сервис historyserver:
```
mapred --daemon start historyserver
```

Вернемся на jump node (team):
```
exit
```
```
exit
```
Создадим конфиг для NameNode:
```
sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/nn
```
```
sudo vim /etc/nginx/sites-available/nn
```
И настроим конфиг так: порт меняем на 
```
listen 9870;
```
комментируем строку listen [::]:80 default_server;
А location меняем на:
```
location / {
    auth_basic "Administrator's Area";
    auth_basic_user_file /etc/.htpasswd;
    # First attempt to serve request as file, then
    # as directory, then fall back to displaying a 404.
    # try_files $uri $uri/ =404;
    proxy_pass http://tmpl-nn:9870;
}
```

Сделаем отдельный конфиг для resource manager и для historyserver:
```
sudo cp /etc/nginx/sites-available/nn /etc/nginx/sites-available/ya
```
```
sudo cp /etc/nginx/sites-available/nn /etc/nginx/sites-available/dh
```
И отредактируем конфиги:
```
sudo vim /etc/nginx/sites-available/ya
```

порт меняем на 
```
listen 8088;
```
А location меняем на:
```
location / {
    auth_basic "Administrator's Area";
    auth_basic_user_file /etc/.htpasswd;
    # First attempt to serve request as file, then
    # as directory, then fall back to displaying a 404.
    # try_files $uri $uri/ =404;
    proxy_pass http://tmpl-nn:8088;
}
```
```
sudo vim /etc/nginx/sites-available/dh
```
порт меняем на 
```
listen 19888;
```
А location меняем на:
```
location / {
    auth_basic "Administrator's Area";
    auth_basic_user_file /etc/.htpasswd;
    # First attempt to serve request as file, then
    # as directory, then fall back to displaying a 404.
    # try_files $uri $uri/ =404;
    proxy_pass http://tmpl-nn:19888;
}
```
Сделаем конфигурации активными:
```
sudo ln -s /etc/nginx/sites-available/nn /etc/nginx/sites-enabled/nn
```
```
sudo ln -s /etc/nginx/sites-available/ya /etc/nginx/sites-enabled/ya
```
```
sudo ln -s /etc/nginx/sites-available/dh /etc/nginx/sites-enabled/dh
```
И дадим nginx посмотреть обновленные конфиги:
```
sudo systemctl reload nginx
```

Теперь можно подключиться, пробросив (помимо порта для hadoop) еще два порта для yarn и historyserver. Для этого сначала выйдем:
```
exit
```
А затем снова зайдем с помощью такой команды:
```
ssh -L 9870:127.0.0.1:9870 -L 8088:127.0.0.1:8088 -L 19888:127.0.0.1:19888 team@176.109.81.242
```
