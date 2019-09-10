# install docker

```
sudo apt install docker.io

docker version

sudo usermod -a -G docker $USER

sudo reboot
```

# mantipulate docker

### show images
```docker images```

### show images' size
```docker system df```

### run docker
```docker run -p 8080:8080 tomcat```

```docker run -p 8080:8080 -d tomcat``` 

### delete docker
```docker rmi```

### show running docker
```docker ps```

### show all docker
```docker ps -a```

# tip

VMWare Fusion 改了network之后,虚拟机连不上
```
'sudo /Applications/VMware\ Fusion.app/Contents/Library/vmnet-cli --stop' and
'sudo /Applications/VMware\ Fusion.app/Contents/Library/vmnet-cli --start'
```