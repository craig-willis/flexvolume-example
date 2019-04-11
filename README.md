# FlexVolume example


Provision an Ubuntu VM (Jetstream, Chameleon, Nebula...)

Insetall davfs2, jq
`sudo apt-get install davfs2 jq`

Deploy Kubernetes (single node) via https://github.com/nds-org/kubeadm-bootstrap 

Create the Flexvolume demonset:
`kubectl create -f driver/ds.yaml`

Create the WebDav server:
`kubectl create -f webdav-server/webdav-server.yaml` 

Get the service IP:
`kubectl get ep webdav-service`

Edit the `client/nginx-dav.yaml`, setting the endpoint IP/port

Create the client:
`kubectl create -f client/nginx-dav.yaml`

Confirm that the volume mounts (`kubectl describe pod podname`) and that you can write to it:
```
kubectl exec -it podnash bash
$ touch /data/x
```
