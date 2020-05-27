# Openshift-Jaeger
A guide on how to deploy jaeger tracing on Openshift

## Introduction

With the ever growing adoption of cloud native application development and microservices architecture, the applications being deployed to the cloud are now comprised of from a few to hundreds of microservices. In order to track the inter communication between each individual microservices, distributed tracing has been implemented to help DevOps teams to keep track of requests being relayed from one service to another. Among all the distributed tracing frameworks, Jaeger and Zipkin are the most popular stack loved by the developers. In the context of Openshift, Jaeger is arguably the best choice for distributed tracing as it's already an integral component of the Openshift Service Mesh framework. Speaking of Openshift Service Mesh, it is the complete solution to optimize the network of deployed services on Openshift with functions like load balancing, service-to-service securities, distributed tracing and monitoring, etc. If you are looking for the complete all in one solution for macro-managing your microservices on Openshift, take a look at the Openshift Service Mesh documents. For developers who are interested only in the distributed tracing side of the equation, the following sections will explain in detail on how to deploy and configure Jaeger and your applications properly for tracing.

## Install Jaeger Operator


## Configure Jaeger Operator


## Prepare your application for Openshift

## Additional Configurations

## Summary
