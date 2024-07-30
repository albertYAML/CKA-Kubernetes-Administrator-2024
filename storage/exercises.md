# Storage
NOTE: The following exercises can be found on the Killercoda website, created by Sachin HR.  
https://killercoda.com/sachin/course/CKA

## Exercise 1 - Storage Class
  ### Create a storage class called green-stc  as per the properties given below:
  ### Requirements:
  - Provisioner should be kubernetes.io/no-provisioner 
  - Volume binding mode should be WaitForFirstConsumer
  - Volume expansion should be enabled
  
  **Solution:**  
  It is not possible to make a template as we do with pods, for example using kubectl run pod -oyaml. For this, we can go to the official kubernetes documentation (kubernetes.io) and take an example, where we will leave the YAML file with the characteristics that the exercise asks for.
  ```
  vim storageclasses.yaml
  ```
  <pre>
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  <b>name: green-stc</b>
  annotations:
    storageclass.kubernetes.io/is-default-class: "false"
<b>provisioner: kubernetes.io/no-provisioner</b>
<b>allowVolumeExpansion: true</b>
<b>volumeBindingMode: WaitForFirstConsumer</b>
</pre>

  Now let's create the StorageClass, applying our created YAML file:
  ```
  kubectl apply -f storageclasses.yaml 
  storageclass.storage.k8s.io/green-stc created
  ```

## Exercise 2 - Persistent Volume Claim Resize
  ### Modify the size of the existing Persistent Volume Claim (PVC) named yellow-pvc-cka to request 60Mi of storage from the yellow-pv-cka volume. Ensure that the PVC successfully resizes to the new size and remains in the Bound state.

 **Solution:**  
  To modify the request of our PVC, we can do it in the following way:
  ```
  kubectl edit pvc yellow-pvc-cka
  ```

  <pre>
  apiVersion: v1
kind: PersistentVolumeClaim
...
------------------------------------------------
  resources:
    requests:
      <b>storage: 60Mi</b>
  storageClassName: yellow-stc-cka
  volumeMode: Filesystem
------------------------------------------------
...                  
  </pre>

  When the modification is saved, the following prompt should be displayed:
  ```
  persistentvolumeclaim/yellow-pvc-cka edited
  ```
  Verify that PVC remains in "Bound" state:
  ```
  kubectl get pvc   
  NAME             STATUS   VOLUME          CAPACITY   ACCESS MODES   STORAGECLASS     VOLUMEATTRIBUTESCLASS   AGE
  yellow-pvc-cka   Bound    yellow-pv-cka   100Mi      RWO            yellow-stc-cka   <unset>                 6m4s
  ```

## Exercise 3 - Persistent Volume Claim
  ### A persistent volume named red-pv-cka is available. Your task is to create a PersistentVolumeClaim (PVC) named red-pvc-cka and request 30Mi of storage from the red-pv-cka PersistentVolume (PV).
  ### Requirements:
  - Access mode: ReadWriteOnce
  - Storage class: manual 

 **Solution:**  
  It is not possible to make a template as we do with pods as we said. Create a .yaml archive with the following configuration and apply the YAML:
  ```
  vim pvc.yaml
  ```
  <pre>
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  <b>name: red-pvc-cka</b>
spec:
  accessModes:
    <b>- ReadWriteOnce</b>
  volumeMode: Filesystem
  resources:
    requests:
      <b>storage: 30Mi</b>
  <b>storageClassName: "manual"</b>
  </pre>

  ```
  kubectl apply -f pvc.yaml
  persistentvolumeclaim/red-pvc-cka created
  ```

## Exercise 4 - Persistent Volume
  ### Create a PersistentVolume (PV) named black-pv-cka with the following specifications:  
  ### Requirements:
  - Volume Type: hostPath
  - Path: /opt/black-pv-cka
  - Capacity: 50Mi

**Solution:**  
  It is not possible to make a template as we do with pods as we said. Create a .yaml archive with the following configuration and apply the YAML:
  ```
  vim pv.yaml
  ```
  <pre>
apiVersion: v1
kind: PersistentVolume
metadata:
  <b>name: black-pv-cka</b>
