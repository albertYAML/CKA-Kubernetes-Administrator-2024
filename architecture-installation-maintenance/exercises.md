# Architecture, Installation & Maintenance
NOTE: The following exercises can be found on the Killercoda website, created by Sachin HR.  
https://killercoda.com/sachin/course/CKA
## Exercise 1 - Log reader

  ### log-reader-pod pod is running, save **All pod logs** in a file podalllogs.txt  
  **Solution:**
  ```
  kubectl logs pods/log-reader-pod > podalllogs.txt
  ```
## Exercise 2 - Log reader

  ### alpine-reader-pod pod is running, save **All INFO and ERROR's** pod logs in podlogs.txt   
  **Solution:**
  ```
  kubectl logs alpine-reader-pod | grep -e INFO -e ERROR > podlogs.txt
  ```
## Exercise 3 - Pod log

  ### Create a Kubernetes Pod configuration to facilitate real-time monitoring of a log file. Specifically, you need to set up a Pod named alpine-pod-pod that runs an Alpine Linux container.

  ### Requirements:
  - Name the Pod alpine-pod-pod  
  - Use alpine:latest image  
  - Container name alpine-container  
  - Configure the container to execute the tail -f /config/log.txt command (using args) with /bin/sh (using command) to continuously monitor and display the contents of a log file.  
  - Set up a volume named config-volume that maps to a ConfigMap named log-configmap, this log-configmap already available.  
  - Ensure the Pod has a restart policy of Never.  

  **Solution:**  
  First of all, we will dry-run this command to get a YAML POD template.
  ```
  kubectl run alpine-pod-pod --image=alpine:latest --restart=Never --dry-run=client -oyaml > pod.yaml
  ```
  ```
  cat pod.yaml
  ```
  ```
  apiVersion: v1
  kind: Pod
  metadata:
    creationTimestamp: null
    labels:
      run: alpine-pod-pod
    name: alpine-pod-pod
  spec:
    containers:
    - image: alpine:latest
      name: alpine-pod-pod
      resources: {}
    dnsPolicy: ClusterFirst
    restartPolicy: Never
  status: {}
  ```
  Now, modify the YAML adding/modifying these bold lines:
  ```
  vim pod.yaml
  ```
  <pre>
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: alpine-pod-pod
  name: alpine-pod-pod
spec:
  containers:
  - image: alpine:latest
    <b>name: alpine-container</b>
    <b>command: ['/bin/sh', '-c']</b>
    <b>args:</b>
    <b>- 'tail -f /config/log.txt'</b>
    <b>volumeMounts:</b>
    <b>- mountPath: '/config'</b>
      <b>name: config-volume</b>
  <b>volumes:</b>
  <b>- name: config-volume</b>
    <b>configMap:</b>
      <b>name: log-configmap</b>
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
</pre>

Finally, replace the old pod with the following command.
```
kubectl replace -f pod.yaml --force
```

## Exercise 4 - Pod log 1

  ### product pod is running. when you access logs of this pod, it displays the output **"Mi Tv Is Good"**. Please update the pod definition file to utilize an environment variable with the value "Sony Tv Is Good". Then, recreate this pod with the modified configuration.

  **Solution:**  
  First of all, we will dry-run this command to get a YAML template.
  ```
  kubectl get pod product -oyaml > pod.yaml
  ```
  The next step is find the line where the command echo is and replace the content 'Mi Tv Is Good' for 'Sony Tv Is Good'
  ```
  vim pod.yaml
  ```
  <pre>
apiVersion: v1
kind: Pod
metadata:
...
-------------------------------------------
  - command:
    - sh
    - -c
    <b>- echo 'Sony Tv Is Good' && sleep 3600</b>
    image: busybox
-------------------------------------------
...
  </pre>

Then, replace the old pod with the following command.
```
kubectl replace -f pod.yaml --force
```

## Exercise 5 - Create Pod

  ### Create a pod called sleep-pod  using the nginx  image and also sleep (using command ) for give any value for seconds.
  **Solution:**  
  First of all, we will dry-run this command to get a YAML POD template.
  ```
  kubectl run sleep-pod --image=nginx --dry-run=client -oyaml > pod.yaml
  ```
  Add the command line under container branch with the sleep command. In this case, I put 3600 seconds as an example.
  ```
  vim pod.yaml
  ```
  <pre>
  apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: sleep-pod
  name: sleep-pod
spec:
  containers:
  - image: nginx
    name: sleep-pod
    command: ['sleep', '3600']
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Always
status: {}
  </pre>

  Finally, replace the old pod with the following command.
  ```
  kubectl replace -f pod.yaml --force
  ```

