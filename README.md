# gke-calico-enterprise


Create a cluster with intranode visibility enabled
```
gcloud container clusters create nigel-gke-cluster /
    --zone europe-west2-a /
    --machine-type e2-standard-8 /
    --num-nodes 3 /
    --enable-intra-node-visibility /
    --no-enable-network-policy
```

<img width="1288" alt="Screenshot 2021-05-27 at 13 07 34" src="https://user-images.githubusercontent.com/82048393/119823588-ef394800-beec-11eb-80e4-a1957a6ae4b5.png">

Should take 1-2 mins to complete cluster creation:

<img width="1289" alt="Screenshot 2021-05-27 at 13 11 52" src="https://user-images.githubusercontent.com/82048393/119824087-6ec71700-beed-11eb-9557-abac609c23ed.png">


If you disable network policy enforcement, make sure to also update any add-ons
```
gcloud container clusters update nigel-gke-cluster --update-addons=NetworkPolicy=DISABLED
```

StorageClass tells Calico Enterprise to use the GCE Persistent Disks for log storage
```
kubectl apply -f https://raw.githubusercontent.com/n1g3ld0uglas/netpolTest/main/sc/gce.yaml
```

Install the Tigera operator and custom resource definitions
```
kubectl create -f https://docs.tigera.io/manifests/tigera-operator.yaml
```

<img width="1283" alt="Screenshot 2021-05-27 at 13 19 12" src="https://user-images.githubusercontent.com/82048393/119824847-40960700-beee-11eb-9d22-943417caa7a5.png">


Install the Prometheus operator and related custom resource definitionss
```
kubectl create -f https://docs.tigera.io/manifests/tigera-prometheus-operator.yaml
```

<img width="1283" alt="Screenshot 2021-05-27 at 13 21 08" src="https://user-images.githubusercontent.com/82048393/119825039-75a25980-beee-11eb-8df1-3b277e2b9964.png">


Install your pull secret - referencing the same config.json file provided to your by the team at Tigera
```
kubectl create secret generic tigera-pull-secret --type=kubernetes.io/dockerconfigjson -n tigera-operator --from-file=.dockerconfigjson=config.json
```

<img width="1292" alt="Screenshot 2021-05-27 at 13 22 34" src="https://user-images.githubusercontent.com/82048393/119825258-ad110600-beee-11eb-83a2-efc05f85bf29.png">

Install the Tigera custom resources

```
kubectl create -f https://docs.tigera.io/manifests/custom-resources.yaml
```

<img width="890" alt="Screenshot 2021-05-27 at 13 24 22" src="https://user-images.githubusercontent.com/82048393/119825582-0416db00-beef-11eb-86d1-3d7184fb0264.png">


Monitor progress with the following command.
Once the 'apiserver' is running, feel free to proceed
```
watch kubectl get tigerastatus
```

<img width="627" alt="Screenshot 2021-05-27 at 13 26 39" src="https://user-images.githubusercontent.com/82048393/119825803-40e2d200-beef-11eb-895a-36b0ca7ae60c.png">

Apply the license file - provided to you by the team at Tigera.
```
kubectl apply -f license.yaml
```

To check on the status of pods in creation, run the below command:
```
kubectl get pods -A
```

<img width="1071" alt="Screenshot 2021-05-27 at 13 28 16" src="https://user-images.githubusercontent.com/82048393/119826758-47be1480-bef0-11eb-8be4-4b9b4d91033b.png">


If you're seeing slow changes in 'watch tigerastatus', you can 'describe' a pod to see what is happening:
```
kubectl describe pod tigera-manager-79c79478cc-rsj7j -n tigera-manager
```

<img width="1299" alt="Screenshot 2021-05-27 at 13 29 51" src="https://user-images.githubusercontent.com/82048393/119826881-658b7980-bef0-11eb-8195-184dae8da2e7.png">

There was no issue identified in 'kubectl describe'.
Finally, confirm all processes are now running successfully.

```
watch kubectl get tigerastatus
```

