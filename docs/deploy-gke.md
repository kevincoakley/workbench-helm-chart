Google Kubernetes Engine Deployment
===================================

Deploy Instructions
-------------------

1. Provision an External IP Address in the region that the GKE cluster will be 
deployed and write down the External IP Address that was assigned. 
1. Create DNS hostname for the Workbench and point it the External IP Address. 
    1. Using one of the .googleusercontent.com domains with a self signed SSL cert is 
    impossible due to HTTP Strict Transport Security (HSTS).  
1. Create GKE cluster.
1. Install Helm.
1. Provision the NFS server using Helm.
    1. `helm install --name nfs-server-provisioner stable/nfs-server-provisioner --set=persistence.enabled=True,persistence.size=200Gi`
1. Provision nginx-ingress using Helm.
    1. Update values in examples/nginx-ingress-values.yaml
    1. `helm install stable/nginx-ingress --name nginx-ingress --values ./docs/examples/nginx-ingress-values.yaml`
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
