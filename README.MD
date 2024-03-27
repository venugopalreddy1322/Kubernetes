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
    ##output:
  
                             _             _
                _         _ ( )           ( )
      ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
    /' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
    | ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
    (_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)               




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
        Update `C:\Windows\System32\drivers\etc\hosts` with admin privileges.

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

### Note

- Ensure labels in deployment and service manifests match for proper service discovery.
- Update the hosts file with the appropriate IP address for Ingress to work correctly.
- For Windows users, modify `C:\Windows\System32\drivers\etc\hosts` with administrative privileges.

Enjoy exploring the demo! 🚀
