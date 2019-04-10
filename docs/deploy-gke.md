Google Kubernetes Engine Deployment
===================================

Deploy Instructions
-------------------

1. Create GKE cluster.
1. Install Helm.

**Storage** 

1. Home Directories 
    1. If you don't have an NFS server then you can provision an NFS server in Kubernetes using Helm.   
        1. If you want to use a new GCE Disk every time the Helm Chart is installed:
            1. `helm install --name nfs-server-provisioner stable/nfs-server-provisioner --set=persistence.enabled=True,persistence.size=200Gi`
        1. If you want to reuse the same GCE Disk every time the Helm Chart is installed:
            1. Provision an GCE Disk in the region that the GKE cluster was deployed, write down the Name and the Size of the GCE Disk. This step only needs to be done once.
            1. Create a PersistentVolume referencing the GCE Disk from the last step.
                1. Update the values in gke-examples/nfs-data-persistent-volume.yaml, add the Name and Size of the GCE disk. 
                1. `kubectl create -f ./docs/gke-examples/nfs-data-persistent-volume.yaml` This step has to be done every time a new GKE cluster is created.
            1. Provision nfs-server-provisioner using Helm.  
                1. `helm install stable/nfs-server-provisioner --name nfs-server-provisioner --values ./docs/gke-examples/nfs-server-provisioner-values.yaml` This step has to be done every time a new GKE cluster is created.
    1. If you want to use your own NFS server:
        1. Create a StorageClass backed by the NFS server:
            1. Update nfs.server, nfs.path and nfs.mountOptions in ./docs/gke-examples/nfs-client-provisioner-values.yaml.
            1. `helm install stable/nfs-client-provisioner --name nfs-client-provisioner --values ./docs/gke-examples/nfs-client/nfs-client-provisioner-values.yaml`
        1. Update the Workbench Helm Chart ./values.yaml to use the NFS backed Storage Class:
            1. home_storage.pvc_storage_class: match storageClass.name in ./docs/gke-examples/nfs-client/nfs-client-provisioner-values.yaml. The Default is nfs-client.         
1. ETCD
    1. If you want to use a new GCE Disk every time the Helm Chart is installed then you don't have to do anything.
    1. If you want to reuse the same GCE Disk every time the Helm Chart is installed:  
        1. Provision an GCE Disk in the region that the GKE cluster was deployed, write down the Name and the Size of the GCE Disk. This step only needs to be done once.
        1. Update the Workbench Helm Chart ./values.yaml to use the GCE Disk:
             1. etcd_storage.persistent.type: "gce"
             1. etcd_storage.persistent.volume_id: the GCE Disk Name from the previous step. 
             1. etcd_storage.size: the GCE Disk Size from the previous step.

**Ingress**

1. Provision an External IP Address in the region that the GKE cluster will be 
deployed and write down the External IP Address that was assigned. 
1. Create DNS hostname for the Workbench and point it the External IP Address. 
    1. Using one of the .googleusercontent.com domains with a self signed SSL cert is 
    impossible due to HTTP Strict Transport Security (HSTS).  
1. Provision nginx-ingress using Helm.
    1. Update values in gke-examples/nginx-ingress-values.yaml, add the External IP Address that was provisioned in the first step. 
    1. Update default-ssl-certificate in gke-examples/nginx-ingress-values.yaml if you decide not to use the workbench namespace for launching the workbench or if you change the tls secretName in values.yaml.
    1. `helm install stable/nginx-ingress --name nginx-ingress --values ./docs/gke-examples/nginx-ingress-values.yaml`

**Workbench**

1. Update values.yaml:
    1. Update the domain, subdomain_prefix and support_email as appropriate.
    1. The webui and apiserver images should use the develop tag if using Kubernetes 
    version 1.10 or greater.
        1. webui: "ndslabs/angular-ui:develop"
        1. apiserver: "ndslabs/apiserver:develop" 
    1. To use gmail for SMTP:
        1. host: 
        1. port: 
        1. gmail_user: user@domain
        1. gmail_pass: <app password>        
    1. Add SSL cert/key.
1. Provision Workbench using Helm.
    1. `helm install . --name=workbench --namespace=workbench`