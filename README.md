### Istio egress use case comparison between sidecar mode and ambient mode

Use Case:
Allow outbound traffic to external services only from specified pods via specified nodes.   

- Use network policies to ensure that traffic to external services is only allowed from selected nodes in a given cluster. This is not dependent on istio at all and can be achieved via network policies.
- use sidecar mode to route traffic out of the cluster via these nodes  for these external services
- sometimes these external services are only reachable via an external proxy so we need to route the traffic from egress gateway via that proxy

I need to make 2 and 3 above work for ambient mode and have 2 working but not 3.

```

Workload----https---->SideCar----https---->Egress---------https-----External_Proxy------>External_Service

External_Proxy here is a squid proxy reachable on address squid.kind from within the cluster, 
Below is an example request from a netshoot pod (note that curl does not specify a proxy) to https://postman-echo.com/ip

$ k exec -it netshoot-847bc77f76-qk72j --  curl  https://postman-echo.com/ip -k
 {
   "ip": "47.20.213.132"
 }
```
