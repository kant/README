# How to create applications which span multiple namespaces

By default, the components of an application are in the same namespace as the application itself - that is the Kubernetes resources (such as the Pods, Deployments, ReplicaSets, etc) are all created in the same namespace. The default behaviour of kAppNav is to scope the label selector specified in the Application cutom resource to the namespace in which it is created. This means an application's label selector is applied only to the specified component kinds in the same namespace as the application.

Components, including other Application custom resources, from other namespaces can be included in the label selector search by listing multiple namespaces within a comma-delimited list on the `kappnav.component.namespaces` annotation on the Application custom resource.

Spanning namespaces is particularly valuable when an application is composed of multiple microservices which are created and managed by teams which deploy to isolated namespaces.

## A microservice architecture example 

Consider the following example: a business application called 'music-service' which is composed of microservices created by two teams: the UI team and the API team.

Each team is responsible for creating multiple microservices, and they organize themselves into their own namespaces, namely 'ui-namespace' and 'api-namespace'. Each team develops and deploys multiple microservices in their respective namespaces.

In this example, the 'music-service' application uses the 'web-ui' developed by the UI team, and the 'streaming-api' developed by the API team.

To create an Application custom resource which reflects the 'music-service' application, additional configuration is needed to instruct kAppNav to span the namespaces:
```
metadata:
  annotations:
    kappnav.component.namespaces: ui-namespace, api-namespace
```

The optional `kappnav.component.namespaces` annotation instructs kAppNav to apply the application's label selector to namespaces 'ui-namespace' and 'api-namespace' in addition to the application's own namespace.

The [music-service sample application](https://github.com/kappnav/samples/tree/master/music-service) implements this example scenario using the `solution: music-service` label to group all of the resources together.

![image](/images/span-namespace-music-service.png?raw=true)

The [application yaml](https://github.com/kappnav/samples/blob/master/music-service/music-service-app-cr.yaml):
```
$ kubectl get application music-service -n music-service -o yaml
apiVersion: app.k8s.io/v1beta1
kind: Application
metadata:
  name: music-service
  annotations:
    kappnav.component.namespaces: ui-namespace, api-namespace
  labels:
    app: music-service
    solution: music-service
spec:
  componentKinds:
  - group: resources
    kind: Pod
  - group: resources
    kind: Deployment
  - group: resources
    kind: Service
  - group: resources
    kind: Route
  selector:
    matchLabels:
      solution: music-service
```
