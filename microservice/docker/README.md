# install docker

```
sudo apt install docker.io

docker version

sudo usermod -a -G docker $USER

sudo reboot
```


# tip

VMWare Fusion 改了network之后,虚拟机连不上
```
'sudo /Applications/VMware\ Fusion.app/Contents/Library/vmnet-cli --stop' and
'sudo /Applications/VMware\ Fusion.app/Contents/Library/vmnet-cli --start'
```