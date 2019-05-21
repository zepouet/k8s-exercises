# ETCDCTL

CLI for ETCD

## Etudiez les processus en cours

```
ps -ef| grep etcd 
```

## Travailler avec la nouvelle API

```
export ETCDCTL_API=3
```

## Liste des membres du cluster ETCD

```
etcdctl --endpoints=https://127.0.0.1:2379 \
    --cacert=/etc/kubernetes/ssl/etcd/ca.pem \
    --cert=/etc/kubernetes/ssl/etcd/node-...-master-1.pem \
    --key=/etc/kubernetes/ssl/etcd/node-...-master-1-key.pem \
	member list
```

## Faire un backup de ETCD

```
etcdctl --endpoints=https://127.0.0.1:2379 \
    --cacert=/etc/kubernetes/ssl/etcd/ca.pem \
    --cert=/etc/kubernetes/ssl/etcd/node-...-master-1.pem \
    --key=/etc/kubernetes/ssl/etcd/node-...-master-1-key.pem \
        snapshot save backup/etcd-snapshot-latest.db
```

## Vérifier les données du backup

```
etcdctl --endpoints=https://127.0.0.1:2379 \
    --cacert=/etc/kubernetes/ssl/etcd/ca.pem \
    --cert=/etc/kubernetes/ssl/etcd/node-...-master-1.pem \
    --key=/etc/kubernetes/ssl/etcd/node-...-master-1-key.pem \
        snapshot status backup/etcd-snapshot-latest.db
```

## Supprimer les données de ETCD

```
# etcdctl --endpoints=https://127.0.0.1:2379 \
#    --cacert=/etc/kubernetes/ssl/etcd/ca.pem \
#    --cert=/etc/kubernetes/ssl/etcd/node-...-master-1.pem \
#    --key=/etc/kubernetes/ssl/etcd/node-...-master-1-key.pem \
#        del "" --prefix
```

## Restaurer les données de ETCD

```
# etcdctl --endpoints=https://127.0.0.1:2379 \
#    --cacert=/etc/kubernetes/ssl/etcd/ca.pem \
#    --cert=/etc/kubernetes/ssl/etcd/node-...-master-1.pem \
#    --key=/etc/kubernetes/ssl/etcd/node-...-master-1-key.pem \
#        snapshot restore backup/etcd-snapshot-latest.db
```

Puis déplacer le tout dans ETCD avant de redémarrer l'API Docker via **kubeadm**

```
mv members.default /var/lib/etcd/members
```


