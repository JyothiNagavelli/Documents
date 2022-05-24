# k3s install cmds

## Curl the instaletion
```
curl -sfL https://get.k3s.io | sh -
```
### for specific version
```
curl -sfL https://get.k3s.io | INSTALL_K3S_VERSION=<version> sh -
```

## Export the config file
```
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
```

## Change the mode of the file 
```
sudo chmod 644 $KUBECONFIG
```
