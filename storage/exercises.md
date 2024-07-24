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

  Now let's create the StorageClass:
  ```
  kubectl apply -f storageclasses.yaml 
  storageclass.storage.k8s.io/green-stc created
  ```
