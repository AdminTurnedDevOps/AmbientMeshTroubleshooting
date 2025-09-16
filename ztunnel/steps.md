## Get All Ztunnel (xDS) Configurations

```
istioctl ztunnel-config all -o json
```


## Pod Logs

```
istioctl zc log POD_NAME -n istio-system
```

```
kubectl logs ds/ztunnel -n istio-system  | grep inpod
```

```
kubectl logs ds/ztunnel -n istio-system
```

```
kubectl logs ISTIOD_POD -n istio-system
```

## Ztunnel Workloads/Certs
```
istioctl zc workloads
```

## Troubleshooting traffic redirection

```
kubectl exec <pod-name> -n istio-system -- iptables-save -c > pod-iptables-with-counters.txt

kubectl exec <pod-name> -- sh -c "iptables-save -t nat && echo '--- MANGLE TABLE ---' && iptables-save -t mangle" > ambient-iptables.txt
```

Confirm Socket State. The goal is to confirm that the `HBONE` ports are open (15001, 15006, 15008)
```
kubectl debug $(kubectl get pod -l app=frontend -n microapp -o jsonpath='{.items[0].metadata.name}') -it -n microapp --image nicolaka/netshoot  -- ss -ntlp
```

Get iptables.
```
sudo kubectl debug $(kubectl get pod -l app=frontend -n microapp -o jsonpath='{.items[0].metadata.name}') -it --image gcr.io/istio-release/base --profile=netadmin -n microapp -- iptables-save
```

## Get Service Entries

Service Entries are local configs in Istiod that tell Istio about Services outside of the service mesh and the cluster that the service mesh is installed on. If you're doing multi-cluster routing, the cluster (where Istiod exists) that the route is set up on will need to know about the Services outside of that cluster.

tldr; Service Entries make external services known to the mesh.
```
kubectl get serviceentry --context $context -n istio-system
kubectl get workloadentry --context $context -n istio-system
```

Example output:
```
NAME                                 HOSTS                                    LOCATION        RESOLUTION   AGE
autogen.microapp.frontend            ["frontend.microapp.mesh.internal"]                      STATIC       9s
dummy-route-blackhole-solo-unused    ["dummy-route.blackhole.solo.unused"]    MESH_INTERNAL   STATIC       14m
graphql-host-blackhole-solo-unused   ["graphql-host.blackhole.solo.unused"]   MESH_INTERNAL   STATIC       14m
NAME                                     AGE   ADDRESS
autogen.workerclus01.microapp.frontend   8s    

Service entries and workload entries for cluster gke_field-engineering-us_us-central1_workerclus01:

NAME                                 HOSTS                                    LOCATION        RESOLUTION   AGE
autogen.microapp.frontend            ["frontend.microapp.mesh.internal"]                      STATIC       10s
dummy-route-blackhole-solo-unused    ["dummy-route.blackhole.solo.unused"]    MESH_INTERNAL   STATIC       11m
graphql-host-blackhole-solo-unused   ["graphql-host.blackhole.solo.unused"]   MESH_INTERNAL   STATIC       11m
NAME                                   AGE   ADDRESS
autogen.mgmtclus01.microapp.frontend   10s 
```

## Local Network Testing
```
kubectl run test-pod --image=curlimages/curl --restart=Never -n microapp --rm -it -- curl -v http://frontend:80 --connect-timeout 10
```