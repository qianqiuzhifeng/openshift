# dockr

## docke指令

```shell
#停止/启动/重启/删除所有已停止容器
docker stop/start/restart/rm $(docker ps -a -q)
#删除所有已停止容器
docker rm $(docker ps -q -f status=exited)
#基于docker推送镜像
sudo ./oc import-image redis 172.30.1.1:5000/demo/redis --confirm
```
