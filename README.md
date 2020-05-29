# Openshift-Jaeger
A guide on how to deploy jaeger tracing on Openshift

## Introduction

With the ever growing adoption of cloud native application development and microservices architecture, the applications being deployed to the cloud are now comprised of from a few to hundreds of microservices. In order to track the inter communication between each individual microservices, distributed tracing has been implemented to help DevOps teams to keep track of requests being relayed from one service to another. Among all the distributed tracing frameworks, Jaeger and Zipkin are the most popular stack loved by the developers. In the context of Openshift, Jaeger is arguably the best choice for distributed tracing as it's already an integral component of the Openshift Service Mesh framework. Speaking of Openshift Service Mesh, it is the complete solution to optimize the network of deployed services on Openshift with functions like load balancing, service-to-service securities, distributed tracing and monitoring, etc. If you are looking for the complete all in one solution for macro-managing your microservices on Openshift, take a look at the Openshift Service Mesh documents. For developers who are interested only in the distributed tracing side of the equation, the following sections will explain in detail on how to deploy and configure Jaeger and your applications properly for tracing.

## Install Jaeger Operator

The best way to deploy Jaeger to Openshift is using the Jaeger Operator. It's extremely easy to install via the use of Operator Hub. It also helps the user to automate the setup of many security related of configurations. We also have to choose a backend storage solution to persist the data. For Jaeger, the official supported storage options are either Cassandra or Elasticsearch. For this article, I'll pick Elasticsearch due to my past familiarity with Elasticsearch before. 

Before the installation, let's first create a new project on our Openshift cluster called jaeger-tracing.
```
oc new-project jaegertracing
```

Next, we'll first install ElasticSearch Operator and then Jaeger Operator. This is so that the Jaeger Operator can self provision the existing ElasticSearch Opeartor with their already generated certificates stored as the Kubernetes Secrets.

To install Elasticsearch Operator, nagivate to the Operator Hub and search for Elasticsearch. Follow the prompt to install the operator and make sure that it is installed to the **jaeger-tracing** project created earlier. 

**Insert screenshot here**

After the installation of Elasticsearch Operator, follow the similar steps on the Operator hub to install the Jaeger Operator. Similarly, make sure that it is installed to the same **jaeger-tracing** project.

**Insert screenshot here**

After the installation of both Elasticsearch Operator and Jaeger Operator, navigate to the Installed Operators page to verify that both of operators have been installed successfully.

## Configure Jaeger Operator

Before creating a Jaeger instance, let's recap on the Jaeger components. The Jaeger is comprised of agent, collector and query three main components. Additionally, there are jaeger client which is part of your application responsible for instrumenting the appliaction and creating spans. And finally, Jaeger makes use of third party storage solutions such as Cassandra and Elasticsearch for data persistence. The overall architecure is illustrated in the diagram below.

**Insert screenshot here**

Next, we need to figure out which deployment strategy to use. A jaeger strategy dictates how the jaeger components should be deployed on Openshift. The Jaeger CR from Jaeger Operator offers three types of deployment strategy:
1. allInOne
2. Production
3. Streaming

As tempting as one might want to use allInOne since it's the default strategy, its extremely limited capability makes it not suitable for even some serious test/development settings. As the name suggets, allInOne strategy packs the agent, collector and query in a single pod. It's an easy way to start and manage all Jaeger components for some POC works. However there are two limitations that severely handicap the potential of this setup. For one, the allInOne strategy is using memory as its backend storage, so there is no real data persistence as killing the pod will wipe out every single collected trace spans. And secondly, allInOne does not allow any scaling as the replica count is locked to 1.

Production strategy in contrast deploys the collector and query processes individually as seperate pods on Openshift, and can be scaled individually on demand. In addition, it supports secured connection with backend storage services like Cassandra and Elasticsearch, which fits our need perfectly. Streaming strategy is essentially production strategy with added support for streaming platform like Kafka, which will ignore for now for the sake of simplicity.

