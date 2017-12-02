## Docker上一些内容记录

**集群中查看具体错误信息**

&emsp;通过 `docker stack ps ing --no-trunc` 命令查看到具体的错误信息是"starting container failed: Address already in use"，

**重启Docker 服务**

&emsp;通过 `sudo service docker restart` 命令

**开发集群故障时重启服务器的办法**

```shell
sudo ssh root@dev-server-swarm
reboot
sudo reboot

```

docker service logs -t  - - raw  service_name 

