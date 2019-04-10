# etcd �� �ֲ�ʽ��

## etcd ���
  
etcd�Ǹ߿��õķֲ�ʽ��ֵ(key-value)���ݿ�, ��go����ʵ��, etcdʵ����Raft�ֲ�ʽһ����Э��, ��౻������ΪKubernetes�Լ�΢����ķ��������. ��֮���Ƶ����Ż���Դ��zookeeper. ��java��д, ʵ����Paxos�ֲ�ʽһ����Э��.
ͬ����Tidb��Tikv���, Ҳ��rust����ʵ����һ����Ϊ��Ч��RaftЭ��. ������Tidb�Ļ�����ʩ.  
  
### CAP
* ǿһ����(Consistency)
* ������(Availability)
* �����ݴ���(Partition Tolerance)
  
etcd �� zookeeper ���������е�CP  

### ��������

```java  

export ETCDCTL_API=3

etcdctl --endpoints=192.168.1.124:2379 member list
etcdctl --endpoints=192.168.1.124:2379 put foo "Hello World!"
etcdctl --endpoints=192.168.1.124:2379 get foo
etcdctl --endpoints=192.168.1.124:2379 del key
etcdctl --endpoints=192.168.1.124:2379 get web --prefix
etcdctl --endpoints=192.168.1.124:2379 lock resource --ttl=5

```

### gRPCЭ��

