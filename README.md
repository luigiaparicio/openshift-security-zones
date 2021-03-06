# openshift-security-zones
Openshift Nodes for specific Security Network Zones

## MachineSet for Service Nodes

Create a new MachineSet for provisioning new Nodes
<wip>
  
or label some prexisting nodes

    oc label <NODE> srv-node=true ingressaccess=true

## New IngressController

    apiVersion: operator.openshift.io/v1
    kind: IngressController
    metadata:
      name: srv-ingress
      namespace: openshift-ingress-operator
    spec:
      endpointPublishingStrategy:
        type: HostNetwork
      domain: some-other-apps.ocp-cluster.example.com
      replicas: 2
      nodePlacement:
        nodeSelector:
          matchLabels:
            srv-node: "true"
            ingressaccess: "true"
      namespaceSelector:
        matchExpressions:
          - key: srv-ingress
            operator: In
            values:
            - true
        
        
        

## Modify default IngressController
    oc edit ingresscontrollers.operator.openshift.io -n openshift-ingress-operator default
    
    ...
    spec:
      namespaceSelector:
        matchExpressions:
         - key: securityzone
           operator: DoesNotExist
    ...


 ## Namespace
 
    apiVersion: v1
    kind: Namespace
    metadata:
      name: test-srv-app
      labels:
        srv-ingress: true
 

## Deployment

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      namespace: test-srv-app
      name: test-srv-app
      labels:
        app: test-srv-app
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: test-srv-app
      template:
        metadata:
          labels:
            app: test-srv-app
        spec:
          containers:
          - name: test-srv-app
            image: gcr.io/google-samples/hello-app:1.0
          tolerations:
            - key: "srv-ingress"
              operator: "Equal"
              value: "true"
              effect: "NoSchedule"
          nodeSelector:
            srv-node: 'true'


## Service

    apiVersion: v1
    kind: Service
    metadata:
      name: test-srv-app
      namespace: test-srv-app
    spec:
      selector:
        app: test-srv-app
      ports:
        - name: 8080-tcp
          protocol: TCP
          port: 8080
          targetPort: 8080



## Route
    kind: Route
    apiVersion: route.openshift.io/v1
    metadata:
      name: test-srv-app
      namespace: test-srv-app
    spec:
      host: hello.some-other-apps.ocp-cluster.example.com
      to:
        kind: Service
        name: test-srv-app
      port:
        targetPort: 8080-tcp
        
## Include a label in the default namespace to permit IngressController traffic

Because we are using HostNetwork for our new IngressController\
Add this label _network.openshift.io/policy-group: ingress_

    oc edit namespace default
    
    apiVersion: v1
    kind: Namespace
    metadata:
      labels:
        network.openshift.io/policy-group: ingress
      annotations:
...
