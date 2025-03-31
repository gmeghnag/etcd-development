~~~
oc new-project etcd-development
~~~
~~~
cat > server.ext <<EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth, clientAuth
subjectAltName = @alt_names

[alt_names]
DNS.1 = etcd-0.etcd-discovery.etcd-development.svc
DNS.2 = etcd-1.etcd-discovery.etcd-development.svc
DNS.3 = etcd-2.etcd-discovery.etcd-development.svc
DNS.4 = localhost
IP.1 = 127.0.0.1
EOF
~~~
~~~
openssl genrsa -out ca.key 2048
openssl req -x509 -new -nodes -key ca.key -subj "/CN=etcd-ca" -days 3650 -out ca.crt
~~~
~~~
openssl genrsa -out server.key 2048
openssl req -new -key server.key -subj "/CN=etcd-server" -out server.csr
openssl x509 -req -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt -days 3650 -extfile server.ext
~~~
~~~
openssl genrsa -out peer.key 2048
openssl req -new -key peer.key -subj "/CN=etcd-peer" -out peer.csr
openssl x509 -req -in peer.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out peer.crt -days 3650 -extfile server.ext
~~~
~~~
openssl genrsa -out etcd-client.key 2048
openssl req -new -key etcd-client.key -subj "/CN=etcd-client" -out etcd-client.csr
openssl x509 -req -in etcd-client.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out etcd-client.crt -days 3650 -extfile server.ext
~~~
~~~
oc create secret generic etcd-server-tls --from-file=server.crt --from-file=server.key --from-file=ca.crt
oc create secret generic etcd-peer-tls --from-file=peer.crt --from-file=peer.key --from-file=ca.crt
oc create secret generic etcd-client-tls --from-file=etcd-client.crt --from-file=etcd-client.key --from-file=ca.crt
~~~
~~~
oc create configmap etcd-ca --from-file=ca.crt
oc create configmap etcd-metrics-ca --from-file=ca.crt
~~~
~~~
curl -sk https://raw.githubusercontent.com/openshift/hypershift/3c58ea8935069eb29a7687d5ce4da8de21549db3/control-plane-operator/controllers/hostedcontrolplane/v2/assets/etcd/defrag-serviceaccount.yaml | oc create -f -
curl -sk https://raw.githubusercontent.com/openshift/hypershift/3c58ea8935069eb29a7687d5ce4da8de21549db3/control-plane-operator/controllers/hostedcontrolplane/v2/assets/etcd/defrag-role.yaml | oc create -f -
curl -sk https://raw.githubusercontent.com/openshift/hypershift/3c58ea8935069eb29a7687d5ce4da8de21549db3/control-plane-operator/controllers/hostedcontrolplane/v2/assets/etcd/defrag-rolebinding.yaml | oc create -f -
curl -sk https://raw.githubusercontent.com/openshift/hypershift/3c58ea8935069eb29a7687d5ce4da8de21549db3/control-plane-operator/controllers/hostedcontrolplane/v2/assets/etcd/discovery-service.yaml | oc create -f -
curl -sk https://raw.githubusercontent.com/openshift/hypershift/3c58ea8935069eb29a7687d5ce4da8de21549db3/control-plane-operator/controllers/hostedcontrolplane/v2/assets/etcd/service.yaml | oc create -f -
curl -sk https://raw.githubusercontent.com/openshift/hypershift/refs/heads/main/control-plane-operator/controllers/hostedcontrolplane/v2/assets/etcd/pdb.yaml | oc create -f -
~~~
~~~
curl -sk https://raw.githubusercontent.com/gmeghnag/etcd-development/refs/heads/main/k8s/etcd-development.StatefulSet.yaml | oc create -f -
~~~