* [gRPC API](https://coreos.com/etcd/docs/latest/learning/api.html)
* [gRPC concurrent API](https://coreos.com/etcd/docs/latest/dev-guide/api_concurrency_reference_v3.html)

#### gRPC API

* LeaseGrantRequest -> LeaseGrantResponse
* LeaseRevokeRequest -> LeaseRevokeResponse
* LeaseKeepAliveRequest -> LeaseKeepAliveResponse

#### gRPC concurrent API

* LockRequest -> LockResponse
* UnlockRequest -> UnlockResponse

### restЭ��

etcd֧��grpc-gateway, �ṩһ��rest����, ���ܵ�rest����֮��, �����gRPC����, ���͵�etcd-server, �ٽ�gRPC����ֵ�����rest response���ظ��ͻ���  
  
[Swagger](https://coreos.com/etcd/docs/latest/dev-guide/apispec/swagger/rpc.swagger.json)  

```java  

<<COMMENT
https://www.base64encode.org/
foo is 'Zm9v' in Base64
bar is 'YmFy'
COMMENT

curl -L http://localhost:2379/v3alpha/kv/put \
	-X POST -d '{"key": "Zm9v", "value": "YmFy"}'
# {"header":{"cluster_id":"12585971608760269493","member_id":"13847567121247652255","revision":"2","raft_term":"3"}}

curl -L http://localhost:2379/v3alpha/kv/range \
	-X POST -d '{"key": "Zm9v"}'
# {"header":{"cluster_id":"12585971608760269493","member_id":"13847567121247652255","revision":"2","raft_term":"3"},"kvs":[{"key":"Zm9v","create_revision":"2","mod_revision":"2","version":"1","value":"YmFy"}],"count":"1"}

# get all keys prefixed with "foo"
curl -L http://localhost:2379/v3alpha/kv/range \
	-X POST -d '{"key": "Zm9v", "range_end": "Zm9w"}'
# {"header":{"cluster_id":"12585971608760269493","member_id":"13847567121247652255","revision":"2","raft_term":"3"},"kvs":[{"key":"Zm9v","create_revision":"2","mod_revision":"2","version":"1","value":"YmFy"}],"count":"1"}

```

## raft ���

 * [raft animation](http://thesecretlivesofdata.com/raft/)  

## jetcd ���

jetcd��etcd�Ĺٷ�java�ͻ���, ʵ����etcd��gRPC��API, ��etcd-server����ͨ��. ���etcd-server��DNS SRV discovery, ���Խ���etcd�ķ�����, ��failover����.

```xml  

    <dependency>
        <groupId>com.coreos</groupId>
        <artifactId>jetcd-core</artifactId>
        <version>0.0.2</version>
    </dependency>

```

## Ӧ�� jetcd ʵ�ֲַ�ʽ��

### Lock����

1. ����LeaseGrantRequest ����һ������ttlʱ��� leaseId. LeaseGrantRequest(ttl) -> LeaseGrantResponse(leaseId)
2. ����LockRequest �����õ�һ�������leaseId�Լ�Ҫlock����Դ, �����������lock����. LockRequest(resource, leaseId) -> LockResponse(key)
3. ����ڶ����ɹ�, ��ô����һ��key, ˵����ס����Ӧ����Դ,��ת������4 ����ڶ�����ʱ, ˵�������Դ���ڱ�����������ס, ��ת������5
4. ����Ѿ��������, ��ô���ڷ���LeaseKeepAliveRequest(leaseId), ά��������Ļ��ʱ��, ������������ʧ��, ��ô��ttl��ʱ����ͷ���, ��ת������1
5. ��������Դ����������Դ��ס, ��ô����LeaseRevokeRequest(leaseId) ����, ���������������, Ȼ������ת������1

### Unlock����

1. ��ס��Դ��ʱ��᷵��һ��key, ͨ�����key����UnlockRequest(key)

### ʵ�ֵ�һЩϸ��

* gondor�����µ�EtcdNamedLocker
* ttl
* retries
* heartbeat interval

```java  
ttl, retries, heartbeat interval ��һ����������  
ttl = 8000
retries = 2
heartbeat interval = 2000

get lock |-----------ttl-------------|                                <---main thread 
         |--2s--|renew |--2s--|retry1|--2s--|retry2| release lock     <---heartbeat thread
         
         
         |---------------------------|����ģ���ڴ�ʱ���������           <---other module main thread
         
һ����ȷ������
ttl = 8000
retries = 2
heartbeat interval = 1000



get lock |-------------------------ttl---------------------------|                         <---main thread 
         |--1s--|renew |--1s--|retry1|--1s--|retry2|release lock                           <---heartbeat thread
         
         
         |-------------------------------------------------------|����ģ���ڴ�ʱ���������    <---other module main thread
```

## etcd ��Ⱥ������

etcd���������DNS+SRV  
  
```java  
# nslookup                     
> set type=srv
> etcd.nextop.cn

etcd.nextop.cn  service = 10 10 2379 etcd02.nextop.cn.
etcd.nextop.cn  service = 10 10 2379 etcd03.nextop.cn.
etcd.nextop.cn  service = 10 10 2379 etcd01.nextop.cn.
```

ʵ��jetcd��URIResolverLoader�ӿ��Լ�URIResolver�ӿ�, ��URIResolver�ӿڵ�ʵ����, Ӧ��jndi��dns discovery���ַ���˵Ķ�̨etcd-server

```java  

Hashtable<String, String> env = new Hashtable<>();
env.put("java.naming.provider.url", "dns:");
env.put("java.naming.factory.initial", "com.sun.jndi.dns.DnsContextFactory");
final DirContext context = new InitialDirContext(env);
Attributes attributes = context.getAttributes(authority, new String[]{"SRV"});
NamingEnumeration<?> it = attributes.get("srv").getAll();

while (it.hasMore()) {
    // 10 10 2379 etcd01.nextop.cn.
    String record = (String)it.next();
    
}
				
```

## jetcd gRPC failover��ԭ��

1. ѡȡ��������Ķ��host, port�е�һ�����ӷ�������
2. ���������������ʧЧ��, ѡȡ��һ��host port�������ӷ�������
3. ����������ʧЧ��, ����NameResolver.refresh, ���½���dns�ĵ�ַ�����Чhost port, ����ִ�в���1

## references
 * [etcd](https://github.com/etcd-io/etcd/blob/master/Documentation/docs.md)
 * [jetcd](https://github.com/etcd-io/jetcd)  
 * [raft raw paper](https://ramcloud.atlassian.net/wiki/download/attachments/6586375/raft.pdf)  
 * [raft paper chinese version](https://github.com/maemual/raft-zh_cn/blob/master/raft-zh_cn.md)  
 * [raft resources](https://raft.github.io/)  
 * [raft animation](http://thesecretlivesofdata.com/raft/)  
 * [raft paper shorter version](https://www.usenix.org/conference/atc14/technical-sessions/presentation/ongaro)  
 * [raft blog](http://www.thinkingyu.com/articles/Raft/)  