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
            - secure
        
        
        

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
      name: test-srv
      labels:
        srv-ingress: secure
 

## App

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      namespace: test-srv
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
              value: "secure"
              effect: "NoSchedule"
          nodeSelector:
            srv-node: 'true'


## Service

    apiVersion: v1
    kind: Service
    metadata:
      name: test-srv-app
      namespace: test-srv
    spec:
      selector:
        app: test-srv-app
      ports:
        - name: 8080-tcp
          protocol: TCP
          port: 8080
          targetPort: 8080



## Route
    oc expose svc test-srv-app -n test-srv
