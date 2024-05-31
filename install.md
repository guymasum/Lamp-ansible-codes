
copy tls.crt and tls.key from to 
 
openssl req -x509 -nodes -days 3650 -newkey rsa:2048 -out tls.crt -keyout tls.key -subj "/CN=${AWX_HOST}/O=${AWX_HOST}" -addext "subjectAltName = DNS:${AWX_HOST}"

Create the tls secret for awx-dev.imf.org
<blockquote>
kubectl create secret tls awx-secret-tls --cert=tls.crt  --key=tls.key 
</blockquote>
Create the secret for accessing Azure ACR <p>
<blockquote>
kubectl create secret docker-registry  \<br>
    --namespace awx \<br>
    --docker-server=<container-registry-name>.azurecr.io \<br>
    --docker-username=<service-principal-ID> \<br>
    --docker-password=<service-principal-password><br>
</blockquote>
<p>
Generate and create the awx admin password <br>
<blockquote>
kubectl create secret generic awx-admin-password --from-literal=password=$(openssl rand -hex 5) -n awx
</blockquote>
<p>
To view the encoded data of the awx admin password,type this command
<blockquote>
kubectl get secret awx-admin-password1 -o jsonpath='{.data}' -n awx <br>
{"password":"NGQ4MmY5NmJjNjIzOWI0MzhlNGQ4MzlhOGUzZjFiMjg4ODI4OTIyOA=="}
</blockquote>
To decode the encoded data of the awx admin password,type this command
<blockquote>
echo "<encoded-string>" | base64 --decode
</blockquote>







<blockquote>

<rawtext>---</awtext><br>
apiVersion: awx.ansible.com/v1beta1<br>
kind: AWX<br>
metadata:<br>
  name: awx<br>
spec:<br>
  <rawtext>#</rawtext>These parameters are designed for use with:<br>
  <rawtext>#</rawtext> - AWX Operator: 2.17.0<br> 
  <rawtext>#</rawtext>  https://github.com/ansible/awx-operator/blob/2.17.0/README.md<br> 
    image: <container-registry-name>.azurecr.io/awx<br>
    image_version: latest<br>
    image_pull_policy: Always<br>
    image_pull_secrets:<br>
     - acr_pull_secret<br>
    ee_images:<br>
      - name: awx-ee<br>
        image: <container-registry-name>.azurecr.io/awx-ee<br>
    control_plane_ee_image: <container-registry-name>.azurecr.io/awx-ee:latest<br>
    init_container_image: <container-registry-name>.azurecr.io/awx-ee<br>
    init_container_image_version: latest<br>
    init_projects_container_image: <container-registry-name>.azurecr.io/centos:stream9<br>

    admin_user: admin
    admin_password_secret: awx-admin-password

    ingress_type: ingress
    ingress_hosts:
      - hostname: awx.example.com
        tls_secret: awx-secret-tls

    postgres_configuration_secret: awx-postgres-configuration

    web_replicas: 3
    task_replicas: 3	
</blockquote>
