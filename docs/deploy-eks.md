Elastic Container Service for Kubernetes (Amazon EKS) Deployment
================================================================

Deploy Instructions
-------------------

1. [Create EKS cluster and add workers nodes.](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html). 
1. Install Helm.
1. Provision the NFS server using Helm.
    1. If you want to use a new EBS Disk every time the Helm Chart is installed:
        1. `helm install --name nfs-server-provisioner stable/nfs-server-provisioner --set=persistence.enabled=True,persistence.size=200Gi`
    1. If you want to reuse the same EBS Volume every time the Helm Chart is installed:
        1. Provision an EBS Volume in the region and AZ that the EKS workers were deployed, write down the Volume ID and the Size of the EBS Volume. This step only needs to be done once.
        1. Create a PersistentVolume referencing the EBS Volume from the last step.
            1. Update the values in eks-examples/nfs-data-persistent-volume.yaml, add the Volume ID and Size of the EBS Volume. 
            1. `kubectl create -f ./docs/eks-examples/nfs-data-persistent-volume.yaml` This step has to be done every time a new EKS cluster is created.
        1. Provision nfs-server-provisioner using Helm.  
            1. `helm install stable/nfs-server-provisioner --name nfs-server-provisioner --values ./docs/eks-examples/nfs-server-provisioner-values.yaml` This step has to be done every time a new EKS cluster is created.
1. Add the ELB Application Load Balancer for nginx-ingress
    1. Update the security groups and subnet values in eks-examples/alb-ingress.yaml. 
    1. `kubectl create -f ./docs/eks-examples/alb-ingress.yaml`
1. Provision nginx-ingress using Helm.
    1. Update default-ssl-certificate in eks-examples/nginx-ingress-values.yaml if you decide not to use the workbench namespace for launching the workbench or if you change the tls secretName in values.yaml.
    1. `helm install stable/nginx-ingress --name nginx-ingress --values ./docs/eks-examples/nginx-ingress-values.yaml`
1. Update DNS CNAME with the ELB hostname
    1. `aws elb describe-load-balancers | grep DNSName`
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


Todo
----
