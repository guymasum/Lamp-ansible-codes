
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

<rawtext>---</awtext>
apiVersion: awx.ansible.com/v1beta1
kind: AWX
metadata:
  name: awx
spec:
  <rawtext># These parameters are designed for use with:</rawtext>
  <rawtext># - AWX Operator: 2.17.0</rawtext>
  <rawtext>#   https://github.com/ansible/awx-operator/blob/2.17.0/README.md</rawtext>
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
</blockquote>
