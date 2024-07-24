# Storage
NOTE: The following exercises can be found on the killercoda website, created by Sachin HR.
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
