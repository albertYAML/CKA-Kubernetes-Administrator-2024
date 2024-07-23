# Architecture, Installation & Maintenance
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
  First of all, we will dry-run this command to get a YAML template.
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
  <b>volumes:</ins>
  <b>- name: config-volume</b>
    <b>configMap:</b>
      <b>name: log-configmap</b>
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
</pre>
