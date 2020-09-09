# openshift-security-zones
Openshift Nodes for specific Security Network Zones

## MachineSet for Service Nodes

Create a new MachineSet for provisioning new Nodes
<wip>
  
Or Label some prexisting nodes

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
      domain: srvapps.ocplab.tcloud.telefonica.com.ar
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
 
 ## Namespace
 
 ## App
 
 ## Service
 
 ## Route
