# minikube-python-kafka
Little project to handle a python program + kafka.
Install process was done on macosx.

# Install pyenv
```
$ brew update && brew install pyenv
# Only do this the first time you setup your local env, might change if you are using another term
$ echo "eval '$(pyenv init -)'" >> ~/.bash_profile
$ pyenv install 3.7.4
$ pyenv local 3.7.4
# The next should return Python 3.7.4
$ python -V
```

# Install minikube
```
$ brew update && brew install minikube
# This can take a little while, admin password will be asked (option is mainly there to enable the metrics server)
$ minikube start --extra-config=kubelet.authentication-token-webhook=true
$ kubectl get pods --all-namespaces
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE
kube-system   coredns-6955765f44-fn62g           1/1     Running   0          63s
kube-system   coredns-6955765f44-jds5r           1/1     Running   0          63s
kube-system   etcd-minikube                      1/1     Running   0          69s
kube-system   kube-addon-manager-minikube        1/1     Running   0          69s
kube-system   kube-apiserver-minikube            1/1     Running   0          69s
kube-system   kube-controller-manager-minikube   1/1     Running   0          69s
kube-system   kube-proxy-tq2rx                   1/1     Running   0          62s
kube-system   kube-scheduler-minikube            1/1     Running   0          69s
kube-system   storage-provisioner                1/1     Running   0          60s
$ minikube addons enable registry
✅  registry was successfully enabled
```

# Install helm & download kafka + metrics server (optional if you already have helm installed)
```
$ brew install helm
$ helm search hub kafka
# If it is the first time you setup helm, you should have helm3+ so run these commands to load the charts
$ helm repo add stable https://kubernetes-charts.storage.googleapis.com
$ helm repo add bitnami https://charts.bitnami.com/bitnami
$ helm search repo kafka
# The next three commands are useless as far as I commited the charts inside the repo
$ mkdir charts && cd charts
$ helm pull bitnami/kafka --untar
$ helm pull stable/metrics-server --untar
# To have the initial setup for the next commands, create new folder 'charts' and move all your charts inside it
```

# Install metrics-server, kafka, and local docker registry
```
$ helm install broker-0 ./charts/kafka/
# Note that the one inside the git repository has some extra args to make the metrics server work with minikube
$ helm install metrics ./charts/metrics-server/
$ kubectl get pods --all-namespaces
NAMESPACE     NAME                                      READY   STATUS    RESTARTS   AGE
default       broker-0-kafka-0                          1/1     Running   1          2m38s
default       broker-0-zookeeper-0                      1/1     Running   0          2m38s
default       metrics-metrics-server-7b7f68b575-6md6c   1/1     Running   0          50s
kube-system   coredns-6955765f44-fn62g                  1/1     Running   0          3h33m
kube-system   coredns-6955765f44-jds5r                  1/1     Running   0          3h33m
kube-system   etcd-minikube                             1/1     Running   0          3h33m
kube-system   kube-addon-manager-minikube               1/1     Running   0          3h33m
kube-system   kube-apiserver-minikube                   1/1     Running   0          3h33m
kube-system   kube-controller-manager-minikube          1/1     Running   0          3h33m
kube-system   kube-proxy-tq2rx                          1/1     Running   0          3h33m
kube-system   kube-scheduler-minikube                   1/1     Running   0          3h33m
kube-system   registry-proxy-cq9f9                      1/1     Running   0          3h33m
kube-system   registry-wrq4k                            1/1     Running   0          3h33m
kube-system   storage-provisioner                       1/1     Running   0          3h33m
$ kubectl get --raw "/apis/metrics.k8s.io/v1beta1/nodes"
{"kind":"NodeMetricsList","apiVersion":"metrics.k8s.io/v1beta1","metadata":{"selfLink":"/apis/metrics.k8s.io/v1beta1/nodes"},"items":[{"metadata":{"name":"minikube","selfLink":"/apis/metrics.k8s.io/v1beta1/nodes/minikube","creationTimestamp":"2020-01-30T14:00:27Z"},"timestamp":"2020-01-30T13:59:58Z","window":"30s","usage":{"cpu":"321069224n","memory":"1597272Ki"}}]}
$ kubectl apply -f charts/minikube-docker-registry/
daemonset.apps/registry-aliases-hosts-update created
configmap/registry-aliases created
# Note that yq is required for this script (brew install it!)
$ bash charts/minikube-docker-registry/patch-coredns.sh
configmap/coredns patched
```
