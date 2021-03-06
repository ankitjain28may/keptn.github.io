---
title: Helm
description: Explains how Keptn uses Helm to deploy services
weight: 95
---

## Intro

During `keptn install`, we are installing Tiller (Helm v2.12.3) on the Kubernetes cluster. Keptn then uses Helm for
 deploying onboarded services to a Kubernetes cluster. This is currently implemented in Keptn's
 [helm-service](https://github.com/keptn/keptn/tree/0.6.2/helm-service).

### Onboarding

Based on the [onboarding the carts service tutorial](../../usecases/onboard-carts-service) we will describe the two
 different deployment types that the helm-service manages and what this does on Kubernetes level.
 
Those deployments are based on the provided helm-chart for the carts microservice, see https://github.com/keptn/examples/tree/0.6.2/onboarding-carts/carts
for details.

### Direct deployments

In case of `deployment_strategy: direct` (see 
 [shipyard.yaml](https://github.com/keptn/examples/blob/0.6.2/onboarding-carts/shipyard.yaml)), Helm deploys a 
 release called `sockshop-dev-carts` as `carts` in namespace `sockshop-dev`.
 
```console
$ kubectl get deployments -n sockshop-dev carts -owide
```

```
NAME    READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                 SELECTOR
carts   1/1     1            1           56m   carts        docker.io/keptnexamples/carts:0.11.1   app=carts
```

When sending a new-artifact, we are updating the helm charts values.yaml file with the respective image name.

* [chart/values.yaml](https://github.com/keptn/examples/blob/0.6.2/onboarding-carts/carts/values.yaml#L1)
* [chart/templates/deployment.yaml](https://github.com/keptn/examples/blob/0.6.2/onboarding-carts/carts/templates/deployment.yaml#L22)

### Blue-green deployments

In case of `deployment_strategy: blue_green_service` (see 
 [shipyard.yaml](https://github.com/keptn/examples/blob/0.6.2/onboarding-carts/shipyard.yaml)), Helm creates two
 deployments within the Kubernetes cluster: the primary and the canary deployment. This can be inspected using the
 following command:

```console
$ kubectl get deployments -n sockshop-staging carts carts-primary -owide
```

```
NAME            READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                 SELECTOR
carts-primary   1/1     1            1           56m   carts        docker.io/keptnexamples/carts:0.11.1   app=carts-primary
carts           0/0     0            0            3m   carts        docker.io/keptnexamples/carts:0.11.2   app=carts
```


When a new artifact is deployed (e.g., 0.11.2), a canary deployment will be modified and scaled up.

```
NAME            READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                 SELECTOR
carts-primary   1/1     1            1           57m   carts        docker.io/keptnexamples/carts:0.11.1   app=carts-primary
carts           0/1     1            1            1m   carts        docker.io/keptnexamples/carts:0.11.2   app=carts
```

The primary deployment is always available (and called `carts-primary`). The canary deployment (called `carts`) gets 
 scaled up in the case of a new-artifact event (e.g., in this case someone has sent a new-artifact for 0.11.2). Traffic 
 is shifted to the canary release. 
 
Once testing has finished, the primary deployment is upgraded to the new version (0.11.2). 
 
```
NAME            READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                 SELECTOR
carts-primary   1/1     1            1            3m   carts        docker.io/keptnexamples/carts:0.11.2   app=carts-primary
carts           1/1     1            1            1d   carts        docker.io/keptnexamples/carts:0.11.2   app=carts
```

After a new pod for the primary deployment has been successfully deployed, traffic is shifted to the primary deployment
 and the canary deployment is scaled down:

```
NAME            READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES                                 SELECTOR
carts-primary   1/1     1            1            4m   carts        docker.io/keptnexamples/carts:0.11.2   app=carts-primary
carts           0/0     0            0            1d   carts        docker.io/keptnexamples/carts:0.11.2   app=carts
```

### Clean-up after deleting a project

When executing `keptn delete project PROJECTNAME` (see [cli docs](../cli/#keptn-delete-project)), we do not clean up
 existing deployments nor helm releases. To do so, the following manual steps are necessary:
 
1. Delete all relevant helm releases:
  * Connect directly to the helm-service using ``kubectl exec -it svc/helm-service -n keptn sh``
  * Execute the following command to verify that releases still exists using `helm ls`, e.g.: 
  
    ```console
    helm ls | grep sockshop
    ```
    
    **Note**: Take special care and double check that this command only lists the releases related to the project, and no others!
  
  * Delete the relevant releases using ``helm delete --purge``, e.g.:
  
    ```console
    helm delete --purge $(helm ls | grep sockshop | cut -f1 | tr '\n' ' ')
    ```

2. Delete all relevant namespaces:
  * For each stage defined the shipyard.yaml file, execute `kubectl delete namespace PROJECTNAME-STAGENAME`, e.g.:
  
    ```console
    kubectl delete namespace sockshop-dev
    kubectl delete namespace sockshop-staging
    kubectl delete namespace sockshop-production
    ```
