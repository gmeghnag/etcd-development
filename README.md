# etcd-development
---

### Install
Single-command etcd cluster setup based on [hypershift](https://github.com/openshift/hypershift).
~~~
oc apply -k https://github.com/gmeghnag/etcd-development
~~~

### Interact
- Check endpoint status:
~~~
oc exec -n etcd-development etcd-0 -c etcd -- sh -c 'etcdctl --endpoints="https://etcd-0.etcd-discovery.${NAMESPACE}.svc:2379,https://etcd-1.etcd-discovery.${NAMESPACE}.svc:2379,https://etcd-2.etcd-discovery.${NAMESPACE}.svc:2379" --cacert=/etc/etcd/tls/etcd-ca/ca.crt --cert=/etc/etcd/tls/client/etcd-client.crt --key=/etc/etcd/tls/client/etcd-client.key endpoint status -w table'
~~~
- Check endpoint health:
~~~
oc exec -n etcd-development etcd-0 -c etcd -- sh -c 'etcdctl --endpoints="https://etcd-0.etcd-discovery.${NAMESPACE}.svc:2379,https://etcd-1.etcd-discovery.${NAMESPACE}.svc:2379,https://etcd-2.etcd-discovery.${NAMESPACE}.svc:2379" --cacert=/etc/etcd/tls/etcd-ca/ca.crt --cert=/etc/etcd/tls/client/etcd-client.crt --key=/etc/etcd/tls/client/etcd-client.key endpoint health -w table'
~~~
- Check member list:
~~~
oc exec -n etcd-development etcd-0 -c etcd -- sh -c 'etcdctl --endpoints="https://etcd-0.etcd-discovery.${NAMESPACE}.svc:2379,https://etcd-1.etcd-discovery.${NAMESPACE}.svc:2379,https://etcd-2.etcd-discovery.${NAMESPACE}.svc:2379" --cacert=/etc/etcd/tls/etcd-ca/ca.crt --cert=/etc/etcd/tls/client/etcd-client.crt --key=/etc/etcd/tls/client/etcd-client.key member list -w table'
~~~
- Take a snapshot and copy it to your local directory:
~~~
oc exec -n etcd-development etcd-0 -c etcd -- sh -c 'etcdctl --cacert=/etc/etcd/tls/etcd-ca/ca.crt --cert=/etc/etcd/tls/client/etcd-client.crt --key=/etc/etcd/tls/client/etcd-client.key snapshot save /tmp/snapshot.db'
oc exec -n etcd-development etcd-0 -c etcd -- sh -c 'cat /tmp/snapshot.db' > snapshot.db
~~~
