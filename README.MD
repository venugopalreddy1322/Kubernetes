## NodePort Service Demo

This repository contains YAML manifests and commands to demonstrate NodePort service in Kubernetes.

### Prerequisites
- Kubernetes cluster (e.g., Minikube)
- kubectl installed
- curl installed

### Deployment Steps

1. Apply the deployment YAML manifest:
    ```shell
    $ kubectl apply -f deployment.yml
    ```
    This creates the nginx-deployment.

2. Apply the service YAML manifest:
    ```shell
    $ kubectl apply -f service_nodeport.yml
    ```
    This creates the my-service NodePort service.

3. Verify deployment:
    ```shell
    $ kubectl get all
    ```
###output:
  NAME                                    READY   STATUS    RESTARTS   AGE
pod/nginx-deployment-74895669fb-pg4kh   1/1     Running   0          116s
pod/nginx-deployment-74895669fb-v45m6   1/1     Running   0          116s

NAME                 TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
service/kubernetes   ClusterIP   10.96.0.1      <none>        443/TCP          3h57m
service/my-service   NodePort    10.99.238.95   <none>        8000:30007/TCP   100s

NAME                               READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-deployment   2/2     2            2           116s

NAME                                          DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-deployment-74895669fb   2         2         2       116s
    
    Ensure that the deployment and service are running correctly.

### Accessing the Demo

#### Using Minikube IP

1. SSH into Minikube:
    ```shell
    $ minikube ssh
    ```
2. Test the service using curl:
    ```shell
    $ curl -L http://<minikube_ip>:30007/demo
    ```

3. You should receive a response indicating successful deployment.

4. Exit Minikube:
    ```shell
    $ exit
    ```

#### Using Ingress

1. Enable Ingress controller in Minikube:
    ```shell
    $ minikube addons enable ingress
    ```

2. Confirm Ingress controller is running:
    ```shell
    $ kubectl get pods -n ingress-nginx
    $ kubectl get svc -n ingress-nginx
    ```

3. Apply deployment and service YAMLs for Ingress:
    ```shell
    $ kubectl apply -f deployment2.yml
    $ kubectl apply -f django-service.yml
    ```

4. Apply Ingress YAML:
    ```shell
    $ kubectl apply -f nginx-ingress.yml
    ```

5. Verify Ingress:
    ```shell
    $ kubectl get ing
    ```

6. Update hosts file:
    - On Linux/Mac:
        ```shell
        $ sudo nano /etc/hosts
        ```
    - On Windows:
        Update `:/c/Windows/System32/drivers/etc/hosts` with admin privileges.

    Add:
    ```
    <minikube_ip> ingress-demo.com
    ```

