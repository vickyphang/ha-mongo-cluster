# MongoDB Cluster on Kubernetes 🍃
[![GitHub license](https://img.shields.io/github/license/vickyphang/ha-mongo-cluster)](https://github.com/vickyphang/ha-mongo-cluster/blob/main/LICENSE)
![GitHub stars](https://img.shields.io/github/stars/vickyphang/ha-mongo-cluster)

The **MongoDB Community Kubernetes Operator** is a Custom Resource Definition and a controller.

### Prerequisites:
- Kubernetes cluster
- Kubectl
- Helm
- Dynamic volume provisioning

## 1. Install the Operator
- Add the MongoDB Helm Charts for Kubernetes repository
    ```bash
    helm repo add mongodb https://mongodb.github.io/helm-charts
    ```
- Update helm repo
    ```bash
    helm repo update
    ```
- By default, this Operator is using cluster name `cluster.local`, so we want to make sure that we use the correct cluster name or domain. Create a new file `values.yaml`. Add extra env `CLUSTER_DOMAIN` inside `values.yaml`
    ```yaml
    operator:
      extraEnvs:
        - name: CLUSTER_DOMAIN
          value: cluster.local
    ```
- Finally we install the Operator on namespace 'mongodb'
    ```bash
    helm install community-operator mongodb/community-operator --namespace mongodb --create-namespace -f values.yaml
    ```
- Wait untill all resources are running
    ```bash
    # List all resource in ns mongodb
    kubectl get all -n mongodb

    # Output
    NAME                                               READY   STATUS    RESTARTS   AGE
    pod/mongodb-kubernetes-operator-77cd5c7c66-xwhx4   1/1     Running   0          39m

    NAME                                          READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/mongodb-kubernetes-operator   1/1     1            1           49m

    NAME                                                     DESIRED   CURRENT   READY   AGE
    replicaset.apps/mongodb-kubernetes-operator-77cd5c7c66   1         1         1       39m
    ```


## 2. Deploy MongoDB Replica Set Cluster
- Clone mongodb github repository
    ```bash
    git clone https://github.com/mongodb/mongodb-kubernetes-operator.git
    ```
- Change directory into `mongodb-kubernetes-operator`
    ```bash
    cd mongodb-kubernetes-operator/
    ```
- Replace `<your-password-here>` in `config/samples/mongodb.com_v1_mongodbcommunity_cr.yaml` to the password you wish to use
    ```yaml
    [...]
    ---
    apiVersion: v1
    kind: Secret
    metadata:
      name: my-user-password
    type: Opaque
    stringData:
      password: <your-password-here>
    ```
- Apply the manifest
    ```bash
    kubectl apply -f config/samples/mongodb.com_v1_mongodbcommunity_cr.yaml --namespace mongodb
    ```
- Verify that the MongoDBCommunity resource deployed
    ```bash
    # List all resources in ns mongodb
    watch -d kubectl get all -n mongodb

    # Output
    NAME                                               READY   STATUS    RESTARTS   AGE
    pod/example-mongodb-0                              2/2     Running   0          49m
    pod/example-mongodb-1                              2/2     Running   0          49m
    pod/example-mongodb-2                              2/2     Running   0          48m
    pod/mongodb-kubernetes-operator-77cd5c7c66-xwhx4   1/1     Running   0          50m

    NAME                          TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)     AGE
    service/example-mongodb-svc   ClusterIP   None         <none>        27017/TCP   49m

    NAME                                          READY   UP-TO-DATE   AVAILABLE   AGE
    deployment.apps/mongodb-kubernetes-operator   1/1     1            1           60m

    NAME                                                     DESIRED   CURRENT   READY   AGE
    replicaset.apps/mongodb-kubernetes-operator-77cd5c7c66   1         1         1       50m

    NAME                                   READY   AGE
    statefulset.apps/example-mongodb       3/3     49m
    statefulset.apps/example-mongodb-arb   0/0     49m
    ```
- Obtain the MongoDB connection string
    > The following command requires jq version 1.6 or higher.
    ```bash
    # Invoke the following kubectl command
    kubectl get secret example-mongodb-admin-my-user -n mongodb -o json | jq -r '.data | with_entries(.value |= @base64d)'

    # Output
    {
        "connectionString.standard": "mongodb://<username>:<password>@example-mongodb-0.example-mongodb-svc.mongodb.svc.cluster.local:27017,example-mongodb-1.example-mongodb-svc.mongodb.svc.cluster.local:27017,example-mongodb-2.example-mongodb-svc.mongodb.svc.cluster.local:27017/admin?ssl=true",
        "connectionString.standardSrv": "mongodb+srv://<username>:<password>@example-mongodb-svc.mongodb.svc.cluster.local/admin?ssl=true",
        "password": "<password>",
        "username": "<username>"
    }
    ```
- Try connecting to your MongoDB cluster with `connectionString.standard` above using `mongosh` from `control-plane` node. If mongosh not installed on your machine, follow [this](https://www.mongodb.com/docs/mongodb-shell/install/).
    ```bash
    mongosh "mongodb://<username>:<password>@example-mongodb-0.example-mongodb-svc.mongodb.svc.cluster.local:27017,example-mongodb-1.example-mongodb-svc.mongodb.svc.cluster.local:27017,example-mongodb-2.example-mongodb-svc.mongodb.svc.cluster.local:27017/admin?replicaSet=example-mongodb&ssl=false"

    # Output
    Current Mongosh Log ID: 667a7b5e10ee02719a149f47
    Connecting to:          mongodb://<credentials>@example-mongodb-0.example-mongodb-svc.mongodb.svc.cluter.simple:27017,example-mongodb-1.example-mongodb-svc.mongodb.svc.cluter.simple:27017,example-mongodb-2.example-mongodb-svc.mongodb.svc.cluter.simple:27017/admin?replicaSet=example-mongodb&ssl=false&appName=mongosh+2.2.10
    Using MongoDB:          6.0.5
    Using Mongosh:          2.2.10

    For mongosh info see: https://docs.mongodb.com/mongodb-shell/

    ------
    The server generated these startup warnings when booting
    2024-06-25T07:11:13.373+00:00: Using the XFS filesystem is strongly recommended with the WiredTiger storage engine. See http://dochub.mongodb.org/core/prodnotes-filesystem
    2024-06-25T07:11:17.034+00:00: vm.max_map_count is too low
    ------

    example-mongodb [primary] admin>
    ```