
说下```Dubbo```的负载均衡算法


### ```RandomLoadBalance```

加权随机算法，为dubbo的负载均衡算法；按照设置的权重，利用随机生成的随机数随机负载到服务提供者


### ```LeastActiveLoadBalance```

最小活跃数负载均衡算法，活跃调用数越小，表明该服务提供者效率越高


### ```ConsistentHashLoadBalance```

一致性hash算法


### ```RoundRobinLoadBalance```

轮询算法