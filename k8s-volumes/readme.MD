# k8s Volumes:

* its not good practice to store the important data like logs or busiess data at Pod or host level. because most of the k8s resources are ephemeral. if Pod or Node is crashed, all the data stored is gone.

**Container Storage is Temporary:**

* Any files saved inside a container are lost if the container crashes, restarts, or stops. This is because containers are designed to have temporary storage by default.

**Why This is a Problem:**

* For complex applications, this can cause issues. For example: If the container saves important data (like user information, logs, or temporary work files), that data is gone if the container crashes or restarts.
* If multiple containers in a pod need to share data, they can’t easily share files since each container has its own temporary storage.

**How Kubernetes Volumes Help:**

* Kubernetes provides a volume feature that allows you to create storage outside the container that is more permanent. This storage can survive even if the container crashes or restarts, meaning important data isn’t lost.
* Volumes can also be shared by multiple containers in the same pod, making it easier for containers to work together by accessing common files.

In Kubernetes on AWS, EBS (Elastic Block Store) and EFS (Elastic File System) are two common storage options that provide persistent storage for applications.

**Amazon EBS (Elastic Block Store):**

**How it works:** EBS is like an external hard drive attached to a single worker node in your Kubernetes cluster. Each EBS volume is only accessible by one node at a time.

**Use case:** Good for applications that need block storage with high performance, like databases, where only one instance accesses the data.

**In Kubernetes:** create a PersistentVolume (PV) that uses an EBS volume. This volume is attached to the pod running on the specific node, providing fast, single-node storage.

Limitation: If a pod needs to move to another node (say, in a failover), the EBS volume has to detach and reattach, which takes a bit of time.

There are two ways PVs may be provisioned: 
statically or dynamically.

`static provisioning` is a manual process. we have to get required approvals from manager and then request the administrator to create a volume/disk on EBS.

`dynamic provisioning` lets kubernetes automatically create storage based on the specified requirement. all we need to do is instruct k8s how to create volume dynamically by specifiying characterstics like storagetype, when to create volume, what is recliam policy by configuring `StorageClass`.

**simple steps to attach EBS volume to Node:**

* Create an EBS volume with the required size (in GB) and `select the same availability zone as the worker node` where the pod is running. This volume will then be attached to that node later steps.

* install EBS CSI driver

* Cluster IAM Role must have access to EBS service.

* Create a K8S PersistentVolume resource by specifying the EBS volume ID of the volume created in the previous step. PV now points to external volume created out side the cluster.

* Cliam the required volume by creating  `PersistentVolumeClaim` resource.

* configure `volumes and volumeMounts` for a pod that requires volume for storage.

**For dynamic provisioning, volumes are not created manually by the administrator. Kubernetes automatically creates the EBS volume based on the StorageClass configuration.**

**reamining all steps are common for static and dynamic provisioning except few config changes**

**For static provisioning, we will pass storageClassName as empty string and for dyanamic provisioning, we have to pass the StorageClass resource name**

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: expense-ebs
reclaimPolicy: Retain
provisioner: ebs.csi.aws.com
volumeBindingMode: WaitForFirstConsumer
```

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ebs-dynamic
spec:
  storageClassName: "expense-ebs" # reference to storge class name created in 02-dynamic-storage-class
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```