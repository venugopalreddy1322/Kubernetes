----------------------------------------------------------------
cm.yml, deployment.yml and deployment-mount.yml
-----------------------------------------------
With the help of above mentioned files demonstated the use of values in cm.yml() ConfigMap) are used in deployment.yml file using Env-variables.
With this approach of using environment-variables, changes in cm.yml files are not reflects in deployment.yml until it is restarted.
So to overcome this we used Voume-Mounts in deployment-mount.yml
--------------------------------------------------------------

----------------------------------------------------------------
                    SECRETS
---------------------------------------------------------------
create secrets using kubectl:

Use raw data
Run the following command:

kubectl create secret generic db-user-pass \
    --from-literal=username=admin \
    --from-literal=password='S!B\*d$zDsb='    
    # here password is encoded

 Use source files
Store the credentials in files:

echo -n 'admin' > ./username.txt
echo -n 'S!B\*d$zDsb=' > ./password.txt 

Pass the file paths in the kubectl command:

kubectl create secret generic db-user-pass \
    --from-file=./username.txt \
    --from-file=./password.txt

The default key name is the file name. You can optionally set the key name using --from-file=[key=]source. For example:

kubectl create secret generic db-user-pass \
    --from-file=username=./username.txt \
    --from-file=password=./password.txt

Verify the Secret
Check that the Secret was created:

kubectl get secrets

View the details of the Secret:

kubectl describe secret db-user-pass

Decode the Secret
View the contents of the Secret you created:

kubectl get secret db-user-pass -o jsonpath='{.data}'
The output is similar to:

{ "password": "UyFCXCpkJHpEc2I9", "username": "YWRtaW4=" }
Decode the password data:

echo 'UyFCXCpkJHpEc2I9' | base64 --decode
The output is similar to:

S!B\*d$zDsb=

IN ONE STEP:

kubectl get secret db-user-pass -o jsonpath='{.data.password}' | base64 --decode


----------------------------------------------
deployment-secret-mount.yml and secret.yml
-----------------------------------------------

The secret.yml file(manifest) contains the secrets like username and password which are encripted using base64.
echo -n 'admin' | base64 -- the resulted value is used in data: username:  
deployment-secret-mount.yml file reads the secrets from secret.yml file using mount volume.
steps:
1) kubectl apply -f secret.yml
-- kubectl get secrets
-- kubectl describe secrets <secret-deploy/secret-name>
2) kubectl apply -f deployment-secret-mount.yml
3) kubectl get pods
4) exec into one of the pods
-- kubectl exec -it <pod-name> -- bash
-- ls /opt/db-secrets     --mountpoint location
-- cat /opt/db-secrets/username
-- will get the decoded username -- because we used the encoded values of username and password in secret.yml file which again converted these using base64.
