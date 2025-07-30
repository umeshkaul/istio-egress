## Test separate SE for squid proxy

- Apply wttr manifests with its own SE for squid proxy 

```
# apply wttr manifests

[shambu]ukaul in ~/Projects/istio-egress/ambient-mode/external-proxy on main ● λ k apply -f wttr-in-external-svc-with-proxy.yaml
gateway.gateway.networking.k8s.io/waypoint created
serviceentry.networking.istio.io/wttr.in created
serviceentry.networking.istio.io/squid-wttr-proxy created
tcproute.gateway.networking.k8s.io/tcp-wttr-in created
destinationrule.networking.istio.io/wttr-in-via-proxy created

[shambu]ukaul in ~/Projects/istio-egress/ambient-mode/external-proxy on main ● λ k get se,gateway,tcproute,dr,authorizationpolicy -A
NAMESPACE      NAME                                                HOSTS            LOCATION        RESOLUTION   AGE
istio-system   serviceentry.networking.istio.io/squid-wttr-proxy   ["squid.kind"]   MESH_EXTERNAL   DNS          70s
istio-system   serviceentry.networking.istio.io/wttr.in            ["wttr.in"]      MESH_EXTERNAL   DNS          70s

NAMESPACE      NAME                                         CLASS            ADDRESS        PROGRAMMED   AGE
istio-system   gateway.gateway.networking.k8s.io/waypoint   istio-waypoint   10.96.197.30   True         70s

NAMESPACE      NAME                                             AGE
istio-system   tcproute.gateway.networking.k8s.io/tcp-wttr-in   70s

NAMESPACE      NAME                                                    HOST         AGE
istio-system   destinationrule.networking.istio.io/wttr-in-via-proxy   squid.kind   70s

# test wttr
[shambu]ukaul in ~/Projects/istio-egress/ambient-mode/external-proxy on main ● λ k -n curl-client1 exec -it shell -- curl -s 'https://wttr.in/{NewYork,NewDelhi}?format=3'
NewYork: ☀️   +71°F
NewDelhi: 🌧   +77°F

 # proxy log
1750428695.328    447 192.168.132.2 TCP_TUNNEL/200 3661 CONNECT wttr.in:443 - HIER_DIRECT/5.9.243.187 -

# istio log
ztunnel-bd6r8 istio-proxy 2025-06-20T14:11:35.328438Z    info    access    connection complete    src.addr=10.244.0.11:35964 src.workload="shell" src.namespace="curl-client1" src.identity="spiffe://cluster.local/ns/curl-client1/sa/curl-client1" dst.addr=10.244.0.42:15008 dst.hbone_addr=240.240.0.88:443 dst.service="wttr.in" dst.workload="waypoint-dd5c9497f-xknp9" dst.namespace="istio-system" dst.identity="spiffe://cluster.local/ns/istio-system/sa/waypoint" direction="outbound" bytes_sent=848 bytes_recv=3622 duration="452ms"
waypoint-dd5c9497f-xknp9 istio-proxy [2025-06-20T14:11:34.876Z] "- - -" 0 - - - "-" 848 3622 451 - "-" "-" "-" "-" "192.168.132.3:3128" outbound|3128||squid.kind 10.244.0.42:38432 240.240.0.88:443 10.244.0.11:42462 - -


```

- Apply httpbin  manifests with its own SE for squid proxy

