# consul集群搭建

```sh
consul agent -data-dir /tmp/node0 -node=node0 -bind=192.168.64.59 -datacenter=dc1 -ui -client=192.168.64.59 -server -bootstrap-expect 1

consul agent -data-dir /tmp/node1 -node=node1 -bind=192.168.64.94 -datacenter=dc1 -ui

consul agent -data-dir /tmp/node2 -node=node2 -bind=192.168.64.249 -datacenter=dc1 -ui -client=192.168.64.249

consul join 192.168.64.59

consul members -rpc-addr=192.168.64.59:8400

consul join -rpc-addr=192.168.64.249:8400  192.168.64.59
```

```shell
agent -data-dir /tmp/node0 -node=node0 -bind=192.168.64.59 -datacenter=dc1 -ui -server -bootstrap-expect 1
```



```sh
consul agent -server -bootstrap-expect 3 -data-dir /tmp/consul -node 192.168.64.59-datacenter dc1 –ui

consulagent -server -bootstrap-expect 3 -data-dir /tmp/consul -node 192.168.64.94-datacenter dc1 –ui

consulagent -server -bootstrap-expect 3 -data-dir /tmp/consul -node 192.168.64.249-datacenter dc1 -ui

consul join 192.168.64.59
consul join 192.168.64.59

consul operator raft list-peers
```



```sh
consul agent -server -bind 192.168.64.59 -data-dir /tmp/consul -client 192.168.64.59 -ui -datacenter dc1 -node 192.168.64.59

consul agent -server -bind 192.168.64.94 -data-dir /tmp/consul -client 192.168.64.94 -ui -datacenter dc1 -node 192.168.64.94

consul agent -server -bind 192.168.64.249 -data-dir /tmp/consul -client 192.168.64.249 -ui -datacenter dc1 -node 192.168.64.249

consul join -rpc-addr=192.168.64.94:8400  192.168.64.59

consul join -rpc-addr=192.168.64.249:8400  192.168.64.59

```

