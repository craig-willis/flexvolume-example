# FlexVolume example

This is an example of a WebDav flexvolume driver based on the 
[NFS example](https://github.com/kubernetes/examples/tree/master/staging/volumes/flexvolume)
and @hategan's [CSI Driver](https://github.com/whole-tale/wt-kubernetes/tree/master/csi-test)

Highlighs:
* WebDav flexvolume driver is a simple bash script (minimal changes from nfs example)
* Uses anonymous WebDav, but could easily be extended to pass credentials
* Requires `davfs` and `jq` to be installed on the host
* Requires WebDav service IP to be used, since the host is doing the mount and can't access the Kubernetes DNS

Issues:
* Via Flexvolume, Kubernetes manages the volume mount/unmount lifecycle, but it doesn't solve the problem of needing install dependencies on the host. If we do this with GirderFS, it means ensuring that the GirderFS version on the host is kept in sync with what's needed by the system.  If GirderFS were, say, a `go` binary this would be a bit easier.

## Setup via Kubeadm Bootstrap

The `kubeadm-bootstrap` repo is used as the basis for some OpenStack Kubernetes deployments.  
For development, we setup a single Ubuntu VM via `kubeadm` that is closer to what
we would use on OpenStack than Minikube or GKE.

First, provision an Ubuntu VM (Jetstream, Chameleon, Nebula...)

Next, peploy Kubernetes (single node):
```
git clone https://github.com/nds-org/kubeadm-bootstrap -b 1.12-update
cd kubeadm-bootstrap
sudo ./install-kubeadm.bash
sudo ./init-master.bash
```
This gives you v1.12 Kubernetes on a single node Ubuntu VM. 

## Install host dependencies

Install `davfs2`, `jq`:
```
sudo apt-get install -y davfs2 jq
```

## Install the flexvolume example
Create the Flexvolume demonset:
```
kubectl create -f driver/ds.yaml
```

Create the WebDav server:
```
kubectl create -f webdav-server/webdav-server.yaml
```

Get the service IP. We need the IP for the host to reach the service:
```
kubectl get ep webdav-service
```

Edit the `client/nginx-dav.yaml`, setting the endpoint IP/port
```
  volumes:
  - name: test
    flexVolume:
      driver: "wholetale/webdav"
      fsType: "webdav"
      options:
        server: "http://<ip>:8080"
```


Create the client:
```
kubectl create -f client/nginx-dav.yaml
```

Confirm that the volume mounts (`kubectl describe pod podname`) and that you can write to it:
```
kubectl exec -it podnash bash
$ touch /data/x
```

Mount locally and confirm that you see data:
```
sudo mkdir /mnt/dav
sudo mount -t davfs  http://<ip>:8080 /mnt/dav
ls /mnt/dav
```
