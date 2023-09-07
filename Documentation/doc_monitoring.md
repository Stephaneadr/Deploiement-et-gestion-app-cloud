Les services de monitoring sont accéssible via la commande :
` sudo kubectl get svc `

Cela affichera la liste suivante : 
```
NAME                                     TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                     
kubernetes                               ClusterIP   10.43.0.1       <none>        443/TCP                     
vault-internal                           ClusterIP   None            <none>        8200/TCP,8201/TCP           
vault-agent-injector-svc                 ClusterIP   10.43.172.9     <none>        443/TCP                     
vault                                    ClusterIP   10.43.28.150    <none>        8200/TCP,8201/TCP           
my-prometheus-alertmanager-headless      ClusterIP   None            <none>        9093/TCP                    
my-prometheus-alertmanager               ClusterIP   10.43.28.169    <none>        9093/TCP                    
my-prometheus-server                     ClusterIP   10.43.164.127   <none>        80/TCP                      
my-prometheus-kube-state-metrics         ClusterIP   10.43.163.97    <none>        8080/TCP                    
my-prometheus-prometheus-pushgateway     ClusterIP   10.43.170.138   <none>        9091/TCP                    
my-prometheus-prometheus-node-exporter   ClusterIP   10.43.197.218   <none>        9100/TCP                    
prometheus-server-ext                    NodePort    10.43.220.252   <none>        80:30381/TCP                
my-grafana                               ClusterIP   10.43.120.89    <none>        80/TCP                      
grafana-ext                              NodePort    10.43.202.247   <none>        80:30391/TCP                
loki-backend-headless                    ClusterIP   None            <none>        3100/TCP,9095/TCP           
query-scheduler-discovery                ClusterIP   None            <none>        3100/TCP,9095/TCP           
loki-read-headless                       ClusterIP   None            <none>        3100/TCP,9095/TCP           
loki-memberlist                          ClusterIP   None            <none>        7946/TCP                    
loki-write-headless                      ClusterIP   None            <none>        3100/TCP,9095/TCP           
loki-write                               ClusterIP   10.43.91.40     <none>        3100/TCP,9095/TCP           
loki-backend                             ClusterIP   10.43.69.213    <none>        3100/TCP,9095/TCP           
loki-gateway                             ClusterIP   10.43.198.26    <none>        80/TCP                      
loki-read                                ClusterIP   10.43.8.220     <none>        3100/TCP,9095/TCP           
loki-canary                              ClusterIP   10.43.129.44    <none>        3500/TCP                    
kubelet                                  ClusterIP   None            <none>        10250/TCP,10255/TCP,4194/TCP

```


Le moteur de monitoring Prometheus est accessible via le lien : [http://172.16.238.14:30381]

La liste des différentes métriques fournies par Prometheus est consultable via le lien : 

Le serveur Grafana est accessible via le lien : [http://172.16.238.14:30391]

Le tableau de bord de monitoring du cluster est consultable via le lien : [http://172.16.238.14:30391/d/1bI4P-xiz/k3s-cluster-monitoring?orgId=1&refresh=1m]

La gestion des logs via Loki et Promtail n'est malheuresement pas fonctionnel.