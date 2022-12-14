

# How to Set Up a Demo etcd Cluster

- [How to Set Up a Demo etcd Cluster](#how-to-set-up-a-demo-etcd-cluster)
  * [Download And Install](#download-and-install)
  * [Run 1 Node Etcd](#run-1-node-etcd)
  * [Run 3 Node Etcd Cluster](#run-3-node-etcd-cluster)
    + [For Node 1](#for-node-1)
    + [For Node 2](#for-node-2)
    + [For Node 3](#for-node-3)
  
## Download And Install

```shell
#set env vars
export ETCD_VER=v3.4.20
export DOWNLOAD_URL=https://github.com/etcd-io/etcd/releases/download
export DOWNLOAD_URL=https://ghproxy.com/${DOWNLOAD_URL}

#download
echo "Downloading "${DOWNLOAD_URL}/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz
curl -sL ${DOWNLOAD_URL}/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz -o /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz

#install
mkdir -p /tmp/etcd-${ETCD_VER} && tar xzf /tmp/etcd-${ETCD_VER}-linux-amd64.tar.gz -C /tmp/etcd-${ETCD_VER} --strip-components=1

mkdir -p /usr/local/etcd/{bin,ssl} && mv /tmp/etcd-${ETCD_VER}/etcd* /usr/local/etcd/bin/
chmod +x /usr/local/etcd/bin/*

#clear tmp files
rm -rf /tmp/etcd-${ETCD_VER}

```

## Run 1 Node Etcd

```
export ETCD_TOKEN=token
export ETCD_CLUSTER_STATE=new
export ETCD_NAME_1=node-1
export ETCD_NODE_1=192.168.1.101
export ETCD_CLUSTER=${ETCD_NAME_1}=http://${ETCD_NODE_1}:2380

export THIS_NAME=${ETCD_NAME_1}
export THIS_IP=${ETCD_NODE_1}
export ENDPOINTS=$ETCD_NODE_1:2379

cat > /usr/lib/systemd/system/etcd.service << EOF
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
WorkingDirectory=/usr/local/etcd/
#User=etcd

# set GOMAXPROCS to number of processors
ExecStart=/bin/bash -c "GOMAXPROCS=$(nproc) /usr/local/etcd/bin/etcd \
  --name ${THIS_NAME} \
  --data-dir /usr/local/etcd/data.etcd \
  --listen-client-urls http://${THIS_IP}:2379 \
  --advertise-client-urls http://${THIS_IP}:2379 \
  --listen-peer-urls http://${THIS_IP}:2380 \
  --initial-advertise-peer-urls http://${THIS_IP}:2380 \
  --initial-cluster ${ETCD_CLUSTER} \
  --initial-cluster-token ${ETCD_TOKEN} \
  --initial-cluster-state ${ETCD_CLUSTER_STATE} \
  --heartbeat-interval=100 \
  --election-timeout=500 \
  --snapshot-count=5000 \
  --log-level info \
  --logger zap \
  --log-outputs stderr"

Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl start etcd

/usr/local/etcd/bin/etcdctl --endpoints=${ENDPOINTS} -w table member list
/usr/local/etcd/bin/etcdctl --endpoints=${ENDPOINTS} -w table endpoint status

```

## Run 3 Node Etcd Cluster

### For Node 1
```
export ETCD_TOKEN=tkn.0x0E
export ETCD_CLUSTER_STATE=new
export ETCD_NAME_1=node-1
export ETCD_NAME_2=node-2
export ETCD_NAME_3=node-3
export ETCD_NODE_1=192.168.11.101
export ETCD_NODE_2=192.168.11.102
export ETCD_NODE_3=192.168.11.103
export ETCD_CLUSTER=${ETCD_NAME_1}=http://${ETCD_NODE_1}:2380,${ETCD_NAME_2}=http://${ETCD_NODE_2}:2380,${ETCD_NAME_3}=http://${ETCD_NODE_3}:2380

#for node 1
export THIS_NAME=${ETCD_NAME_1}
export THIS_IP=${ETCD_NODE_1}

cat > /usr/lib/systemd/system/etcd.service << EOF
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
WorkingDirectory=/usr/local/etcd/
#User=etcd

# set GOMAXPROCS to number of processors
ExecStart=/bin/bash -c "GOMAXPROCS=$(nproc) /usr/local/etcd/bin/etcd \
  --name ${THIS_NAME} \
  --data-dir /k8s/etcd/data.etcd \
  --listen-client-urls http://${THIS_IP}:2379 \
  --advertise-client-urls http://${THIS_IP}:2379 \
  --listen-peer-urls http://${THIS_IP}:2380 \
  --initial-advertise-peer-urls http://${THIS_IP}:2380 \
  --initial-cluster ${ETCD_CLUSTER} \
  --initial-cluster-token ${ETCD_TOKEN} \
  --initial-cluster-state ${ETCD_CLUSTER_STATE} \
  --heartbeat-interval=100 \
  --election-timeout=500 \
  --snapshot-count=5000 \
  --log-level info \
  --logger zap \
  --log-outputs stderr"

Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```

### For Node 2
```
export ETCD_TOKEN=tkn.0x0E
export ETCD_CLUSTER_STATE=new
export ETCD_NAME_1=node-1
export ETCD_NAME_2=node-2
export ETCD_NAME_3=node-3
export ETCD_NODE_1=192.168.11.101
export ETCD_NODE_2=192.168.11.102
export ETCD_NODE_3=192.168.11.103
export ETCD_CLUSTER=${ETCD_NAME_1}=http://${ETCD_NODE_1}:2380,${ETCD_NAME_2}=http://${ETCD_NODE_2}:2380,${ETCD_NAME_3}=http://${ETCD_NODE_3}:2380

#for node 1
export THIS_NAME=${ETCD_NAME_2}
export THIS_IP=${ETCD_NODE_2}

cat > /usr/lib/systemd/system/etcd.service << EOF
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
WorkingDirectory=/usr/local/etcd/
#User=etcd

# set GOMAXPROCS to number of processors
ExecStart=/bin/bash -c "GOMAXPROCS=$(nproc) /usr/local/etcd/bin/etcd \
  --name ${THIS_NAME} \
  --data-dir /k8s/etcd/data.etcd \
  --listen-client-urls http://${THIS_IP}:2379 \
  --advertise-client-urls http://${THIS_IP}:2379 \
  --listen-peer-urls http://${THIS_IP}:2380 \
  --initial-advertise-peer-urls http://${THIS_IP}:2380 \
  --initial-cluster ${CLUSTER} \
  --initial-cluster-token ${ETCD_TOKEN} \
  --initial-cluster-state ${ETCD_CLUSTER_STATE} \
  --heartbeat-interval=100 \
  --election-timeout=500 \
  --snapshot-count=5000 \
  --log-level info \
  --logger zap \
  --log-outputs stderr"

Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```


### For Node 3
```
export ETCD_TOKEN=tkn.0x0E
export ETCD_CLUSTER_STATE=new
export ETCD_NAME_1=node-1
export ETCD_NAME_2=node-2
export ETCD_NAME_3=node-3
export ETCD_NODE_1=192.168.11.101
export ETCD_NODE_2=192.168.11.102
export ETCD_NODE_3=192.168.11.103
export ETCD_CLUSTER=${ETCD_NAME_1}=http://${ETCD_NODE_1}:2380,${ETCD_NAME_2}=http://${ETCD_NODE_2}:2380,${ETCD_NAME_3}=http://${ETCD_NODE_3}:2380

#for node 3
export THIS_NAME=${ETCD_NAME_3}
export THIS_IP=${ETCD_NODE_3}

cat > /usr/lib/systemd/system/etcd.service << EOF
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target

[Service]
Type=notify
WorkingDirectory=/usr/local/etcd/
#User=etcd

# set GOMAXPROCS to number of processors
ExecStart=/bin/bash -c "GOMAXPROCS=$(nproc) /usr/local/etcd/bin/etcd \
  --name ${THIS_NAME} \
  --data-dir /k8s/etcd/data.etcd \
  --listen-client-urls http://${THIS_IP}:2379 \
  --advertise-client-urls http://${THIS_IP}:2379 \
  --listen-peer-urls http://${THIS_IP}:2380 \
  --initial-advertise-peer-urls http://${THIS_IP}:2380 \
  --initial-cluster ${CLUSTER} \
  --initial-cluster-token ${ETCD_TOKEN} \
  --initial-cluster-state ${ETCD_CLUSTER_STATE} \
  --heartbeat-interval=100 \
  --election-timeout=500 \
  --snapshot-count=5000 \
  --log-level info \
  --logger zap \
  --log-outputs stderr"

Restart=on-failure
LimitNOFILE=65536

[Install]
WantedBy=multi-user.target
EOF
```
