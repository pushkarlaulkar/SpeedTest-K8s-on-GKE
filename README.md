Instructions to deploy **SpeedTest** on GCP GKE Auto Pilot cluster
  1. Deploy GKE Auto Pilot cluster GCP Console.
  2. Create a namespace. ` kubectl create ns speedtest `
  3. Deploy the **SpeedTest** deployment & service using the below command

     ```
     kubectl -n speedtest apply -f speedtest-dep.yml -f speedtest-svc.yml
     ```
  4. Deploy **Ingress**, **Managed Certificate** & **Frontend Config** which will create an ALB listening on port 443 by running below command. Before running below command, in the **Managed Certificate** put the domain name you need for your application.
     ```
     kubectl -n speedtest apply -f ingress-all.yml
     ```
  7. Run ` kubectl -n speedtest get ingress ` to retrieve the ALB IP ( Please wait couple of minutes ). Create an A record in **Cloud DNS** or your own DNS service pointing your domain name to this IP.
  8. Once the DNS entry has been added it will take couple of minutes ( can be 60 minutes in some cases ) for the certificate to be generated. Run ` kubectl -n speedtest get managedcertificate ` or in GCP console to check for **Active** status.
  9. Access the app using `https://your_domain_name`.

-----------------------------

**Helm**
To install this app using Helm, perform below steps
  1. Run the command

     ```
     helm install speedtest ./helm --namespace speedtest --create-namespace --set domain_name=domain_name
     ```
  2. Get the ALB IP using ` kubectl -n speedtest get ingress ` ( Please wait couple of minutes ). Create an A record in **Cloud DNS** or your own DNS service pointing your domain name to this IP.
  3. Once the DNS entry has been added it will take couple of minutes ( can be 60 minutes in some cases ) for the certificate to be generated. Run ` kubectl -n speedtest get managedcertificate ` or in GCP console to check for **Active** status.
  4. Access the app using ` https://your_domain_name `.
  5. Uninstall the app using ` helm uninstall speedtest --namespace speedtest `.

-----------------------------

**ArgoCD** installation. **NAT Gateway** is required to download **ArgoCD** container images.

To install this app using ArgoCD, perform below steps
  1. Create a namespace. ` kubectl create ns argocd `.
  2. Apply ArgoCD manifest file
     
     ```
     kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
     ```
  3. Patch the **argocd-server** service in the **argocd** namespace to add the below annotation
     
     ```
     kubectl patch service argocd-server -n argocd -p '{"metadata": {"annotations": {"cloud.google.com/backend-config": "{\"ports\": {\"http\": \"argocd-backend-config\"}}"}}}'
     ```
  4. Patch the ConfigMap **argocd-cmd-params-cm** to add `server.insecure : true`, then scale the deployment to 0 & then 1

     ```
     kubectl -n argocd patch configmap argocd-cmd-params-cm --type merge -p '{"data":{"server.insecure":"true"}}'
     kubectl -n argocd scale deploy argocd-server --replicas=0
     kubectl -n argocd scale deploy argocd-server --replicas=1
     ```
  4. Deploy **ArgoCD Ingress**, **Managed Certificate**, **Frontend Config** & **Backend Config** which will create an ALB listening on port 443 by running below command. Before running below command, in the **ArgoCD Managed Certificate** put the domain name you need for your ArgoCD application.

     ```
     kubectl -n argocd apply -f argocd-ingress.yml
     ```
  5. Run ` kubectl -n argocd get ingress ` to retrieve the ALB IP ( Please wait couple of minutes ). Create an A record in **Cloud DNS** or your own DNS service pointing your ArgoCD domain name to this IP.
  6. Once the DNS entry has been added it will take couple of minutes ( can be 60 minutes in some cases ) for the certificate to be generated. Run ` kubectl -n argocd get managedcertificate ` or in GCP console to check for **Active** status.
  7. Access ArgoCD using ` https://argocd_domain_name `.
  8. To get the initial admin user password run the command

     ```
     kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 --decode && echo
     ```