## Exercise 6 - Pod resource

  ### Find the pod that consumes the most CPU in all namespace (including kube-system) in all cluster (currently we have single cluster). Then, store the result in the file high_cpu_pod.txt with the following format: pod_name,namespace 
  **Solution:**  
  With the following command, we will find the pods that consume more CPU, including the -A parameter to search in all namespaces.
  ```
  kubectl top pod --sort-by cpu -A
  NAMESPACE            NAME                                       CPU(cores)   MEMORY(bytes)   
  kube-system          kube-apiserver-controlplane                28m          233Mi           
  kube-system          canal-wzjz6                                13m          99Mi            
  kube-system          etcd-controlplane                          11m          44Mi            
  kube-system          canal-q652m                                11m          98Mi            
  kube-system          kube-controller-manager-controlplane       8m           73Mi            
  default              redis                                      2m           3Mi             
  kube-system          metrics-server-675d858c9-2qvbl             2m           13Mi            
  kube-system          kube-scheduler-controlplane                2m           23Mi            
  kube-system          calico-kube-controllers-75bdb5b75d-2b6mr   2m           20Mi            
  kube-system          kube-proxy-nhmtq                           1m           29Mi            
  kube-system          kube-proxy-dp5fn                           1m           18Mi            
  default              httpd                                      1m           6Mi             
  kube-system          coredns-5c69dbb7bd-xfk7l                   1m           30Mi            
  kube-system          coredns-5c69dbb7bd-6xvhl                   1m           10Mi            
  local-path-storage   local-path-provisioner-75655fcf79-xw9b6    1m           16Mi            
  default              nginx                                      0m           2Mi  
  ```
  Now create the .txt with this following command.
  ```
  echo "kube-apiserver-controlplane,kube-system" > high_cpu_pod.txt
  ```

## Exercise 7 - Pod resource

  ### You have a script named pod-filter.sh - Update this script to include a command that filters and displays the value of application of a pod named nginx-pod using jsonpath only. It should be in the format kubectl get pod <pod-name> <remainingcmd>  
  **Solution:**
  We have the script pod-filter.sh:
  ```
  cat pod-filter.sh
  #!/bin/bash
  ```
  With this following command using jsonpath we will get the value of application label.
  ```
  kubectl get pod nginx-pod -o jsonpath="{.metadata.labels.application}"
  frontend
  ```
  Now, add this kubectl command to the pod-filter.sh script:
  ```
  vim pod-filter.sh
  kubectl get pod nginx-pod -o jsonpath="{.metadata.labels.application}"
  ```

