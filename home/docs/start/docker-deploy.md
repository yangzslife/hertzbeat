---
id: docker-deploy  
title: Install HertzBeat via Docker   
sidebar_label: Install via Docker      
---

> Recommend to use docker deploy HertzBeat


1. Download and install the Docker environment   
   Docker tools download refer to [Docker official document](https://docs.docker.com/get-docker/)。
   After the installation you can check if the Docker version normally output at the terminal.
   ```
   $ docker -v
   Docker version 20.10.12, build e91ed57
   ```

2. pull HertzBeat Docker mirror  
   you can look up the mirror version TAG in [official mirror repository](https://hub.docker.com/r/tancloud/hertzbeat/tags)     
   ``` 
   $ docker pull tancloud/hertzbeat   
   ```

3. Mounted HertzBeat configuration file (optional)    
   Create `application.yml` in the host directory, eg:`/opt/application.yml`    
   For the complete content of the configuration file, see the project repository [/script/application.yml](https://github.com/dromara/hertzbeat/raw/master/script/application.yml).    
   You can modify the configuration file according to your needs.      
   - If you need to use email to send alarms, you need to replace the email server parameters `spring.mail` in `application.yml`   
   - **Recommended** If you need to use an external Mysql database to replace the built-in H2 database, you need to replace the `spring.datasource` parameter in `application.yml` For specific steps, see [Using Mysql to replace H2 database](mysql-change)  
   - **Recommended** If you need to use the time series database TDengine to store indicator data, you need to replace the `warehouse.store.td-engine` parameter in `application.yml` for specific steps, see [Using TDengine to store metrics data](tdengine-init)   
   - **Recommended** If you need to use the time series database IotDB to store the indicator database, you need to replace the `warehouse.storeiot-db` parameter in `application.yml` For specific steps, see [Use IotDB to store metrics data](iotdb-init)   

4. Mounted the account file(optional)           
   HertzBeat default built-in three user accounts, respectively `admin/hertzbeat tom/hertzbeat guest/hertzbeat`       
   If you need add, delete or modify account or password, configure `sureness.yml`. Ignore this step without this demand.    
   Create `sureness.yml` in the host directory，eg:`/opt/sureness.yml`    
   The configuration file content refer to project repository [/script/sureness.yml](https://github.com/dromara/hertzbeat/blob/master/script/sureness.yml)
   For detail steps, please refer to [Configure Account Password](account-modify)    

5. Start the HertzBeat Docker container    

```shell 
$ docker run -d -p 1157:1157 \
    -e LANG=zh_CN.UTF-8 \
    -e TZ=Asia/Shanghai \
    -v /opt/data:/opt/hertzbeat/data \
    -v /opt/logs:/opt/hertzbeat/logs \
    -v /opt/application.yml:/opt/hertzbeat/config/application.yml \
    -v /opt/sureness.yml:/opt/hertzbeat/config/sureness.yml \
    --name hertzbeat tancloud/hertzbeat
```

   This command starts a running HertzBeat Docker container with mapping port 1157. If existing processes on the host use the port, please modify host mapped port.  
   - `docker run -d` : Run a container in the background via Docker
   - `-p 1157:1157`  : Mapping container ports to the host
   - `-e LANG=zh_CN.UTF-8`  : (optional) set the LANG  
   - `-e TZ=Asia/Shanghai` : (optional) set the TimeZone  
   - `-v /opt/data:/opt/hertzbeat/data` : (optional, data persistence) Important⚠️ Mount the H2 database file to the local host, to ensure that the data is not lost due creating or deleting container.  
   - `-v /opt/logs:/opt/hertzbeat/logs` : (optional, if you don't have a need, just delete it) Mount the log file to the local host, to ensure the log will not be lost due creating or deleting container.
   - `-v /opt/application.yml:/opt/hertzbeat/config/application.yml`  : (optional, if you don't have a need, just delete it) Mount the local configuration file into the container which has been modified in the previous step, namely using the local configuration file to cover container configuration file.    
   - `-v /opt/sureness.yml:/opt/hertzbeat/config/sureness.yml`  : (optional, if you don't have a need, just delete it) Mount account configuration file modified in the previous step into the container. Delete this command parameters if no needs.  
   - `--name hertzbeat` : Naming container name hertzbeat 
   - `tancloud/hertzbeat` : Use the pulled latest HertzBeat official application mirror to start the container. Version can be looked up in [official mirror repository](https://hub.docker.com/r/tancloud/hertzbeat/tags)   

6. Begin to explore HertzBeat  

   Access http://ip:1157/ using browser. You can explore HertzBeat with default account `admin/hertzbeat` now!     

**HAVE FUN**   

### FAQ  

**The most common problem is network problems, please check in advance**

1. **MYSQL, TDENGINE, IoTDB and HertzBeat are deployed on the same host by Docker,HertzBeat use localhost or 127.0.0.1 connect to the database but fail**     
The problems lies in Docker container failed to visit and connect localhost port. Beacuse the docker default network mode is Bridge mode which can't access loacl machine through localhost.
> Solution A：Configure application.yml. Change database connection address from localhost to external IP of the host machine.     
> Solution B：Use the Host network mode to start Docker, namely making Docker container and hosting share network. `docker run -d --network host .....`   

2. **According to the process deploy，visit http://ip:1157/ no interface**   
Please refer to the following points to troubleshoot issues：  
> 1：If you switch to dependency service MYSQL database，check whether the database is created and started successfully.
> 2：Check whether dependent services, IP account and password configuration is correct in HertzBeat's configuration file `application.yml`.
> 3：`docker logs hertzbeat` Check whether the container log has errors. If you haven't solved the issue, report it to the communication group or community.

3. **Log an error TDengine connection or insert SQL failed**  
> 1：Check whether database account and password configured is correct, the database is created.   
> 2：If you install TDengine2.3+ version, you must execute `systemctl start taosadapter` to start adapter in addition to start the server.  

4. **Historical monitoring charts have been missing data for a long time**  
> 1：Check whether you configure Tdengine or IoTDB. No configuration means no historical chart data.  
> 2：Check whether Tdengine database `hertzbeat` is created. 
> 3: Check whether IP account and password configuration is correct in HertzBeat's configuration file `application.yml`.

5. If the history chart on the monitoring page is not displayed，popup [please configure time series database]
> As shown in the popup window，the premise of history chart display is that you need install and configure hertzbeat's dependency service - IoTDB or TDengine database.
> Installation and initialization this database refer to [TDengine Installation](tdengine-init) or [IoTDB Installation](iotdb-init)  

6. The historical picture of monitoring details is not displayed or has no data, and TDengine has been deployed  
> Please confirm whether the installed TDengine version is near 2.4.0.12, version 3.0 and 2.2 are not compatible.  

7. The time series database is installed and configured, but the page still displays a pop-up [Unable to provide historical chart data, please configure dependent time series database]
> Please check if the configuration parameters are correct
> Is iot-db or td-engine enable set to true
> Note⚠️If both hertzbeat and IotDB, TDengine are started under the same host for docker containers, 127.0.0.1 cannot be used for communication between containers by default, and the host IP is changed
> You can check the startup logs according to the logs directory
