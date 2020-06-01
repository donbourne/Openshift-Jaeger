#Sample Open Liberty Application with Jaeger Support

This repository contain two application deployment yaml files that will deploy two OpenLiberty Java applications with MicroProfile OpenTracing 1.3 feature. Both of the applications will send tracing spans to Jaeger on Red Hat Openshift with Jaeger Operator.

##Prerequesites

Before deploying the applications to Red Hat Openshift cluster, make sure:

1. Elasticsearch operator is installed
2. Jaeger Operator is installed
3. A Jaeger instance is created with Production strategy

##Deploy

Use oc command to deploy the applications.
```
oc create -f system.yaml
```
```
oc create -f inventory.yaml
```

##Generate traces

To generate traces, visit the following url:
- http://<inventory-route-jaegertrarcing>/inventory/systems/system
- http://<system-route>/system/properties
