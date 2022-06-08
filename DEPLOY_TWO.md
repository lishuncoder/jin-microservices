# docker-composer安装

docker-compose

```bash  
 1.vi /etc/sysctl.conf
    #末尾添加一行
    vm.max_map_count=262144
    #查看结果
    sysctl -p
 2.修改docker-composer.services.hyperf.volumes目录映射地址
 3.docker-compose up
 4.docker exec -it elk bin/bash
 5.cd /etc/logstash/conf.d
 6.vim logstash.conf
		  input {
			  beats {
				port => 5044
				codec => plain { charset => "UTF-8" }
			  }
		  }
			# 格式化日志
		  filter {
			  grok {
				  match => [ "message","\[%{TIMESTAMP_ISO8601:logtime}\] %{WORD:env}\.(?<level>[A-Z]{4,5})\: %{GREEDYDATA:msg}}" ]
				  }
		   }
		   output {
				elasticsearch {
					 action => "index"
					 hosts => ["localhost"]
					 index => "jm-log"
				 }
			}
 7.rm 02-beats-input.conf  10-syslog.conf  11-nginx.conf  30-output.conf
 8.docker restart elk
 9.连接mysql
 10.新建order-srv数据库
 11.导入/jin-microservices/order-srv.sql
 12.新建user-srv数据库
 13.导入/jin-microservices/user-srv.sql
 14.访问http://127.0.0.1:8848/nacos/#/login
 15.用户名密码: nacos/nacos
 16.命名空间->新建命名空间->增加空间 `dev`
 17.配置管理->配置列表->dev->导入配置->/jin-microservices/nacos_config.zip
```

Filebeat
 ```bash 
 1.wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-7.13.3-linux-x86_64.tar.gz
 2.tar -xvf filebeat-7.13.3-linux-x86_64.tar.gz
 3.vi filebeat.yml
    filebeat.inputs:
      - type: log
        paths:
          - /jin-microservices/*/runtime/logs/*.log
        multiline.pattern: '^\[[0-9]{4}-[0-9]{2}-[0-9]{2}'
        multiline.negate: true
        multiline.match: after
        multiline.timeout: 5s
        scan_frequency: 5s
    output:
      # 输出到logstash中,logstash更换为自己的ip
      logstash:
        hosts: ["127.0.0.1:5044"]
 4. ./filebeat-7.13.3-linux-x86_64/filebeat -e -c filebeat.yml 
 ```

验证
```bash
 1.访问http://127.0.0.1:8848/nacos/#/login 用户名密码: nacos/nacos
 2.访问http://39.108.236.73:9200
 3.访问http://39.108.236.73:5601/app/kibana
 4.访问http://127.0.0.1:9411/zipkin/ 
 5.访问http://127.0.0.1:36789
```

启动服务
```bash
1.docker exec -it hyperf /bin/bash
2.cd /data/project/
3.git clone https://github.com/Double-Jin/jin-microservices.git
4.cd jin-microservices/api-gateway/
    composer update
    复制.env.example为.env配置
    php bin/hyperf.php start
5.cd jin-microservices/user_srv/
    composer update
    复制.env.example为.env配置
    php bin/hyperf.php start
6.cd jin-microservices/order_srv/
    composer update
    复制.env.example为.env配置
    php bin/hyperf.php start
```
