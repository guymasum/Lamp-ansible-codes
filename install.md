
copy tls.crt and tls.key from to 
``` 
openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -out tls.crt -keyout tls.key -subj "/CN=${AWX_HOST}/O=${AWX_HOST}" -addext "subjectAltName = DNS:${AWX_HOST}"
```
Create the tls secret for awx-dev.imf.org
```
kubectl create secret tls awx-secret-tls --cert=tls.crt  --key=tls.key 
```
Generate and create the awx admin password 
```
kubectl create secret generic awx-admin-password --from-literal=password=$(openssl rand -hex 15) -n awx
```
To view the encoded data of the awx admin password,type this command <p>
```
kubectl get secret awx-admin-password \
 -o jsonpath='{.data}' -n awx 
```
<p>To decode the encoded data of the awx admin password,type this command
```
<pre>echo </rawtext><encoded-string><rawtext>"</rawtext> | base64 --decode</pre>
```
Create the secret for accessing Azure ACR <p>
```
kubectl create secret docker-registry  \
    --namespace awx \
    --docker-server=<container-registry-name>.azurecr.io \
    --docker-username=<service-principal-ID> \
    --docker-password=<service-principal-password>
```








```yaml
--- 
apiVersion: awx.ansible.com/v1beta1 
kind: AWX
metadata:
  name: awx
spec:
  # These parameters are designed for use with:
  # - AWX Operator: 2.17.0 
  #  https://github.com/ansible/awx-operator/blob/2.17.0/README.md 
    image: <container-registry-name>.azurecr.io/awx
    image_version: latest
    image_pull_policy: Always
    image_pull_secrets:
     - acr_pull_secret
    ee_images:
      - name: awx-ee
        image: <container-registry-name>.azurecr.io/awx-ee
    control_plane_ee_image: <container-registry-name>.azurecr.io/awx-ee:latest
    init_container_image: <container-registry-name>.azurecr.io/awx-ee
    init_container_image_version: latest
    init_projects_container_image: <container-registry-name>.azurecr.io/centos:stream9

    admin_user: admin
    admin_password_secret: awx-admin-password

    ingress_type: ingress
    ingress_hosts:
      - hostname: awx.example.com
        tls_secret: awx-secret-tls

    postgres_configuration_secret: awx-postgres-configuration

    web_replicas: 3
    task_replicas: 3
```	
