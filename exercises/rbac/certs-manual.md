## Exercice

Nous allons maintenant créer un "espace étanche" sur votre cluster pour l'utilisateur "treeptik".

### Génération des clés

- Se connecter en ssh au node master du cluster:
~~~
ssh -i <path du certificat> root@<ip du node master>
~~~

- Générez la clé privée du nouvel utilisateur:
~~~bash
openssl genrsa -out treeptik.key 2048
~~~

- Créez une demande de certificat (attribut CN : group et O : utilisateur) :
~~~bash
openssl req -new -key treeptik.key \
            -out treeptik.csr \
            -subj "/CN=treeptik-reader/O=treeptik"
~~~


- Validez la demande de certificat avec le certificat administrateur (ca.pem, ca-key.pem) :
~~~bash
sudo openssl x509 -req -in treeptik.csr \
                  -CA /etc/kubernetes/ssl/ca.pem \
                  -CAkey /etc/kubernetes/ssl/ca-key.pem \
                  -CAcreateserial \
                  -out treeptik.crt -days 500
~~~

ou bien si K8S 1.12

~~~bash
sudo openssl x509 -req -in treeptik.csr \
                  -CA /etc/kubernetes/ssl/ca.crt \
                  -CAkey /etc/kubernetes/ssl/ca.key \
                  -CAcreateserial \
                  -out treeptik.crt -days 500
~~~


### Création Namespace et utilisateur dans K8S

Créez le namespace sur lequel l'utilisateur treeptik pourra agir:
~~~bash
kubectl create namespace treeptik-namespace
~~~

Créez le nouvel utilisateur dans kubernetes:
~~~bash
kubectl config set-credentials treeptik \
               --client-certificate=/root/treeptik.crt \
               --client-key=/root/treeptik.key
~~~

Créez le contexte associé à notre nouvel utilisateur:
~~~bash
kubectl config set-context treeptik-context \
                           --namespace=treeptik-namespace \
                           --user=treeptik \
                           --cluster=cluster.local
~~~

Tentez de lister les pods
~~~bash
kubectl --context=treeptik-context get pods
~~~

Vous allez avoir un message d'erreur comme quoi vous n'avez pas les droits requis.

### Créer le rôle et l'association

Créez l'association avec le fichier `role-reader.yaml`

~~~bash
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: treeptik-namespace
  name: deployment-manager-binding
rules:
- apiGroups: ["", "extensions", "apps"]
  resources: ["deployments", "replicasets", "pods"]
  verbs: ["get", "list", "watch"]
~~~

Créez l'association avec le fichier `role-reader-binding.yaml`

~~~bash
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: deployment-manager-binding
  namespace: treeptik-namespace # Nom du namespace
subjects:
- kind: User
  name: treeptik-reader
  apiGroup: ""
roleRef:
  kind: Role
  name: deployment-manager-binding
  apiGroup: ""
~~~

Ajoutez les au cluster:
~~~bash
kubectl create -f role-reader.yaml -f role-reader-binding.yaml
~~~

Lancez un deployment depuis le compte root par défaut:
Attention vous aurez un message de warning
~~~bash
kubectl run --image nginx mybeautifulpod -n treeptik-namespace
~~~

~~~bash
kubectl run --image nginx mybeautifulpod --generator=run-pod/v1 -n treeptik-namespace
~~~

Désormais vous pouvez afficher les ressources créees:
~~~bash
kubectl --context=treeptik-context get pods,deployment
~~~

Ou même changez de context pour celui nouvellement créé:
~~~bash
kubectl config use-context treeptik-context
kubectl get pods,deployment
~~~

Vérifiez que vous ne pouvez pas afficher de ressources du namespace "default" mais bien celles de votre namespace:
~~~bash
kubectl get pods -n default

kubectl auth can-i get deployments --namespace treeptik-namespace
kubectl auth can-i get deployments --namespace default
~~~

### Export vers un autre poste

Exportez la config dans un fichier :
~~~bash
kubectl config view --flatten > kubeconfig.cfg
~~~

Récupérez le fichier kubeconfig.cfg sur votre machine via scp ou autre.

Configurez la variable d'environnement KUBECONFIG pour la faire pointer sur kubeconfig.cfg:
~~~bash
export KUBECONFIG=<path>/kubeconfig.cfg
~~~

## Retour en mode ADMINISTRATEUR pour le TP suivant

```
kubectl config use-context kubernetes-admin@cluster.local
kubectl delete ns treeptik-namespace
```