<img width="554" alt="Screenshot 2021-05-27 at 13 30 24" src="https://user-images.githubusercontent.com/82048393/119827027-953a8180-bef0-11eb-80ce-57e5a4297b6f.png">


To secure Calico Enterprise component communications, install this set of network policies

```
kubectl create -f https://docs.tigera.io/manifests/tigera-policies.yaml
```

<img width="866" alt="Screenshot 2021-05-27 at 13 37 33" src="https://user-images.githubusercontent.com/82048393/119827293-e185c180-bef0-11eb-9fd8-0fea3c2b9912.png">

To expose the manager using a load balancer, create the following service
```
kubectl apply -f https://raw.githubusercontent.com/n1g3ld0uglas/netpolTest/main/sc/lb.yaml
```

The load balancer service might show an IP address of pending. 
This can take 1-2 minutes to show-up
    
```
kubectl get services -n tigera-manager tigera-manager-external
```


<img width="1023" alt="Screenshot 2021-05-27 at 13 43 45" src="https://user-images.githubusercontent.com/82048393/119828653-48f04100-bef2-11eb-9dd3-a0a534e1fed0.png">

This will the PUBLIC-IP:PORT(9443) will only get us as far as a login page.
However, to proceeed we will need to create our service account for login.

<img width="649" alt="119829212-e9defc00-bef2-11eb-83b0-17953114a7da" src="https://user-images.githubusercontent.com/82048393/119838083-07b05f00-befb-11eb-8f7e-b128833bd952.png">


First, create a service account in the desired namespace    
```
kubectl create sa nigel -n default    
```    

Give the default SA 'Nigel' network admin permissions
```
kubectl create clusterrolebinding nigel-access --clusterrole tigera-network-admin --serviceaccount default:nigel   
``` 

Get Base64 encoded token for SA account (Nigel)
```
kubectl get secret $(kubectl get serviceaccount nigel -o jsonpath='{range .secrets[*]}{.name}{"\n"}{end}' | grep token) -o go-template='{{.data.token | base64decode}}' && echo    
```

<img width="1297" alt="Screenshot 2021-05-27 at 13 55 32" src="https://user-images.githubusercontent.com/82048393/119829692-6a056180-bef3-11eb-903d-df7012d51362.png">

Paste the base64 encoded output token in the Calico Enterprise login window:

<img width="519" alt="Screenshot 2021-05-27 at 13 57 41" src="https://user-images.githubusercontent.com/82048393/119829994-ba7cbf00-bef3-11eb-816d-204dfc835396.png">

Since you installed a bunch of policies earlier, you should see those under the 'allow-tigera' tier.
We will come back to the concept of tiers shortly.

<img width="622" alt="Screenshot 2021-05-27 at 14 00 26" src="https://user-images.githubusercontent.com/82048393/119830378-1e06ec80-bef4-11eb-9f01-7083532b97a7.png">


Connect to Kibana with the 'elastic' username
You can access this via your PUBLIC:IP/PORT(9443)
```
kubectl -n tigera-elasticsearch get secret tigera-secure-es-elastic-user -o go-template='{{.data.elastic | base64decode}}' && echo   
``` 

<img width="501" alt="Screenshot 2021-05-27 at 15 48 18" src="https://user-images.githubusercontent.com/82048393/119847657-113dc500-bf03-11eb-8ee7-058554e5e3bf.png">


If you want to temporarily shutdown your GKE cluster (to reduce costs), you can resize the cluster to 0 nodes:
This can take 2-3 minutes to complete the cluster rescaling.
```
gcloud container clusters resize nigel-gke-cluster --zone=europe-west2-a --num-nodes=0
```

<img width="1300" alt="Screenshot 2021-05-27 at 15 53 45" src="https://user-images.githubusercontent.com/82048393/119848834-0b94af00-bf04-11eb-9f0f-ab751d297451.png">


