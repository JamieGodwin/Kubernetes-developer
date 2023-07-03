# Configurations
- We can configure the image to slepp on startup by changing its configuration: 

![](images/2.3.png)

- The command here is to sleep on startup, and to sleep for 5000s on startup.  

### Working in a Dockerfile
- In a Dockerfile we can set an ENTRYPOINT with a command which will alway be executed on launch. All parameters that are given to docker run <image> are passed as arguments through the ENTRYPOINT. 
![](images/2.4.png)
- Here, the Dockerfile has defined an ENTRYPOINT and the commands to be passed through it.
- In the YAML file we defined a command to overide the command used in the Dockerfile. 

## Environment variable
- We can set environment variables in the following way.

![](images/2.5.png)

- Here, we use a key value pair. 

## Configure Maps
- We can create one for our key: value pairs in either an imperative or declarative way.
- They're useful for stroing and using configuration settings that containers need at runtime. 
- Imperative method:
`kubectl create configmap <config-name> --from-literal=<key>=<value>`
- Declartive:

![](images/2.6.png)

- We can then apply this with: `kubectl apply -f my-config.yaml`

### configure it in a pod
- We can now configure the map to our pod:

![](images/2.7.png)

## Secrets
They're used to store sensitive information, like passwords or keys. 
- Imperative method:
`kubectl create secret generic <name> --from-literal=<key>=<value>`
- Declaritive method:

![](images/2.8.png)

- Here however, the passwords are visible which we don't want. We need to be able to encode the passwords. Instead of writing the passwords, we writ them in an encoded form:

![](images/2.9.png)

- In order to generate the encoded passwords, we run the commands:
    - `echo -n 'mysql' | base64`
    - `echo -n 'root' | base64`
    - `echo -n 'paswrd' | base64`
- To view our created secrets, we run the command:
`kubectl get secrets`
- In order to decode, we can use the command:
`echo -n 'bX1zcWw' |base64 --decode`

- We can make the data in the secret file available in a pod as an environment variable by doing: 

![](images/3.0.png)

- Note: This isn't that safe, as it can quite easily be decoded.

# Encrypting Secret data at rest
It refers to encrypting data while it's stored on a disk. This safeguards data should the storage system be compromised. The data at rest in the ETCD can be encrypted. 

- We can first check whether encryption is already enabled with the command:
`ps -aux | grep kube-api | grep "encryption-provider-config"`

### Creating the encryption
- We first need to create a 32-byte random key:
`head -c 32 /dev/urandom | base64`
- We can then add this to an encription YAML file:

![](images/3.1.png)

- Next, we need to move the file to the local directory:
    - `mkdir /etc/kubernetes/enc`
    - `mv env.yaml /etc/kubernetes/enc`

- We next need to mainfest this into the kube-apiserver
`vim /etc/kubernetes/mainfests/kube-apiserver.yaml`
- We add the following to the file:
`--encryption-provider-config=/etc/kubernetes/enc/enc.yaml`

![](images/3.2.png)

- Under volume mounts we add:
```
- name: enc
  mountPath: /etc/kubernetes/enc
  readonly: true
```
![](images/3.3.png)

- Then under volumes we add:
```
- name: enc
  hostPath:
    path: /etc/kubernetes/enc
    type: Directory0rCreate
```
![](images/3.4.png)

- We can then chack that this has configured by running the command:
`ps aux | grep kube-api | grep encry`

- We should see the following

![](images/3.5.png)

- Now that we know that the encryption is enabled, we can test it. We can create a secret key: 
`kubectl create generic my-secret-3 --from-literal=key2=topsecret`

- We can now run the long command to check the status of the secret-key:
```
ETCDCTL_API=3 etcdctl \
   --cacert=/etc/kubernetes/pki/etcd/ca.crt  \
   --cert=/etc/kubernetes/pki/etcd/server.crt  \
   --key=/etc/kubernetes/pki/etcd/server.key  \
   get /registry/secrets/default/my-secret-2 | hexdump -C
```
- The output should look like: 

![](images/3.6.png)

- We now can't see the key, so it is encrypted. 
- The guide can be found in the official documentation: https://v1-22.docs.kubernetes.io/docs/tasks/administer-cluster/encrypt-data/ 