Let's proceed to create the jaeger yaml file

```
apiVersion: jaegertracing.io/v1
kind: Jaeger
metadata:
  name: simple-prod
spec:
  strategy: production
  storage:
    type: elasticsearch
    elasticsearch:
      nodeCount: 1
      resources:
        requests:
          cpu: 200m
          memory: 1Gi
        limits:
          memory: 1Gi
```

Note that there is barely any detailed configuration for Elasticsearch in the Jaeger yaml file. The Jaeger Operator is able to detect existing Elasticsearch instanced deployed by the Elasticsearch Operator, and autoprovision the secured communication between Elasticsearch and Jaeger processes.

Create the Jaeger instance using the yaml file above. Upon successful creation of the Jaeger instance, the following pod processes will be running on your cluster.

```
[root@fwji1-ocp43-inf ~]# oc get pods
NAME                                                           READY   STATUS      RESTARTS   AGE
elasticsearch-cdm-jaegertrarcingsimpleprod-1-cb8fd749b-kqtkx   2/2     Running     0          50d
simple-prod-collector-6f654659bb-8868l                         1/1     Running     0          9d
simple-prod-query-864dfd87b4-wxrps                             2/2     Running     0          12h
```

## Prepare your application for Openshift

You might wonder why there is no agent pod running after the creation of the Jaeger instance. That's because the agent is supposed to be injected as a sidecar container within the application pod in production (and streaming) strategy. The auto injection of the agent side car can be done through annotating the **Namespace** or **Deployments** with **sidecar.jaegertracing.io/inject: "true"** annotation. For controller types other than **Deployment** (ie. StateFulSet, DeploymentConfig, etc), the jaeger agent sidecar must be explicitly defined in the container spec. An example of a StateFulSet with Jaeger agent sidecar container is available at here.

Below is a sample deployment with automatic agent sidecar injection. We also defined some environment variables that are used by my Jaeger client in my application.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  annotations:
    sidecar.jaegertracing.io/inject: "true"
  creationTimestamp: null
  labels:
    app: system
  name: system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: system
  strategy:
    resources: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: system
    spec:
      containers:
      - env:
        - name: JAEGER_REPORTER_LOG_SPANS
          value: "true"
        - name: JAEGER_SAMPLER_PARAM
          value: "1"
        - name: JAEGER_SAMPLER_TYPE
          value: const
        image: 'frankji/system-image'
        name: system
        ports:
        - containerPort: 9080
        - containerPort: 9443
        resources: {}
      restartPolicy: Always
  test: false
status: {}
```
Once deplyoed, the application pod will be created with a Jaeger agent sidecar. You should then be able to see the trace spans generated by the application on the Jaeger query UI.

**Insert screenshot here**

## Additional Configurations

The above example provided a typical setup of Jaeger tracing on Openshift. However depending on the actual use cases and the requirements, there are many other configurations that are possible with Jaeger Operator. For instance, the Elasticsearch used in this example can be replaced with external Elasitcsearch link by providing the exterrnal server url of the Elasticsearch and its certificates. Another important configuration is the collector sampling strategies. In a production microservice environment, it may be too expensive to collect and store every trace span reported from the client. As such, a user can configure the collector to use either a probablisitic based or a rate limiting based sampling strategy to reduce the frequency of trace collection. For more advanced users, the addition of a streaming platform like Apache Kafka with the streaming strategy is definitely a worthwhile investment. Not only it can reduce the load on the backend storage during peak loads with its additional layer of storage, it also provides the opporunity for data ingesion on the collected tracing data that can give users more insight on the target microservices system.

## Summary

Distributed tracing has become an essential in the world of microservice observability. With Jaeger Operator, user can now start doing distributed tracing on Openshift with minimum configuration. Yet it still offers tons of configurables for advanced users to leverage the full potential of Jaeger. If you want to learn more about Jaeger, the Jaeger tracing home page is still the best resource for Jaeger and Jaeger Operator contents. 
