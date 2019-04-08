---
layout: post
title:  "Gitops and EnMasse"
author: Ulf Lilleengen
categories: technical kubernetes enmasse gitops openshift
---

With the EnMasse 0.27.0 release, using a [Gitops](https://www.weave.works/technologies/gitops/)
workflow to manage your messaging application is even easier than before.

Gitops is a way to do Continuous Delivery where not only the source code of an application but all configuration for an application is stored in git, and changes to a production environment involve creating a pull/change requests to that git repo. Once the PR has been tested and reviewed, it can be merged. When merged, a CD job is triggered that will apply the current state of the git repository to the system.

The declarative gitops model fits well with how Kubernetes works, because you can store your Kubernetes configuration in git, and trigger some process to apply the configuration to a Kubernetes cluster. If you store your application code together with the Kubernetes configuration, you enable development teams to be in full control of their application deployment to any cluster environment.

Traditionally, Kubernetes has mainly been used for stateless services. Stateful services has normally been run outside the Kubernetes cluster.
If a development team wants to use a stateful service, the team normally have to install and manage the service themselves or use a cloud provider service.

EnMasse is a stateful service that runs on Kuberentes, where the responsibility of operating the messaging service is separate from the tenants consuming it. This makes it easy for development teams to use the gitops model with on-cluster services managed by someone else.

Lets assume that you have a team in your organization managing the messaging infrastructure using EnMasse on Kubernetes or OpenShift, and that you have 2 independent developer teams that both want to use messaging in their applications. The following diagram describes the flow: 
![Gitops]({{ site.url }}/images/enmasse_gitops.png)

The operations team will deploy the messaging infrastructure (EnMasse), and commit the desired configuration templates that they want to support to git. A CI process then applies the EnMasse configuration to the cluster.

Independently of the operations team, the development teams commit their application code along with the messaging resource manifests, such as `AddressSpace`, `Address` and `MessagingUser` (we will get back to what these are), for their application. A CI process builds the applications and applies the application and messaging resources manifests.

## Application

First, lets create a simple messaging application. Writing messaging clients can be a challenging task, but is made very simple with [Quarkus](quarkus.io), which makes writing reactive messaging applications a breeze. The following client code is sufficient to process a incoming messages and act on it:

```
@ApplicationScoped
public class App {
    @Incoming("orders")
    @Outgoing("confirmations")
    public Message<String> processOrder(AmqpMessage<String> order) {
        // Make the order
        return Message.of("Order confirmation");
    }
}
```

For the full example client, see [example clients](https://github.com/EnMasseProject/enmasse-example-clients).

## Messaging resources

Once your application written, some configuration for using the messaging resources available on
your cluster is needed.

In EnMasse, an AddressSpace is a group of addresses that share connection endpoints as well as authentication and
authorization policies. When creating an AddressSpace you can configure how your messaging
endpoints are exposed:


```
apiVersion: enmasse.io/v1beta1
kind: AddressSpace
metadata:
  name: app
  namespace: team1
spec:
  type: standard
  plan: standard-small
  endpoints:
  - name: messaging
    service: messaging
    cert:
      provider: openshift
    exports:
    - name: messaging-config
      kind: ConfigMap
```

For more information about address spaces, see the [address space documentation](https://enmasse.io/documentation/master/openshift/#con-address-space-messaging).

Messages are sent and received from an address. An address has a type that determines its semantics, and a plan that determines how much resources
is reserved for this address. An address can be defined like this:

```
apiVersion: enmasse.io/v1beta1
kind: Address
metadata:
  name: app.orders
  namespace: team1
spec:
  address: orders
  type: queue
  plan: standard-small-queue
```

To ensure that only trusted applications are able to send and receive messages to your addresses, a
messaging user must be created. For applications running on-cluster, you can authenticate clients
using a Kubernetes service account. A serviceaccount user can be defined like this:

```
apiVersion: user.enmasse.io/v1beta1
kind: MessagingUser
metadata:
  name: myspace.app
  namespace: team1
spec:
  username: system:serviceaccount:team1:default
  authentication:
    type: serviceaccount
  authorization:
  - operations: ["send", "recv"]
    addresses: ["orders"]
```

With the above 3 resources, you have the basics needed for an application to use the messaging service.

But how does your application get to know the endpoints for its address space?  You may have noticed the `exports` field in the addres space definition. Exports are a way to instruct EnMasse that you want a configmap with the hostname, ports and CA certificate to be created in your namespace. To allow EnMasse to create this resource, we also need to define a Role and RoleBinding for it:

```
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: messaging-config
  namespace: team1
spec:
rules:
  - apiGroups: [ "" ]
    resources: [ "configmaps" ]
    verbs: [ "create" ]
  - apiGroups: [ "" ]
    resources: [ "configmaps" ]
    resourceNames: [ "messaging-config" ]
    verbs: [ "get", "update", "patch" ]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: messaging-config
  namespace: team1
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: messaging-config
subjects:
- kind: ServiceAccount
  name: address-space-controller
  namespace: enmasse-infra
```

## Wiring configuration into application

With messaging configuration in place we can write the deployment manifest for our application:

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  labels:
    app: myapp
spec:
  replicas: 1
  template:
    metadata:
      matchLabels:
        application: demo
    spec:
      containers:
        - name: app
          image: myapp:latest
          env:
            - name: SMALLRYE_MESSAGING_SOURCE_ORDERS_ADDRESS
              value: orders
            - name: SMALLRYE_MESSAGING_SOURCE_ORDERS_TYPE
              value: Amqp
            - name: SMALLRYE_MESSAGING_SOURCE_ORDERS_HOST
              valueFrom:
                configMapKeyRef:
                  name: messaging-config
                  key: service.host
            - name: SMALLRYE_MESSAGING_SOURCE_ORDERS_PORT
              valueFrom:
                configMapKeyRef:
                  name: messaging-config
                  key: service.port.amqps
            - name: SMALLRYE_MESSAGING_SOURCE_CONFIRMATIONS_ADDRESS
              value: confirmations
            - name: SMALLRYE_MESSAGING_SOURCE_CONFIRMATIONS_TYPE
              value: Amqp
            - name: SMALLRYE_MESSAGING_SOURCE_CONFIRMATIONS_HOST
              valueFrom:
                configMapKeyRef:
                  name: messaging-config
                  key: service.host
            - name: SMALLRYE_MESSAGING_SOURCE_CONFIRMATIONS_PORT
              valueFrom:
                configMapKeyRef:
                  name: messaging-config
                  key: service.port.amqps
```

As you can see, the values of the configmap is mapped as environment variables to our application.
Quarkus reactive messaging will take care of wiring the messaging client.


## Summary

We have seen how you can declare your messaging resources as Kubernetes manifests, as well as your application. This allows your team to follow the gitops model when deploying your applications using messaging on Kubernetes and OpenShift. 

Star our project on [github](https://github.com/EnMasseProject/enmasse/) and follow us on [twitter](https://twitter.com/enmasseio)!