spec:
  capacity:
    <b>storage: 50Mi</b>
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  <b>hostPath:</b>
    <b>path: /opt/black-pv-cka</b>
  </pre>

  ```
  kubectl apply -f pv.yaml 
  persistentvolume/black-pv-cka created
  ```

  ## Exercise 5 - Persistent Volume Claim, Pod

  ### A Kubernetes pod definition file named nginx-pod-cka.yaml is available. Your task is to make the following modifications to the manifest file:
  ### Requirements:
  - Name the Pod nginx-pod-cka 
  - Create a Persistent Volume Claim (PVC) with the name nginx-pvc-cka. This PVC should request 80Mi of storage from an existing Persistent Volume (PV) named nginx-pv-cka and Storage Class named nginx-stc-cka . Use the access mode ReadWriteOnce.
  - Add the created nginx-pvc-cka PVC to the existing nginx-pod-cka POD definition.
  - Mount the volume claimed by nginx-pvc-cka at the path /var/www/html within the nginx-pod-cka POD.
  - Add tolerations with the key node-role.kubernetes.io/control-plane set to Exists and effect NoSchedule to the nginx-pod-cka Pod
  - Ensure that the peach-pod-cka05-str POD is running and that the Persistent Volume (PV) is successfully bound.

  **Solution:**
  This is the nginx-pod-cka.yaml template:
  ```
  apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-cka
spec:
  containers:
    - name: my-container
      image: nginx:latest
  ```
  The first step is to create the pvc.yaml file to create the Persisten Volume Claim, as we have done in other exercises.

  <pre>
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  <b>name: nginx-pvc-cka</b>
spec:
  accessModes:
    <b>- ReadWriteOnce</b>
  volumeMode: Filesystem
  resources:
    requests:
      <b>storage: 80Mi</b>
  <b>storageClassName: "nginx-stc-cka"</b>
  <b>volumeName: "nginx-pv-cka"</b>
  </pre>

  Then apply and we'll see that PVC is in "Pending" status.

  ```
  kubectl apply -f pvc.yaml 
  persistentvolumeclaim/nginx-pvc-cka created

  kubectl get pvc
  NAME                                  STATUS    VOLUME         CAPACITY   ACCESS MODES   STORAGECLASS    VOLUMEATTRIBUTESCLASS   AGE
  persistentvolumeclaim/nginx-pvc-cka   Pending   nginx-pv-cka   0                         nginx-stc-cka   <unset>                 3s
  ```

  The next step is to modify the nginx-pvc-cka.yaml and add the rest of the steps.
  To add the created nginx-pvc-cka PVC to the existing nginx-pod-cka POD definition:

  <pre>
  spec:
  .....
  ----------------------------------------
    <b>volumes:
      - name: nginx-pvc-cka
        persistentVolumeClaim:
          claimName: nginx-pvc-cka</b>
  ----------------------------------------
  .....
  </pre>

  Mount the volume claimed by nginx-pvc-cka at the path /var/www/html within the nginx-pod-cka POD:

  <pre>
  spec:
    containers:
      - name: my-container
        image: nginx:latest
        <b>volumeMounts:
          - mountPath: "/var/www/html"
            name: nginx-pvc-cka</b>
  ----------------------------------------
    .....
  </pre>

  Add tolerations with the key node-role.kubernetes.io/control-plane set to Exists and effect NoSchedule to the nginx-pod-cka Pod:

  <pre>
  .....
  ----------------------------------------
  spec:
    <b>tolerations:
    - key: "node-role.kubernetes.io/control-plane"
      operator: "Exists"
      effect: "NoSchedule"</b>
  ----------------------------------------
  .....
  </pre>

  You will have a nginx-pod-cka.yaml like this:

  ```
  apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod-cka
spec:
  containers:
    - name: my-container
      image: nginx:latest
      volumeMounts:
        - mountPath: "/var/www/html"
          name: nginx-pvc-cka
  volumes:
    - name: nginx-pvc-cka
      persistentVolumeClaim:
        claimName: nginx-pvc-cka
  tolerations:
  - key: "node-role.kubernetes.io/control-plane"
    operator: "Exists"
    effect: "NoSchedule"
  ```

  Now apply the pod yaml:

  ```
  kubectl apply -f nginx-pod-cka.yaml
  pod/nginx-pod-cka created
  ```
  Ensure that the nginx-pod-cka POD is running and that the Persistent Volume Claim (PVC) is successfully bound:
  ```
  kubectl get pods
  NAME            READY   STATUS    RESTARTS   AGE
  nginx-pod-cka   1/1     Running   0          2m10s

  kubectl get pvc
  NAME            STATUS   VOLUME         CAPACITY   ACCESS MODES   STORAGECLASS    VOLUMEATTRIBUTESCLASS   AGE
  nginx-pvc-cka   Bound    nginx-pv-cka   100Mi      RWO            nginx-stc-cka   <unset>                 15m
  ```