7. Access the application in the browser using the URL: [http://ingress-demo.com/demo/](http://ingress-demo.com/demo/)

### TLS with Ingress

1. Generate TLS certificate:
    ```shell
    $ openssl req -x509 -newkey rsa:4096 -sha256 -nodes -keyout ca.key -out ca.crt -subj "//CN=ingress-demo.com" -days 365
    ```

2. Create TLS secret:
    ```shell
    $ kubectl create secret tls ingress-demo-com-tls --cert ca.crt --key ca.key
    ```
    secret/ingress-demo-com-tls created
    copy this secret in TLS_ingress.yml file

3. Apply TLS Ingress YAML:
    ```shell
    $ kubectl apply -f tls_ingress.yml
    ```
    ingress.networking.k8s.io/nginx-ingress created

4. Verify Ingress:
    ```shell
    $ kubectl get ing
    ```
    NAME            CLASS   HOSTS              ADDRESS          PORTS     AGE
nginx-ingress   nginx   ingress-demo.com   192.168.59.106   80, 443   10s

5. Access the application securely in the browser using the URL: [https://ingress-demo.com/demo/](https://ingress-demo.com/demo/)
another example:
URL: [https://django-demo.com/demo/](https://django-demo.com/demo/)

![django-demo.com](https://github.com/venugopalreddy1322/project-diagrams/blob/main/kubernetes_pics/Screenshot%20(71).png)

### Note

- Ensure labels in deployment and service manifests match for proper service discovery.
- Update the hosts file with the appropriate IP address for Ingress to work correctly.
- For Windows users, modify `C:\Windows\System32\drivers\etc\hosts` with administrative privileges.

## RBAC Demo
- To Authenticate a user

1. Generate user's private key with openssl:
    ```shell
    $ openssl genrsa -out venu.key 2048
    ```

2. Create a certificate signing request for user with the above key:
    ```shell
    $ openssl req -new -key venu.key -out venu.csr -subj "/CN=venu/O=dev/O=example.org"
    ```
    or

    ```shell
    $ openssl req -new -key venu.key -out venu.csr

    ```
### Note:
You are about to be asked to enter information that will be incorporated into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.

Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:dev,example.org
Organizational Unit Name (eg, section) []:dev,prod
Common Name (e.g. server FQDN or YOUR name) []:venu
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:

3. Now this certificate signing request must be signed by the Certificate Authority:
    - can get the Certificate Authority file details in ~/.kube/config file

    ```shell
    $ openssl x509 -req -CA ~/.minikube/ca.crt -CAkey ~/.minikube/ca.key -CAcreateserial -days 730 -in venu.csr -out venu.crt
    ```
Certificate request self-signature ok
subject=C=AU, ST=Some-State, O=dev,example.org, OU=dev,prod, CN=venu


4. Add the user to the cluster:
    ```shell
    $ kubectl config set-credentials venu --client-certificate=/c/Users/venuc/.minikube/profiles/minikube/venu.crt --client-key=/c/Users/venuc/.minikube/profiles/minikube/venu.key --kubeconfig=/c/Users/venuc/.kube/config
    ```
   User "venu" set.

5.  Create venu-context:

    ```shell
    $ kubectl config set-context venu-minikube --cluster=minikube --user=venu --namespace=default
    ```
    Context "venu-minikube" created.

    to check the contexts and switch between:

    ```shell
    $ kubectl config get-contexts
    ```
    CURRENT   NAME            CLUSTER    AUTHINFO   NAMESPACE
              minikube        minikube   minikube   default
      *       venu-minikube   minikube   venu       default

    ```shell
    $ kubectl config use-context venu-minikube
    
    ```
    Switched to context "venu-minikube".

    ```shell
    $ kubectl get pods

    ```
    Error from server (Forbidden): pods is forbidden: User "venu" cannot list resource "pods" in API group "" in the namespace "default"

    Here comes Roles, cluster roles and Role-bindings, clusterRoleBindings
    - Here ane some examples:

    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
    # "namespace" omitted since ClusterRoles are not namespaced
        name: demo-cluster-role
    rules:
    - apiGroups: ["rbac.authorization.k8s.io"]
      resources: ["roles"]
      verbs: ["get", "watch", "list"]
  
    - apiGroups: [""]
      resources: ["pods"]
      verbs: ["get", "watch", "list"]
    ```

```yaml 
apiVersion: rbac.authorization.k8s.io/v1
# This cluster role binding allows anyone in the "manager" group to read secrets in any namespace.
kind: ClusterRoleBinding
metadata:
  
  name: demo-cluster-role-binding
subjects:
- kind: User
  name: venu # Name is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: demo-cluster-role
  apiGroup: rbac.authorization.k8s.io
```
- Now venu user can get pods and roles in any Namespace across Cluster.

    ```shell
    $ kubectl get pods
    ```
    No resources found in default namespace.


# Kubernetes Service Accounts

In Kubernetes, a Service Account is an identity used by Pods to interact with the Kubernetes API server. It is similar to a user account in an operating system. Every Pod that runs in a Kubernetes cluster has a Service Account associated with it. Service Accounts are used to control access and permissions for Pods within the cluster.

## Key Points about Service Accounts

### Automatically Created
- When you create a Pod, Kubernetes automatically creates a Service Account for that Pod if one is not specified. 
- The Service Account is mounted into the Pod's filesystem, allowing the Pod to access the Kubernetes API server.

### Access Control
- Service Accounts are used to authenticate Pods to the Kubernetes API server and determine the permissions granted to them. 
- They are bound to roles or cluster roles using RoleBindings or ClusterRoleBindings, which specify the permissions that Pods associated with the Service Account have.

### Default Service Account
- Every namespace in a Kubernetes cluster has a default Service Account named `default`. 
- If a Pod does not specify a Service Account, it uses the default Service Account by default.

### Custom Service Accounts
- You can create custom Service Accounts in Kubernetes and assign them to Pods as needed. 
- This allows you to have fine-grained control over the permissions of Pods within your cluster.

### Security Context
- Service Accounts also have associated Security Contexts, which define the security settings for Pods associated with the Service Account. 
- This includes settings such as the user and group IDs under which the processes in the Pod run, as well as filesystem permissions.

Overall, Service Accounts are an essential component of Kubernetes' security model, providing identity and access management capabilities for Pods running within the cluster. They help ensure that Pods can only perform actions they are authorized to do and help enforce the principle of least privilege.

Example:

```yaml 
apiVersion: rbac.authorization.k8s.io/v1
# This cluster role binding allows all resources like pods in the "test-sa" ServiceAccount # in default namespace read pods and roles as specified in demo-cluster-role.
kind: ClusterRoleBinding
metadata:
  
  name: demo-cluster-role-binding
subjects:
- kind: ServiceAccount
  name: test-sa 
  nameSpace: default
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: demo-cluster-role
  apiGroup: rbac.authorization.k8s.io
```
- To check the authority a simple command is
```shell
 $ kubectl auth can-i get pods --as="system:serviceaccount:default:test-sa"

```
    yes





    
Enjoy exploring the demos! 🚀
