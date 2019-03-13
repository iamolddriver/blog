# 修改Harbor默认80端口
系统版本：Ubuntu 16.04  

1、转至harbor目录下：

```Shell
cd /opt/harbor
```

2、修改```harbor.cfg```文件hostname：

```Shell
## Configuration file of Harbor

#This attribute is for migrator to detect the version of the .cfg file, DO NOT MODIFY!
_version = 1.6.0
#The IP address or hostname to access admin UI and registry service.
#DO NOT use localhost or 127.0.0.1, because Harbor needs to be accessed by external clients.
hostname = 192.168.10.10:8081

#The protocol for accessing the UI and token/notification service, by default it is http.
#It can be set to https if ssl is enabled on nginx.
ui_url_protocol = http
```
3、修改```docker-compose.yml```文件映射为8081端口：

```Shell

    networks:
      - harbor
    ports:
      - 8081:80
      - 443:443
      - 4443:4443
    depends_on:
      - postgresql
      - registry
      - ui
      - log
```
4、修改```common/templates/registry/config.yml```文件加入8081端口(**此步修改后会导致public_url:8081:8081 error, 所以并不需要修改**)：

```Shell
auth:
  token:
    issuer: harbor-token-issuer
    realm: $public_url:8081/service/token # 8081:8081 error
    rootcertbundle: /etc/registry/root.crt
    service: harbor-registry
```
5、修改Docker的守护进程文件```/etc/docker/daemon.json```，设置信任的主机与端口：

```Shell
{
    "insecure-registries":["192.168.10.10:8081"],
    "default-runtime": "nvidia",
    "runtimes": {
        "nvidia": {
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    }
}
```
6、重新启动docker：

```Shell
systemctl daemon-reload
systemctl restart docker.service
```
7、停止harbor，重新启动并生成配置文件：

```Shell
docker-compose stop
./install.sh
```
8、最后，测试验证：：

```Shell
# docker login 192.168.10.10:8081
Username: haha123
Password: 
Login Succeeded
```
- Note: 主节点配置Docker和Harbor, 其他节点同样需要添加```insecure-registries```