Similarly, when you need to work on this test cluster again, you can re-scale the cluster back to the 3 nodes:
```
gcloud container clusters resize nigel-gke-cluster --zone=europe-west2-a --num-nodes=3
```

## Honeypods
```
kubectl apply -f https://docs.tigera.io/manifests/threatdef/honeypod/common.yaml 
```
```
kubectl create secret generic tigera-pull-secret --from-file=.dockerconfigjson=config.json --type=kubernetes.io/dockerconfigjson -n tigera-internal
```
```
kubectl apply -f https://docs.tigera.io/manifests/threatdef/honeypod/ip-enum.yaml 
```
```
kubectl apply -f https://docs.tigera.io/manifests/threatdef/honeypod/expose-svc.yaml 
```
```
kubectl apply -f https://docs.tigera.io/manifests/threatdef/honeypod/vuln-svc.yaml 
```

![Screenshot 2021-12-06 at 10 58 40](https://user-images.githubusercontent.com/82048393/144834860-f5ec5d0a-3a47-439f-9bb5-438326e78669.png)



```
kubectl get pods -n tigera-internal
```
```
kubectl get globalalerts
```
```
kubectl delete secret tigera-pull-secret -n tigera-internal
```



![Screenshot 2021-12-06 at 11 01 08](https://user-images.githubusercontent.com/82048393/144835168-86485383-7d82-4b0a-8cd7-8e1b16c783f3.png)



## Trigger Honeypod Alerts

Assume you have a real-world microservice application called ```storefront``` (if not, add it to your cluster)
```
kubectl apply -f https://installer.calicocloud.io/storefront-demo.yaml
```

Once created, create the ```attacker-app``` pod in the ```storefront``` namespace
```
kubectl create -f https://installer.calicocloud.io/rogue-demo.yaml -n storefront
```

This will attempt to probe as many pods as possible - including the ```tigera-internal``` namespaced pods 

![Screenshot 2021-12-06 at 11 11 07](https://user-images.githubusercontent.com/82048393/144836394-6cdfbbbc-97fd-4110-ae4b-eb418b9f2141.png)



Confirm the new attacker app exists within the Storefront namespace:
```
kubectl get storefront -A
```

This should be visible in our Service Graph:

![Screenshot 2021-12-06 at 11 14 27](https://user-images.githubusercontent.com/82048393/144836841-df7cf026-7c77-4b0c-bc4b-4a50291a30db.png)

Especially when you drill-down to the tigera-internal namespace:

![Screenshot 2021-12-06 at 11 15 27](https://user-images.githubusercontent.com/82048393/144836882-de1a4d4c-d309-4442-9298-98042dcb1352.png)

Once you've completed your tests, proceed to remove the attacker app deployment:

```
kubectl delete -f https://installer.calicocloud.io/rogue-demo.yaml -n storefront
```


## Miscellaneous Junk

Ignore fixed dependency issues
```
kubectl annotate ds -n calico-system calico-node unsupported.operator.tigera.io/ignore="true"
```

On your local Linux or macOS computer, you can use the ssh-keygen command to retrieve the public key for your key pair.
```
ssh-keygen -y -f nigel-rancher-key.pem
```

The command returns the public key, as shown in the following example.
```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQClKsfkNkuSevGj3eYhCe53pcjqP3maAhDFcvBS7O6V
hz2ItxCih+PnDSUaw+WNQn/mZphTk/a/gU8jEzoOWbkM4yxyb/wB96xbiFveSFJuOp/d6RJhJOI0iBXr
lsLnBItntckiJ7FbtxJMXLvvwJryDUilBMTjYtwB+QhYXUMOzce5Pjz5/i8SeJtjnV3iAoG/cQk+0FzZ
qaeJAAHco+CY/5WrUBkrHmFJr6HcXkvJdWPkYQS3xqC0+FmUZofz221CBt5IMucxXPkX4rWi+z7wB3Rb
BQoQzd8v7yeb7OzlPnWOyN0qFU0XA246RA8QFYiCNYwI3f05p6KLxEXAMPLE
```
