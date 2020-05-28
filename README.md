# Openshift-Jaeger
A guide on how to deploy jaeger tracing on Openshift

## Introduction

With the ever growing adoption of cloud native application development and microservices architecture, the applications being deployed to the cloud are now comprised of from a few to hundreds of microservices. In order to track the inter communication between each individual microservices, distributed tracing has been implemented to help DevOps teams to keep track of requests being relayed from one service to another. Among all the distributed tracing frameworks, Jaeger and Zipkin are the most popular stack loved by the developers. In the context of Openshift, Jaeger is arguably the best choice for distributed tracing as it's already an integral component of the Openshift Service Mesh framework. Speaking of Openshift Service Mesh, it is the complete solution to optimize the network of deployed services on Openshift with functions like load balancing, service-to-service securities, distributed tracing and monitoring, etc. If you are looking for the complete all in one solution for macro-managing your microservices on Openshift, take a look at the Openshift Service Mesh documents. For developers who are interested only in the distributed tracing side of the equation, the following sections will explain in detail on how to deploy and configure Jaeger and your applications properly for tracing.

## Install Jaeger Operator

The best way to deploy Jaeger to Openshift is using the Jaeger Operator. It's extremely easy to install via the use of Operator Hub. It also helps the user to automate the setup of many security related of configurations. We also have to choose a backend storage solution to persist the data. For Jaeger, the official supported storage options are either Cassandra or Elasticsearch. For this article, I'll pick Elasticsearch due to my past familiarity with Elasticsearch before. 

Before the installation, let's first create a new project on our Openshift cluster called jaeger-tracing.
```
oc new-project jaeger-tracing
```

Next, we'll first install ElasticSearch Operator and then Jaeger Operator. This is so that the Jaeger Operator can self provision the existing ElasticSearch Opeartor with their already generated certificates stored as the Kubernetes Secrets.

To install Elasticsearch Operator, nagivate to the Operator Hub and search for Elasticsearch. Follow the prompt to install the operator and make sure that it is installed to the **jaeger-tracing** project created earlier. 

**Insert screenshot here**

After the installation of Elasticsearch Operator, follow the similar steps on the Operator hub to install the Jaeger Operator. Similarly, make sure that it is installed to the same **jaeger-tracing** project.

**Insert screenshot here**

After the installation of both Elasticsearch Operator and Jaeger Operator, navigate to the Installed Operators page to verify that both of operators have been installed successfully.

## Configure Jaeger Operator

The next step is to define an Jaeger instance.

## Prepare your application for Openshift

## Additional Configurations

## Summary
