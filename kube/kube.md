# [kube Documention](https://kubernetes.io/docs/home/)
https://www.youtube.com/watch?v=EQNO_kM96Mo

# [Cluster Architecture ](https://kubernetes.io/docs/concepts/architecture/)
## Control plane components 

1. ### `kube-apiserver`
2. ### `etcd`
3. ### `kube-scheduler` 
4. ### `kube-controller-manager` 
5. ### `cloud-controller-manager` 

## Node components

1. ### `kubelet`
2. ### `kube-proxy (optional)` 
3. ### `Container runtime`

# [Install and Set Up kubectl on Linux](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/#install-using-native-package-management)

## Installing kubeadm
1. ### Update the apt package index
   ```sh 
   sudo apt-get update 
   ```
2. ###  nstall packages needed to use the Kubernetes apt repository
   ```sh 
   sudo apt-get install -y apt-transport-https ca-certificates curl gpg
   ```
3. ### Download the public signing key for the Kubernetes package repositories.
   ```sh
   # If the folder "/etc/apt/keyrings" does not exist, it should be created before the curl command, read the note below.
   # sudo mkdir -p -m 755 /etc/apt/keyrings
   curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
   ```
4. ### allow unprivileged APT programs to read this keyring
   ```sh 
   sudo chmod 644 /etc/apt/keyrings/kubernetes-apt-keyring.gpg
   ```
5. ### Add the appropriate Kubernetes apt repository
   ```sh
   echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
   ```
6. ### helps tools such as command-not-found to work correctly
   ```sh
   sudo chmod 644 /etc/apt/sources.list.d/kubernetes.list
   ```

7. ### Update apt package index, then install kubectl
   ```sh
   sudo apt-get update
   sudo apt-get install -y kubectl
   ```
8. ### nstall kubelet, kubeadm and kubectl, and pin their version:
   ```sh
   sudo apt-get install -y kubelet kubeadm kubectl
   sudo apt-mark hold kubelet kubeadm kubectl
   ```
----
after install kubernates do the coming
# [Container Runtimes](https://v1-28.docs.kubernetes.io/docs/setup/production-environment/container-runtimes/)

## Install and configure prerequisites 


1. #### Enable IPv4 packet forwarding
    ```bash
      # sysctl params required by setup, params persist across reboots
      cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
      net.ipv4.ip_forward = 1
      EOF   

      # Apply sysctl params without reboot
      sudo sysctl --system

      #Verify that net.ipv4.ip_forward is set to 1 with:
      sysctl net.ipv4.ip_forward
    ```

2. #### Verify that the net.bridge.bridge-nf-call-iptables, net.bridge.bridge-nf-call-ip6tables, and net.ipv4.ip_forward system variables are set to 1 in your sysctl config 
   ```sh
      sysctl net.bridge.bridge-nf-call-iptables net.bridge.bridge-nf-call-ip6tables net.ipv4.ip_forward
   ```

3. #### Configuring the systemd cgroup driver
   To use the systemd cgroup driver in /etc/containerd/config.toml with runc, set
   ```sh
    [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
      ...
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
        SystemdCgroup = true
   ```

4. #### If you apply this change, make sure to restart containerd:
   ```bash
       sudo systemctl restart containerd
   ```
------------------------------------------------------------
[to use dockerd isntead of containerd](https://www.mirantis.com/blog/how-to-install-cri-dockerd-and-migrate-nodes-from-dockershim/)
------------------------------------------------------------
```sh
kubectl describe pod <pod name>
kubectl describe deplyoment <deplyoment name>
kubectl describe service <service name>

kubectl get pods
kubectl get service

kubectl logs pod <pod name> 
```
------------------------------------

# install ingress controller

## kubectl apply -f deployments/rbac/rbac.yaml
```sh
git clone https://github.com/nginxinc/kubernetes-ingress.git --branch v4.0.0
```
### Change the active directory.
```sh
cd kubernetes-ingress
```
### Create a namespace and a service account:
```sh
kubectl apply -f deployments/common/ns-and-sa.yaml
```

### Create a cluster role and binding for the service account:
```sh
kubectl apply -f deployments/rbac/rbac.yaml
```

Create common resources


```sh
kubectl apply -f examples/shared-examples/default-server-secret/default-server-secret.yaml
```

Create a ConfigMap to customize your NGINX settings:
```sh
kubectl apply -f examples/shared-examples/default-server-secret/default-server-secret.yaml
```

Create an IngressClass resource. NGINX Ingress Controller won’t start without an IngressClass resource.
```sh
kubectl apply -f deployments/common/ingress-class.yaml
```

Create core custom resources
INSTALL CRDS FROM SINGLE YAML
```sh
kubectl apply -f https://raw.githubusercontent.com/nginxinc/kubernetes-ingress/v4.0.0/deploy/crds.yaml
```

Deploy NGINX Ingress Controller
```sh
kubectl apply -f deployments/deployment/nginx-ingress.yaml
```

Confirm NGINX Ingress Controller is running
```sh
kubectl get pods --namespace=nginx-ingress
```

-------------------------------------------------------------------

# For monotiring :-
[Monotiring Link](https://github.com/kubernetes-sigs/metrics-server)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: metrics-server
  namespace: kube-system
  labels:
    k8s-app: metrics-server
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
  template:
    metadata:
      name: metrics-server
      labels:
        k8s-app: metrics-server
    spec:
      serviceAccountName: metrics-server
      volumes:
      # mount in tmp so we can safely use from-scratch images and/or read-only containers
      - name: tmp-dir
        emptyDir: {}
      containers:
      - name: metrics-server
        image: k8s.gcr.io/metrics-server/metrics-server:v0.3.7
        imagePullPolicy: IfNotPresent
        args:
          - --kubelet-insecure-tls
          - --cert-dir=/tmp
          - --secure-port=4443
        ports:
        - name: main-port
          containerPort: 4443
          protocol: TCP
        securityContext:
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 1000
        volumeMounts:
        - name: tmp-dir
          mountPath: /tmp
      nodeSelector:
        kubernetes.io/os: linux
```
---------------------------------------------
you must install web server (apache or nginx)
