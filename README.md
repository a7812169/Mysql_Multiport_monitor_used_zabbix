# 使用zabbix监控MYSQL多端口数据库服务

###为什么启动这个项目
上网找了监控mysql的plugin,觉得percona-monitoring-plugins写的特别的专业，可惜percona的仅仅支持一台机器一个mysql端口，而我们的机器一台机器有多个端口，于是产生了想法，结合zabbix的自动发现特性，让zabbix自动的发现服务器的端口，自动实现对mysql多端口的监控。
这样做的好处是<b style='color:red'>保留了percona-monitoring-plugins原生的特性，又新填上了一个原生percona-monitoring-plugins所没有的支持多端口的特性</b>
###说明
<b style='color:red'>已经在生产环境使用</b>   
1. zabbix监控机模版和被监控端脚本均根据percona-monitoring-plugins1.1.5-1版进行修改，因此被监控端安装条件同原版插件相同，比如，必须安装php,php-mysql模块   
2. mysql发现端口采用了网络端口监听的方法(当然也可以采用配置文件等静态的方法),这种方法的可取之处是动态发现，无需任何配置   
3. 对原插件(percona-monitoring-plugins)中的脚本get_mysql_stats_wrapper.sh进行了修改,以适应多端口数据采集   
4. 因一台主机上多个mysql使用了同一个监控脚本，因此，要求多个mysql配置相同的用户名和密码,这样也是为了更标准化的配置   

##拓展思考
因对mysql的监控不依赖于任何主机，只要有合适的权限配置，在任何一台主机上都可以监控所有设备的mysql服务。
如想实现这种需求，需要对"端口发现机制"脚本和"wrapper"脚本进行适当修改.


###配置方法
一.zabbix监控机导入模板
导入名为Mysql_Multiport.xml的模板,导入后,模版默认名为Percona MySQL Server Multiport Template。具体导入方法不再赘述

二.被监控端
1.下载文件
cd /oracle/lhx<br>
git clone https://github.com/lianghx7123/Mysql_Multiport_monitor_used_zabbix.git mysql_monitor<br>

2.复制脚本文件并执行权限
mkdir -p /var/lib/zabbix/percona/scripts/<br>
cp get_mysql_stats_wrapper.sh /var/lib/zabbix/percona/scripts/<br>
cp mysql_low_discovery.sh /var/lib/zabbix/percona/scripts/<br>
cp ss_get_mysql_stats.php /var/lib/zabbix/percona/scripts/<br>
chmod 755 /var/lib/zabbix/percona/scripts/*<br>

3.复制zabbix配置文件
cp userparameter_percona_mysql.conf /usr/local/zabbix/etc/zabbix_agentd.conf.d<br>
注意：请注意zabbix_agentd.conf文件的配置是否包含了以上路径<br>

4.重启zabbix agent

###配置注意事项：

1. 默认的mysql发现测了间隔为1小时.因此刚配置好，可能短时间内不能发现监听的端口，请耐心等待<br>
2. 关于被监控端脚本<br>
	脚本ss_get_mysql_stats.php中的用户名和密码根据实际情况进行修改<br>
		$mysql_user = 'MYSQL_USER';<br>
		$mysql_pass = 'MYSQL_PASSWORD';<br>
	脚本中get_mysql_stats_wrapper.sh包含了监控的地址，默认是127.0.0.1<br>
	因此,命令行之行 mysql -u'MYSQL_USER' -p'MYSQL_PASSWORD' -h127.0.0.1 -P'MYSQL_PORT' 必须成功。否则监控采集不到任何数据.


