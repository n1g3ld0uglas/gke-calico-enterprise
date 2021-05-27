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



