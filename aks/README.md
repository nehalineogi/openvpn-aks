# Architecture diagram

This architecture diagram represents an AKS Cluster with 3 Nodes, three namespaces (red, green and blue) and OpenVPN + Ubuntu pods running in each namespace

![alt text for image](../architecture-diagram/openvpn-architecture-AKS.png)

# Kubernetes deployment using kompose

#### Convert Docker Compose file to Kubernetes

Install kompose [using this link](https://kubernetes.io/docs/tasks/configure-pod-container/translate-compose-kubernetes/)

##### Using Kompose to generate kubernetes manifest file

        cd aks
        kompose -f ../docker/docker-compose.yaml convert

#### Deploy the kubernetes manifest file using either Docker Desktop with k8s or AKS Cluster

        kubectl create namespace openvpn
        kubectl apply -f openvpn-as-deployment.yaml
        kubectl get pods -o wide -n openvpn
        Modify service type to NodePort to run in docker desktop
        kubectl apply -f openvpn-as-service.yaml
        kubectl get pods,service -o wide -n openvpn

#### Docker Desktop Validataions

        kubectl get nodes -o wide
        NAME             STATUS   ROLES    AGE   VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE         KERNEL-VERSION                   CONTAINER-RUNTIME
        docker-desktop   Ready    master   39d   v1.19.3   192.168.65.4   <none>        Docker Desktop   5.4.72-microsoft-standard-WSL2   docker://20.10.5

        kubectl config get-contexts
        CURRENT   NAME             CLUSTER          AUTHINFO         NAMESPACE
        *         docker-desktop   docker-desktop   docker-desktop

        kubectl get pods,service -o wide -n openvpn
        NAME                              READY   STATUS    RESTARTS   AGE    IP          NODE             NOMINATED NODE   READINESS GATES
        pod/openvpn-as-7c49767c46-2qqcd   1/1     Running   0          8m4s   10.1.0.34   docker-desktop   <none>           <none>

        NAME                 TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)                                       AGE     SELECTOR
        service/openvpn-as   NodePort   10.106.111.8   <none>        943:30517/TCP,9443:30974/TCP,1194:30437/TCP   2m37s   io.kompose.service=openvpn-as

#### AKS Cluster Validations

        nehali@nn-linux-dev:~$ kubectl get pod,service -n openvpn
        NAME READY STATUS RESTARTS AGE
        pod/openvpn-as-778b9f859-8ckc5 1/1 Running 0 7d21h
        pod/ubuntu 1/1 Running 0 5d18h

        NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S)
        AGE
        service/openvpn-as LoadBalancer 10.0.184.90 52.190.43.68 943:31663/TCP,9443:31058/TCP,1194:32258/TCP 6d19h

#### Admin Access to OpenVPN management UI via Azure VNET VM:

Configure VPN and user using admin access https://10.240.0.76:943/admin

# Connect using OpenVPN Client

Download OpenVPN Client file and connect

# Run tests

#### From OpenVPN Client

    kubectl exec -it openvpn-as-778b9f859-8ckc5 -n openvpn sh


     ifconfig eth5
    eth5: flags=4163<UP,BROADCAST,RUNNING,MULTICAST> mtu 1500
    inet **172.27.225.10** netmask 255.255.255.0 broadcast 172.27.225.255
    inet6 fe80::d9b2:eb5a:4d72:3918 prefixlen 64 scopeid 0xfd<compat,link,site,host>
    ether 00:ff:96:aa:71:26 (Ethernet)
    RX packets 0 bytes 0 (0.0 B)
    RX errors 0 dropped 0 overruns 0 frame 0
    TX packets 0 bytes 0 (0.0 B)
    TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0

    route -n | grep 10.204
    route -n | grep 10.240
    10.240.0.0 172.27.225.1 255.255.0.0 U 0 0 0 eth5

    ping 10.240.0.12
    PING 10.240.0.12 (10.240.0.12) 56(84) bytes of data.
    64 bytes from 10.240.0.12: icmp_seq=1 ttl=61 time=25.2 ms
    64 bytes from 10.240.0.12: icmp_seq=2 ttl=61 time=24.3 ms
    64 bytes from 10.240.0.12: icmp_seq=3 ttl=61 time=133 ms
    64 bytes from 10.240.0.12: icmp_seq=4 ttl=61 time=24.8 ms
    ^C
    --- 10.240.0.12 ping statistics ---
    5 packets transmitted, 4 received, 20% packet loss, time 4011ms
    rtt min/avg/max/mdev = 24.314/51.837/133.071/46.901 ms

#### From Ubuntu container

    nehali@nn-linux-dev:~$ kubectl exec -it ubuntu -n openvpn sh
    kubectl exec [POD] [COMMAND] is DEPRECATED and will be removed in a future version. Use kubectl kubectl exec [POD] -- [COMMAND] instead.

    ifconfig

    eth0: flags=67<UP,BROADCAST,RUNNING> mtu 1500
    inet 10.240.0.12 netmask 255.255.0.0 broadcast 0.0.0.0
    ether 12:c1:ba:8e:00:d0 txqueuelen 1000 (Ethernet)
    RX packets 11218 bytes 43497147 (43.4 MB)
    RX errors 0 dropped 0 overruns 0 frame 0
    TX packets 8978 bytes 919839 (919.8 KB)
    TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0

    lo: flags=73<UP,LOOPBACK,RUNNING> mtu 65536
    inet 127.0.0.1 netmask 255.0.0.0
    loop txqueuelen 1000 (Local Loopback)
    RX packets 14 bytes 1176 (1.1 KB)
    RX errors 0 dropped 0 overruns 0 frame 0
    TX packets 14 bytes 1176 (1.1 KB)
    TX errors 0 dropped 0 overruns 0 carrier 0 collisions 0

    route -n

    Kernel IP routing table
    Destination Gateway Genmask Flags Metric Ref Use Iface
    0.0.0.0 169.254.1.1 0.0.0.0 UG 0 0 0 eth0
    10.240.0.99 10.240.0.76 255.255.255.255 UGH 0 0 0 eth0
    169.254.1.1 0.0.0.0 255.255.255.255 UH 0 0 0 eth0
    172.27.225.0 10.240.0.76 255.255.255.0 UG 0 0 0 eth0

    tcpdump -ni eth0 icmp

    tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
    listening on eth0, link-type EN10MB (Ethernet), capture size 262144 bytes
    15:52:39.528269 IP 10.240.0.76 > 10.240.0.12: ICMP echo request, id 14047, seq 1, length 64
    15:52:39.528320 IP 10.240.0.12 > 10.240.0.76: ICMP echo reply, id 14047, seq 1, length 64
    15:52:40.528553 IP 10.240.0.76 > 10.240.0.12: ICMP echo request, id 14047, seq 2, length 64
    15:52:40.528590 IP 10.240.0.12 > 10.240.0.76: ICMP echo reply, id 14047, seq 2, length 64
    15:52:41.595468 IP 10.240.0.76 > 10.240.0.12: ICMP echo request, id 14047, seq 3, length 64

# Outbound Connectivity

Currently requires adding port forwarding on the openVPN server

Example: (Note: 172.27.225.10 is the static IP address of the OpenVPN Client)

        iptables -A PREROUTING -t nat -i eth0 -p tcp --dport 8080 -j DNAT --to 172.27.225.10:8000
