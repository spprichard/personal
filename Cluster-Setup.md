# Cluster Setup
This doc describes the process used to deploy a Kubernetes Cluster on Mac Mini's running Ubuntu 18.04. The links found in the additional resources section we mostly followed.

## Steps - On Server - Mac Mini - Master
1. Install kubeadm, kubectl, kubelet, docker.io
- As an example: `sudo apt-get install docker.io`

2. Disable Swap 
- `sudo swapoff -a`

3. Switch to super-user 
- `sudo su`

4. Init Cluster - Master Node
- `kubeadm init --pod-network-cidr=192.168.0.0/16`
- NOTE: `--pod-network-cidr` is needed for pod networking, which will be handled by Calico

5. Assuming everything worked....Config .kube dir
- Exit super-user `exit`
- `mkdir -p $HOME/.kube`
- `sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config`
- `sudo chown $(id -u):$(id -g) $HOME/.kube/config`

6. Write down the information about joining 
- At the bottom of the success message, you will see the token, and connection info for other nodes to join the cluster

7. Allow pods to be scheduled on the Maser Node (optional)
- `kubectl taint nodes --all node-role.kubernetes.io/master-`

At this point if you run `kubectl get node` you might see that the master node is not ready, this is due to the pod network not being setup. Don't panic...

8. Configure Pod Network using Calico
- `kubectl apply -f https://docs.projectcalico.org/v3.10/manifests/calico.yaml`
- NOTE: The version of calico matters here. At the time of writing this, using kubernetes version v1.17.0, version v3.10 of calico worked, while version v3.8 **DID NOT**

9. Verify Things are coming up 
- Set the current context to the `kube-system` namespace: `kubectl config set-context $(k config current-context) --namespace kube-system`
- `kubectl get po`
*should see something like this*
```
NAME                                       READY   STATUS              RESTARTS   AGE
calico-kube-controllers-74c9747c46-xb549   0/1     ContainerCreating   0          44s
calico-node-k4q4w                          1/1     Running             0          44s
coredns-6955765f44-lm7tp                   0/1     ContainerCreating   0          6m2s
coredns-6955765f44-ng49p                   1/1     Running             0          6m2s
etcd-mac-mini-1                            1/1     Running             0          6m13s
kube-apiserver-mac-mini-1                  1/1     Running             0          6m13s
kube-controller-manager-mac-mini-1         1/1     Running             0          6m13s
kube-proxy-nx2r8                           1/1     Running             0          6m2s
kube-scheduler-mac-mini-1                  1/1     Running             0          6m13s
```

10. Profit!

## Deploying Kubernetes Dashboard
- `kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml`


## Additional Resources
- https://medium.com/better-programming/build-your-own-multi-node-kubernetes-cluster-with-monitoring-346a7e2ef6e2
- https://www.mirantis.com/blog/how-install-kubernetes-kubeadm/