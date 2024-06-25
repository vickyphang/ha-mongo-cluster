# MongoDB Cluster on Kubernetes 🍃
[![GitHub license](https://img.shields.io/github/license/vickyphang/ha-mongo-cluster)](https://github.com/vickyphang/ha-mongo-cluster/blob/main/LICENSE)
![GitHub stars](https://img.shields.io/github/stars/vickyphang/ha-mongo-cluster)

## Quickstart
1. Install Community Kubernetes Operator
    ```bash
    # Add the MongoDB Helm Charts for Kubernetes repository
    helm repo add mongodb https://mongodb.github.io/helm-charts

    # Update repo
    helm repo update

    # Install on namespace 'mongodb'
    helm install community-operator mongodb/community-operator --namespace mongodb --create-namespace
    ```
2. Deploy MongoDBCommunity Resource
    - Clone mongodb-kubernetes-operator repostiory
        ```bash
        # Clone repostiory
        git clone https://github.com/mongodb/mongodb-kubernetes-operator.git

        # Change directory
        cd mongodb-kubernetes-operator/
        ```

    - Replace `<your-password-here>` in `config/samples/mongodb.com_v1_mongodbcommunity_cr.yaml` to the password you wish to use

    - Apply the manifest
        ```bash
        # Invoke the following kubectl command
        kubectl apply -f config/samples/mongodb.com_v1_mongodbcommunity_cr.yaml --namespace mongodb
        ```
    
    - Verify that the MongoDBCommunity resource deployed
        ```bash
        watch -d kubectl get all -n mongodb
        ```

    - Obtain the connection string
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
        