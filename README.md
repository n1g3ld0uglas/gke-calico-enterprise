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

The load balancer service might show an IP address of <pending>. This can take 1-2 minutes to show-up
    
```
kubectl get services -n tigera-manager tigera-manager-external
```
