# Kubernetes with live and in-place vertical scaling
----

For the original README.md, refer to README.k8s.md.

This is the repo for Kubernetes with live and in-place vertical scaling that is being developed in IBM Austin Research team.

The main working branch is `qos-master`, which is periodically synced with the upstream master branch by rebase.

----

## Getting started.

If you're familiar with k8s enough to know how to update k8s' binaries of an existing k8s cluster, build binaries and update the binaries of API server, scheduler, resource-controller, and kubelet on master/worker nodes of your cluster.

Otherwise, please follow the following steps to install a new k8s cluster with live and in-place vertical scaling.

1. Install docker, kubeadm, kubelet, and kubectl. 
[https://kubernetes.io/docs/setup/independent/install-kubeadm/](https://kubernetes.io/docs/setup/independent/install-kubeadm/)

2. Unless swap is disabled, add '--fail-swap-on=false’ to kubelet’s config file located at /etc/systemd/system/kubelet.service.d/10-kubeadm.conf and run ‘systemctl daemon-reload’.

```
root@master:~# cat /etc/systemd/system/kubelet.service.d/10-kubeadm.conf 
[Service] 
Environment="KUBELET_KUBECONFIG_ARGS=--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf" 
Environment="KUBELET_SYSTEM_PODS_ARGS=--pod-manifest-path=/etc/kubernetes/manifests --allow-privileged=true" 
Environment="KUBELET_NETWORK_ARGS=--network-plugin=cni --cni-conf-dir=/etc/cni/net.d --cni-bin-dir=/opt/cni/bin" 
Environment="KUBELET_DNS_ARGS=--cluster-dns=10.96.0.10 --cluster-domain=cluster.local" 
Environment="KUBELET_AUTHZ_ARGS=--authorization-mode=Webhook --client-ca-file=/etc/kubernetes/pki/ca.crt" 
Environment="KUBELET_CADVISOR_ARGS=--cadvisor-port=0" 
Environment="KUBELET_CERTIFICATE_ARGS=--rotate-certificates=true --cert-dir=/var/lib/kubelet/pki" 
ExecStart= 
ExecStart=/usr/bin/kubelet $KUBELET_KUBECONFIG_ARGS $KUBELET_SYSTEM_PODS_ARGS $KUBELET_NETWORK_ARGS $KUBELET_DNS_ARGS $KUBELET_AUTHZ_ARGS $KUBELET_CADVISOR_ARGS $KUBELET_CERTIFICATE_ARGS $KUBELET_EXTRA_ARGS --fail-swap-on=false
root@master:~# 
```

3. Run `kubeadm init --ignore-preflight-errors=FileExisting-crictl,Swap` on the master.
[https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm/)

4. Install Calico.

```
kubectl apply -f https://docs.projectcalico.org/v2.6/getting-started/kubernetes/installation/hosted/kubeadm/1.6/calico.yaml
```

[https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm](https://kubernetes.io/docs/setup/independent/create-cluster-kubeadm)

5. On worker nodes, run the `kubeadm join` command that was output by the previous `kubeadm init`.

6. On the master and worker nodes, get the kubelet/kubectl binary with vertical scaling.

Avaliable at [http://arlab224.austin.ibm.com:8000/_output/bin/kubelet](http://arlab224.austin.ibm.com:8000/_output/bin/kubelet), [http://arlab224.austin.ibm.com:8000/_output/bin/kubectl](http://arlab224.austin.ibm.com:8000/_output/bin/kubectl)

7. On the master and worker nodes, restart kubelet with the new binary. (systemctl stop kubelet, copy the new binary into /usr/bin/kubelet, system start kubelet)

With `kubectl get nodes`, you should see that on nodes custom kubelet is running.

```
root@master:~# kubectl  get nodes 
NAME       STATUS    ROLES     AGE       VERSION 
master     Ready     master    1h        v1.10.0-alpha.1.755+5c1eeb1b172242-dirty
worker-1   Ready     <none>    1h        v1.10.0-alpha.1.755+5c1eeb1b172242-dirty
```

8. On the master node, modify manifest yaml files (loated at `/etc/kubernetes/manifest`) for apiserver, scheduler, and controller-manager to use the corresponding image of each with vertical scaling from ‘youngjaelee’ repository.

For scheduler, modify the line for `image` to `image:youngjaelee/kube-scheduler-vscaling:021318` in `/etc/kubernetes/kube-scheduler.yaml`.

For apiserver, `image:youngjaee/kube-apiserver-vscaling:021318` in `/etc/kubernetes/kube-apiserver.yaml`.

For controller-manager, `image:youngjaelee/kube-controller-manager-vscaling:021318` in `/etc/kubernetes/kube-controller-manager.yaml`.

After modifying all these files, restart Kubetlet via systemctl restart kubelet.

9. If in output by kubelet get pods —all-namespaces, you see running pods for scheduler, apiserver, and controller-manager like the following, the switch to vertical scaling version is done.

```
root@master:~# kubectl  get pods --all-namespaces 
NAMESPACE     NAME                                      READY     STATUS    RESTARTS   AGE 
kube-system   calico-etcd-fx9nb                         1/1       Running   1          1h 
kube-system   calico-kube-controllers-d554689d5-dh962   1/1       Running   3          1h 
kube-system   calico-node-gnxhx                         2/2       Running   3          1h 
kube-system   calico-node-rm765                         2/2       Running   2          1h 
kube-system   etcd-master                               1/1       Running   0          1h 
kube-system   kube-apiserver-master                     1/1       Running   0          5m 
kube-system   kube-controller-manager-master            1/1       Running   0          5m 
kube-system   kube-dns-6f4fd4bdf-xk9l9                  3/3       Running   3          1h 
kube-system   kube-proxy-5cc8h                          1/1       Running   1          1h 
kube-system   kube-proxy-g6rqw                          1/1       Running   1          1h 
kube-system   kube-scheduler-master                     1/1       Running   0          5m 
root@master:~# 
```

## Support

If you have questions, reach out to us (leeyo@us.ibm.com)