## Exercise 6 - Persistent Volume Claim, Pod

  ### Your task involves setting up storage components in a Kubernetes cluster. Follow these steps:
  ### Requirements:
  - Step 1: Create a Storage Class named blue-stc-cka with the following properties:
    Provisioner: kubernetes.io/no-provisioner
    Volume binding mode: WaitForFirstConsumer

  - Step 2: Create a Persistent Volume (PV) named blue-pv-cka with the following properties
    Capacity: 100Mi
    Access mode: ReadWriteOnce
    Reclaim policy: Retain
    Storage class: blue-stc-cka
    Local path: /opt/blue-data-cka
    Node affinity: Set node affinity to create this PV on controlplane

  - Step 3: Create a Persistent Volume Claim (PVC) named blue-pvc-cka with the following properties:
    Access mode: ReadWriteOnce
    Storage class: blue-stc-cka
    Storage request: 50Mi
    The volume should be bound to blue-pv-cka .

  **Solution:**
  First of all, we are gonna create the sc.yaml for the StorageClase Step 1:

    ```
  vim sc.yaml

  apiVersion: storage.k8s.io/v1
  kind: StorageClass
  metadata:
    name: blue-stc-cka
    annotations:
      storageclass.kubernetes.io/is-default-class: "false"
  provisioner: kubernetes.io/no-provisioner
  allowVolumeExpansion: true
  volumeBindingMode: WaitForFirstConsumer
  ```

  Then, apply the .YAML using kubectl apply.

  ```
  kubectl apply -f sc.yaml
  storageclass.storage.k8s.io/blue-stc-cka created
  ```

  In the second step, we will create the Persistent Volume yaml file with the following format:

  ```
  vim pv.yaml

  apiVersion: v1
  kind: PersistentVolume
  metadata:
    name: blue-pv-cka
  spec:
    capacity:
      storage: 100Mi
    volumeMode: Filesystem
    accessModes:
      - ReadWriteOnce
    persistentVolumeReclaimPolicy: Retain
    storageClassName: blue-stc-cka
    local:
      path: /opt/blue-data-cka
    nodeAffinity:
      required:
        nodeSelectorTerms:
          - matchExpressions:
            - key: hostname
              operator: In
              values:
              - controlplane
  ```

  Then, apply the .YAML using kubectl apply.

  ```
  kubectl apply -f pv.yaml
  persistentvolume/blue-pv-cka created
  ```

  The last step will be to create the pvc associated to the PV created previously.

  ```
  vim pvc.yaml

  apiVersion: v1
  kind: PersistentVolumeClaim
  metadata:
    name: blue-pvc-cka
  spec:
    accessModes:
      - ReadWriteOnce
    storageClassName: blue-stc-cka
    resources:
      requests:
        storage: 50Mi
    volumeName: blue-pv-cka
  ```

  We will check that the PVC is bound to the PV created and that both have the storageClass created.

  ```
  kubectl get pv,pvc
  NAME                           CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                  STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
  persistentvolume/blue-pv-cka   100Mi      RWO            Retain           Bound    default/blue-pvc-cka   blue-stc-cka   <unset>                          62s

  NAME                                 STATUS   VOLUME        CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
  persistentvolumeclaim/blue-pvc-cka   Bound    blue-pv-cka   100Mi      RWO            blue-stc-cka   <unset>                 2s
  ```
