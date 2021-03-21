# Deploy Mediawiki application in Kubernetes cluster through Kustomize

Kustomize: kubernetes native configuration management

# Prerequisites
- Kubernetes cluster 
- Istio 
- Kustomize

# Structure
```
├── base
│   ├── kustomization.yaml
│   ├── mediawiki-application
│   │   ├── configmap.yaml
│   │   ├── deployment.yaml
│   │   ├── kustomization.yaml
│   │   ├── pvc.yaml
│   │   └── service.yaml
│   ├── mediawiki-database
│   │   ├── kustomization.yaml
│   │   ├── mysql-deployment.yaml
│   │   ├── mysql-pvc.yaml
│   │   ├── mysql-secret.yaml
│   │   └── mysql-services.yaml
│   ├── mediawiki-istio-ingress
│   │   ├── gateway.yaml
│   │   ├── kustomization.yaml
│   │   └── vs.yaml
│   └── mediawiki-storage
│       ├── kustomization.yaml
│       └── mediawiki-storage-class.yaml
├── overlay
│   ├── dev
│   │   ├── kustomization.yaml
│   │   ├── mediawiki-application
│   │   │   └── deployment.yaml
│   │   └── namespace.yaml
│   ├── prod
│   │   ├── kustomization.yaml
│   │   ├── mediawiki-application
│   │   │   └── deployment.yaml
│   │   └── namespace.yaml
│   └── qa
│       ├── kustomization.yaml
│       ├── mediawiki-application
│       │   └── deployment.yaml
│       └── namespace.yaml
└── readme.md
```
- In base folder we define common resources.
- In overlays folders we define environment specific patches.

# Quick Start
Below are instructions to quickly install and deploy MEDIAWIKI application. Currently, only Kubernetes environment is supported.

# Note: 
  - Inorder to simplify the application running on your side I have committed mysql passwords in the yaml file. This is not a recommended approach.


# Deploy the application

- Verify Kubernetes version:
  ```
    kubectl version
  ```

- Verify that the istio pods and service are created with the following command:
  ```
    $ kubectl get svc,pods -n istio-system
  ```
  ```
    output
    NAME                                        READY   STATUS    RESTARTS   AGE
    pod/istio-ingressgateway-6fddb98db8-bjnn5   1/1     Running   0          8d
    pod/istiod-65c9c675d4-fgnm9                 1/1     Running   0          8d

    NAME                           TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)                                                                      AGE
    service/istio-ingressgateway   LoadBalancer   10.2.0.202   X.X.X.X   15021:31033/TCP,80:30175/TCP,443:32752/TCP,15012:31622/TCP,15443:30799/TCP   8d
    service/istiod                 ClusterIP      10.2.0.36    <none>        15010/TCP,15012/TCP,443/TCP,15014/TCP                                        8d
  ```

- Dry-run deployment in dev environment:
  ```sh
    $ kustomize build overlay/dev
  ```

- Deploy the application in the dev environment:
  ```sh
    $ kustomize build overlay/dev | kubectl apply -f -

    output:
    namespace/dev created
    storageclass.storage.k8s.io/dev-managed-premium-retain created
    configmap/dev-mediawiki-application-2mdttf67dh created
    secret/dev-secret-mediawiki created
    service/dev-mediawiki-svc created
    service/dev-mysql-svc created
    persistentvolumeclaim/dev-mediawiki-pvc created
    persistentvolumeclaim/dev-mysql-pvc created
    deployment.apps/dev-mediawiki-deployment created
    deployment.apps/dev-mysql-mediawiki created
  ```

- Confirm that all application pods and services are deployed:
  ```sh
    $ kubectl get svc,pod -n dev
  ```
  ```
    output:
      NAME                        TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
      service/dev-mediawiki-svc   ClusterIP   10.2.0.192   <none>        80/TCP     26m
      service/dev-mysql-svc       ClusterIP   None         <none>        3306/TCP   26m

      NAME                                            READY   STATUS    RESTARTS   AGE
      pod/dev-mediawiki-deployment-74d6784855-qsrdx   1/1     Running   5          26m
      pod/dev-mysql-mediawiki-64ccbfd7bd-pkd7b        1/1     Running   0          19m  
  ```

- Create a Gateway and Virtual Service to expose and route traffic to the Mediawiki Service:
  ```
    $ kubectl apply -f base/mediawiki-istio-ingress/gateway.yaml
    $ kubectl apply -f base/mediawiki-istio-ingress/vs.yaml
  ```
  ```
    NAME           GATEWAYS                           HOSTS   AGE
    mediawiki-vs   [istio-system/mediawiki-gateway]   [*]     11m
  ```
- Get the public IP of the Istio Ingress controller. If the cluster is running in an environment that supports external load balancers:
  ```
    $ kubectl get svc -n istio-system | grep -E 'EXTERNAL-IP|istio-ingress'
  ```
- Open the Mediawiki application in a browser using the following link:
  ```
    http://<Public-IP-of-the-Ingress-Controller>/wiki/Main_Page
  ```
  


