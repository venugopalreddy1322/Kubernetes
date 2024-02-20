For the NodePort- service demo 
--------
in the deployment yaml manifest
 ( template:
    metadata:
      labels:
        app: django-app ) 
and in the service yaml manifest 
( selector:
    app: django-app )
should be same.
commands used to run:

- $ kubectl apply -f deployment.yml
deployment.apps/nginx-deployment created

- $ kubectl apply -f service_nodeport.yml
service/my-service created

- $ kubectl get all
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

- $ minikube ssh
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)


- $ curl -L http://10.99.238.95:8000/demo
curl -L http://10.99.238.95:8000/demo
<!DOCTYPE html>
<html lang="en">
<head>
<title>CSS Template</title>
<meta charset="utf-8">
- so it is getting response....

- $ exit
exit
logout

- $ minikube ip
192.168.59.104

- $ curl -L http://192.168.59.104:30007/demo
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100  1879  100  1879    0     0  60241      0 --:--:-- --:--:-- --:--:-- 60241<!DOCTYPE html>
<html lang="en">
<head>
<title>CSS Template</title>
--- so getting response..

Try on the browser
- http://192.168.59.104:30007/demo
We got the webpage..


For Ingress:
--> enable ingress-controller in minikube
    $ minikube addons enable ingress
    confirm by the following commands
    $ kubectl get pods -n ingress-nginx
    $ kubectl get svc -n ingress-nginx
    deployment:
    $ kubectl apply -f deployment2.yml
    service:
    $ kubectl apply -f django-service.yml
    Ingress:
    $ kubectl apply -f nginx-ingress.yml
    $ kubectl get ing
    NAME            CLASS   HOSTS              ADDRESS          PORTS   AGE
    nginx-ingress   nginx   ingress-demo.com   192.168.59.106   80      35m
    -------------------------------------------------------------------------------------
    NOTE: 
                we need to update /etc/hosts file with the following entry:
                192.168.59.106 ingress-demo.com
        Also because we are using windows, need to update 
            /c/Windows/System32/drivers/etc file (one with administative mode in gitbash)
 
                        192.168.59.106 ingress-demo.com

    ------------------------------------------------------------------------------------
    Now access the ingress-demo.com on browser you will access the application.



For TLS with Ingress
step 1:
$ openssl req -x509 -newkey rsa:4096 -sha256 -nodes -keyout ca.key -out ca.crt -subj "//CN=ingress-demo.com" -days 365
step 2:
$ kubectl create secret tls ingress-demo-com-tls --cert ca.crt --key ca.key 
    secret/ingress-demo-com-tls created
    copy this secret in TLS_ingress.yml file
    then:
$ kubectl apply -f tls_ingress.yml
    ingress.networking.k8s.io/nginx-ingress created

$ kubectl get ing
NAME            CLASS   HOSTS              ADDRESS          PORTS     AGE
nginx-ingress   nginx   ingress-demo.com   192.168.59.106   80, 443   10s
Now goto browser and https://ingress-demo.com/demo/