```
# apply httpbin manifests
[shambu]ukaul in ~/Projects/istio-egress/ambient-mode/external-proxy on main ● λ  k apply -f httpbin-org-external-svc-with-proxy.yaml
gateway.gateway.networking.k8s.io/waypoint unchanged
serviceentry.networking.istio.io/www.httpbin.org created
serviceentry.networking.istio.io/squid-httpbin-proxy created
tcproute.gateway.networking.k8s.io/tcp-httpbin created
destinationrule.networking.istio.io/httpbin-via-proxy created

[shambu]ukaul in ~/Projects/istio-egress/ambient-mode/external-proxy on main ● λ k get se,gateway,tcproute,dr,authorizationpolicy -A
NAMESPACE      NAME                                                   HOSTS                 LOCATION        RESOLUTION   AGE
istio-system   serviceentry.networking.istio.io/squid-httpbin-proxy   ["squid.kind"]        MESH_EXTERNAL   DNS          15s
istio-system   serviceentry.networking.istio.io/squid-wttr-proxy      ["squid.kind"]        MESH_EXTERNAL   DNS          4m29s
istio-system   serviceentry.networking.istio.io/wttr.in               ["wttr.in"]           MESH_EXTERNAL   DNS          4m29s
istio-system   serviceentry.networking.istio.io/www.httpbin.org       ["www.httpbin.org"]   MESH_EXTERNAL   DNS          15s

NAMESPACE      NAME                                         CLASS            ADDRESS        PROGRAMMED   AGE
istio-system   gateway.gateway.networking.k8s.io/waypoint   istio-waypoint   10.96.197.30   True         4m29s

NAMESPACE      NAME                                             AGE
istio-system   tcproute.gateway.networking.k8s.io/tcp-httpbin   15s
istio-system   tcproute.gateway.networking.k8s.io/tcp-wttr-in   4m29s

NAMESPACE      NAME                                                    HOST         AGE
istio-system   destinationrule.networking.istio.io/httpbin-via-proxy   squid.kind   15s
istio-system   destinationrule.networking.istio.io/wttr-in-via-proxy   squid.kind   4m29s

```

- Test wttr again and then httpbin

```
# test wttr - works fine

[shambu]ukaul in ~/Projects/istio-egress/ambient-mode/external-proxy on main ● ● λ k -n curl-client1 exec -it shell -- curl -s 'https://wttr.in/{NewYork,NewDelhi}?format=3'
NewYork: ☀️   +71°F
NewDelhi: 🌧   +77°F

# proxy log
1750429346.137   1145 192.168.132.2 TCP_TUNNEL/200 3661 CONNECT wttr.in:443 - HIER_DIRECT/5.9.243.187 -

# istio log
ztunnel-bd6r8 istio-proxy 2025-06-20T14:22:26.137598Z   info    access  connection complete     src.addr=10.244.0.11:47396 src.workload="shell" src.namespace="curl-client1" src.identity="spiffe://cluster.local/ns/curl-client1/sa/curl-client1" dst.addr=10.244.0.42:15008 dst.hbone_addr=240.240.0.88:443 dst.service="wttr.in" dst.workload="waypoint-dd5c9497f-xknp9" dst.namespace="istio-system" dst.identity="spiffe://cluster.local/ns/istio-system/sa/waypoint" direction="outbound" bytes_sent=848 bytes_recv=3622 duration="1150ms"
waypoint-dd5c9497f-xknp9 istio-proxy [2025-06-20T14:22:24.987Z] "- - -" 0 - - - "-" 848 3622 1149 - "-" "-" "-" "-" "192.168.132.3:3128" outbound|3128||squid.kind 10.244.0.42:33054 240.240.0.88:443 10.244.0.11:42462 - -
```


# test httpbin - as you can see it is redirected to wttr

```
[shambu]ukaul in ~/Projects/istio-egress/ambient-mode/external-proxy on main ● λ k -n curl-client1 exec -it shell -- curl https://www.httpbin.org/ip
curl: (60) SSL: no alternative certificate subject name matches target host name 'www.httpbin.org'
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
command terminated with exit code 60


# proxy log
1750428986.788    233 192.168.132.2 TCP_TUNNEL/200 3301 CONNECT wttr.in:443 - HIER_DIRECT/5.9.243.187 -


# istio log
ztunnel-bd6r8 istio-proxy 2025-06-20T14:16:26.788393Z    info    access    connection complete    src.addr=10.244.0.11:37456 src.workload="shell" src.namespace="curl-client1" src.identity="spiffe://cluster.local/ns/curl-client1/sa/curl-client1" dst.addr=10.244.0.42:15008 dst.hbone_addr=240.240.0.92:443 dst.service="www.httpbin.org" dst.workload="waypoint-dd5c9497f-xknp9" dst.namespace="istio-system" dst.identity="spiffe://cluster.local/ns/istio-system/sa/waypoint" direction="outbound" bytes_sent=605 bytes_recv=3262 duration="238ms"
waypoint-dd5c9497f-xknp9 istio-proxy [2025-06-20T14:16:26.550Z] "- - -" 0 - - - "-" 605 3262 237 - "-" "-" "-" "-" "192.168.132.3:3128" outbound|3128||squid.kind 10.244.0.42:35904 240.240.0.92:443 10.244.0.11:42462 - -

```