## Exercise 8 - Pod resource

  ### etcd-controlplane pod is running in kube-system environment, take backup and store it in /opt/cluster_backup.db file. Now, ETCD backup is stored at the path /opt/cluster_backup.db on the controlplane node. For --data-dir use /root/default.etcd , restore it on the controlplane node itself and also store restore console output store it in restore.txt 
  **Solution:**
  The first thing to do would be to have the cacert , cert and etcd key paths. For it, we can make a describe of the pod that is in the namespace kube-system and write down those values.
  <pre>
  kubectl describe pod etcd-controlplane -n kube-system
    etcd:
    Container ID:  containerd://7c23064e76200e45c9580c87bd181d506b7c9526a9c178528b272f1f5992708f
    Image:         registry.k8s.io/etcd:3.5.12-0
    Image ID:      registry.k8s.io/etcd@sha256:44a8e24dcbba3470ee1fee21d5e88d128c936e9b55d4bc51fbef8086f8ed123b
    Port:          <none>
    Host Port:     <none>
    Command:
      etcd
      --advertise-client-urls=https://172.30.1.2:2379
      <b>--cert-file=/etc/kubernetes/pki/etcd/server.crt</b>
      --client-cert-auth=true
      --data-dir=/var/lib/etcd
      --experimental-initial-corrupt-check=true
      --experimental-watch-progress-notify-interval=5s
      --initial-advertise-peer-urls=https://172.30.1.2:2380
      --initial-cluster=controlplane=https://172.30.1.2:2380
      <b>--key-file=/etc/kubernetes/pki/etcd/server.key</b>
      --listen-client-urls=https://127.0.0.1:2379,https://172.30.1.2:2379
      --listen-metrics-urls=http://127.0.0.1:2381
      --listen-peer-urls=https://172.30.1.2:2380
      --name=controlplane
      --peer-cert-file=/etc/kubernetes/pki/etcd/peer.crt
      --peer-client-cert-auth=true
      --peer-key-file=/etc/kubernetes/pki/etcd/peer.key
      --peer-trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt
      --snapshot-count=10000
      <b>--trusted-ca-file=/etc/kubernetes/pki/etcd/ca.crt</b>
  </pre>
  Now that we have the paths, we will proceed to write the following command where we indicate that we want to save the snapshot in /opt/cluster_backup.db
  ```
  ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /opt/cluster_backup.db

  {"level":"info","ts":1721823517.923694,"caller":"snapshot/v3_snapshot.go:68","msg":"created temporary db file","path":"/opt/cluster_backup.db.part"}
  {"level":"info","ts":1721823517.942504,"logger":"client","caller":"v3/maintenance.go:211","msg":"opened snapshot stream; downloading"}
  {"level":"info","ts":1721823517.9427404,"caller":"snapshot/v3_snapshot.go:76","msg":"fetching snapshot","endpoint":"https://127.0.0.1:2379"}
  {"level":"info","ts":1721823518.0707304,"logger":"client","caller":"v3/maintenance.go:219","msg":"completed snapshot read; closing"}
  {"level":"info","ts":1721823518.151457,"caller":"snapshot/v3_snapshot.go:91","msg":"fetched snapshot","endpoint":"https://127.0.0.1:2379","size":"6.6 MB","took":"now"}
  {"level":"info","ts":1721823518.1519055,"caller":"snapshot/v3_snapshot.go:100","msg":"saved","path":"/opt/cluster_backup.db"}

  Snapshot saved at /opt/cluster_backup.db
  ```
  The last of the exercise is to restore the snapshot and save the log, which we will do as follows:
  ```
  etcdctl --data-dir /root/default.etcd  snapshot restore /opt/cluster_backup.db > restore.txt
  ```
  ```
  cat restore.txt
  2024-07-24T12:24:07Z    info    snapshot/v3_snapshot.go:251     restoring snapshot      {"path": "/opt/cluster_backup.db", "wal-dir": "/root/default.etcd/member/wal", "data-dir": "/root/default.etcd", "snap-dir": "/root/default.etcd/member/snap", "stack": "go.etcd.io/etcd/etcdutl/v3/snapshot.(*v3Manager).Restore\n\t/tmp/etcd-release-3.5.0/etcd/release/etcd/etcdutl/snapshot/v3_snapshot.go:257\ngo.etcd.io/etcd/etcdutl/v3/etcdutl.SnapshotRestoreCommandFunc\n\t/tmp/etcd-release-3.5.0/etcd/release/etcd/etcdutl/etcdutl/snapshot_command.go:147\ngo.etcd.io/etcd/etcdctl/v3/ctlv3/command.snapshotRestoreCommandFunc\n\t/tmp/etcd-release-3.5.0/etcd/release/etcd/etcdctl/ctlv3/command/snapshot_command.go:128\ngithub.com/spf13/cobra.(*Command).execute\n\t/home/remote/sbatsche/.gvm/pkgsets/go1.16.3/global/pkg/mod/github.com/spf13/cobra@v1.1.3/command.go:856\ngithub.com/spf13/cobra.(*Command).ExecuteC\n\t/home/remote/sbatsche/.gvm/pkgsets/go1.16.3/global/pkg/mod/github.com/spf13/cobra@v1.1.3/command.go:960\ngithub.com/spf13/cobra.(*Command).Execute\n\t/home/remote/sbatsche/.gvm/pkgsets/go1.16.3/global/pkg/mod/github.com/spf13/cobra@v1.1.3/command.go:897\ngo.etcd.io/etcd/etcdctl/v3/ctlv3.Start\n\t/tmp/etcd-release-3.5.0/etcd/release/etcd/etcdctl/ctlv3/ctl.go:107\ngo.etcd.io/etcd/etcdctl/v3/ctlv3.MustStart\n\t/tmp/etcd-release-3.5.0/etcd/release/etcd/etcdctl/ctlv3/ctl.go:111\nmain.main\n\t/tmp/etcd-release-3.5.0/etcd/release/etcd/etcdctl/main.go:59\nruntime.main\n\t/home/remote/sbatsche/.gvm/gos/go1.16.3/src/runtime/proc.go:225"}
  2024-07-24T12:24:07Z    info    membership/store.go:119 Trimming membership information from the backend...
  2024-07-24T12:24:07Z    info    membership/cluster.go:393       added member    {"cluster-id": "cdf818194e3a8c32", "local-member-id": "0", "added-peer-id": "8e9e05c52164694d", "added-peer-peer-urls": ["http://localhost:2380"]}
  2024-07-24T12:24:07Z    info    snapshot/v3_snapshot.go:272     restored snapshot       {"path": "/opt/cluster_backup.db", "wal-dir": "/root/default.etcd/member/wal", "data-dir": "/root/default.etcd", "snap-dir": "/root/default.etcd/member/snap"}
  ```
  