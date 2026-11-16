# Event-Driven Architecture 

<!--- cSpell:ignore CNCF qube cntk autoplay allowfullscreen Apicur Strimzi Flink refarch -->

A GitOps solution to deploy the elements of an event-driven application can be found at the link below. 
The products covered are cloud Pak for Integration: Event Streams, MQ, Event end-point management (API Connect), but also
some open sources products like Sealed Secrets, Strimzi, Apicur.io, Elastic Search, OpenShift GitOps, OpenShift Pipelines, Grafana, Apache Flink.

* [Event-Driven Architecture GitOps Solution](https://ibm-cloud-architecture.github.io/refarch-eda/use-cases/gitops/)

For event-driven microservice implementation, the following [eda-quickstart](https://github.com/ibm-cloud-architecture/eda-quickstarts) repository includes code 
templates, based on Domain-driven design practices, for Java Quarkus, Liberty and Spring Cloud implementations
as well as Python.

!!!note
    This solution does not use exactly the same configuration framework as the Production Deployment Guides. It uses the 
    [Catalog repository](https://github.com/ibm-cloud-architecture/eda-gitops-catalog) approach to define Operator's subscriptions and solution based repositories that use the needed operators in
    the scope of the specific even-driven solution. 

The following even-driven solution GitOps repositories are illustrating the proposed approach:

* [refarch-kc-gitops](https://github.com/ibm-cloud-architecture/refarch-kc-gitops): For the shipping fresh food overseas solution we have defined. It includes
the SAGA choreography pattern implemented with Kafka
* [eda-kc-gitops](https://github.com/ibm-cloud-architecture/eda-kc-gitops): For the shipping fresh food overseas solution we have defined. It includes
the SAGA orchestration pattern implemented with MQ
* [eda-rt-inventory-gitops](https://github.com/ibm-cloud-architecture/eda-rt-inventory-gitops) to deploy the demo of real-time inventory
