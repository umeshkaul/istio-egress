Having two service entries using external proxy does not work. As the test below shows the first SE (wttr.in in below example) works but the attempts to connect to second SE (httpbin.org in below example) end up being forwarded to first SE (wttr.in in below example)   

- used external proxy for testing
`docker run -d --rm --name netshoot --network kind nicolaka/netshoot:latest  sleep infinity`

- manifests used for testing 
```
wttr-in-external-svc-with-proxy.yaml
httpbin-org-external-svc-with-proxy.yaml  
curl-client1.yaml  
```

```

k get se,gateway,tcproute,dr,authorizationpolicy -A
NAMESPACE      NAME                                               HOSTS                 LOCATION        RESOLUTION   AGE
istio-system   serviceentry.networking.istio.io/squid.kind        ["squid.kind"]        MESH_EXTERNAL   DNS          2d10h
istio-system   serviceentry.networking.istio.io/wttr.in           ["wttr.in"]           MESH_EXTERNAL   DNS          2d10h
istio-system   serviceentry.networking.istio.io/www.httpbin.org   ["www.httpbin.org"]   MESH_EXTERNAL   DNS          2d10h

NAMESPACE      NAME                                         CLASS            ADDRESS        PROGRAMMED   AGE
istio-system   gateway.gateway.networking.k8s.io/waypoint   istio-waypoint   10.96.21.212   True         2d10h

NAMESPACE      NAME                                             AGE
istio-system   tcproute.gateway.networking.k8s.io/tcp-httpbin   2d10h
istio-system   tcproute.gateway.networking.k8s.io/tcp-wttr-in   2d10h

NAMESPACE      NAME                                                    HOST         AGE
istio-system   destinationrule.networking.istio.io/httpbin-via-proxy   squid.kind   2d10h
istio-system   destinationrule.networking.istio.io/wttr-in-via-proxy   squid.kind   2d10h

```

```
#try wttr.in
k -n curl-client1 exec -it shell -- curl -s 'https://wttr.in/{NewYork,NewDelhi}?format=3'
NewYork: ☁️   +54°F
NewDelhi: 🌦   +89°F

# proxy log
1746846329.405    456 192.168.132.2 TCP_TUNNEL/200 3626 CONNECT wttr.in:443 - HIER_DIRECT/5.9.243.187 -

# istio log
waypoint-dd5c9497f-2q27t istio-proxy [2025-05-10T03:05:28.945Z] "- - -" 0 - - - "-" 848 3587 460 - "-" "-" "-" "-" "192.168.132.3:3128" outbound|3128||squid.kind 10.244.0.21:51914 240.240.0.25:443 10.244.0.11:33470 - -



# now try httpbin.org
λ k -n curl-client1 exec -it shell -- curl https://www.httpbin.org/ip
curl: (60) SSL: no alternative certificate subject name matches target host name 'www.httpbin.org'
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
command terminated with exit code 60

# proxy log shows that connection still goest to wttr.in
1746846402.507    235 192.168.132.2 TCP_TUNNEL/200 3288 CONNECT wttr.in:443 - HIER_DIRECT/5.9.243.187 -

# istio log
waypoint-dd5c9497f-2q27t istio-proxy [2025-05-10T03:06:09.046Z] "- - -" 0 - - - "-" 605 3249 228 - "-" "-" "-" "-" "192.168.132.3:3128" outbound|3128||squid.kind 10.244.0.21:45942 240.240.0.28:443 10.244.0.11:33470 - -

```
