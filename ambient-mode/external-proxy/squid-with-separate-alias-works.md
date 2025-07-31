- having separate alias for same squid proxy works, see config and logs below

```
$>cat docker-compose.yaml
version: '3.8'
services:
  squid-proxy:
    image: ubuntu/squid
    container_name: squid
    ports:
      - "3138:3128"
    networks:
      kind:
        aliases:
          - squid1
          - squid2
          - squid3
networks:
  kind:
    external: true

$>docker ps
CONTAINER ID   IMAGE                  COMMAND                  CREATED        STATUS        PORTS                                         NAMES
a8edb415ec09   ubuntu/squid           "entrypoint.sh -f /e…"   2 hours ago    Up 2 hours    0.0.0.0:3138->3128/tcp, [::]:3138->3128/tcp   squid
0672ffbab2ec   kindest/node:v1.32.2   "/usr/local/bin/entr…"   3 months ago   Up 3 months   127.0.0.1:43465->6443/tcp                     test-ambient1-control-plane



$>k -n curl-client1 exec -it shell -- curl -s 'https://wttr.in/{NewYork,NewDelhi}?format=3'
NewYork: ⛅️  +84°F
NewDelhi: ☀️   +82°F
$>
$>k -n curl-client1 exec -it shell -- curl https://www.httpbin.org/ip
{
  "origin": "47.20.213.132"
}




ztunnel-bd6r8 istio-proxy 2025-07-31T17:52:14.631022Z info  access  connection complete src.addr=10.244.0.11:44552 src.workload="shell" src.namespace="curl-client1" src.identity="spiffe://cluster.local/ns/curl-client1/sa/curl-client1" dst.addr=10.244.0.48:15008 dst.hbone_addr=240.240.0.112:443 dst.service="wttr.in" dst.workload="waypoint-dd5c9497f-59smm" dst.namespace="istio-system" dst.identity="spiffe://cluster.local/ns/istio-system/sa/waypoint" direction="outbound" bytes_sent=848 bytes_recv=3648 duration="444ms"
waypoint-dd5c9497f-59smm istio-proxy [2025-07-31T17:52:14.186Z] "- - -" 0 - - - "-" 848 3648 443 - "-" "-" "-" "-" "192.168.132.3:3128" outbound|3128||squid1.kind 10.244.0.48:35320 240.240.0.112:443 10.244.0.11:48306 - -
ztunnel-bd6r8 istio-proxy 2025-07-31T17:52:22.077933Z info  access  connection complete src.addr=10.244.0.11:38788 src.workload="shell" src.namespace="curl-client1" src.identity="spiffe://cluster.local/ns/curl-client1/sa/curl-client1" dst.addr=10.244.0.48:15008 dst.hbone_addr=240.240.0.115:443 dst.service="www.httpbin.org" dst.workload="waypoint-dd5c9497f-59smm" dst.namespace="istio-system" dst.identity="spiffe://cluster.local/ns/istio-system/sa/waypoint" direction="outbound" bytes_sent=850 bytes_recv=4643 duration="669ms"
waypoint-dd5c9497f-59smm istio-proxy [2025-07-31T17:52:21.409Z] "- - -" 0 - - - "-" 850 4643 668 - "-" "-" "-" "-" "192.168.132.3:3128" outbound|3128||squid2.kind 10.244.0.48:60278 240.240.0.115:443 10.244.0.11:48306 - -



1753984334.630    439 192.168.132.2 TCP_TUNNEL/200 3687 CONNECT wttr.in:443 - HIER_DIRECT/5.9.243.187 -
1753984342.077    664 192.168.132.2 TCP_TUNNEL/200 4682 CONNECT www.httpbin.org:443 - HIER_DIRECT/3.224.80.105 -
```
