# Application Wordpress + Mysql

Compléter les fichiers :
- wordpress-service.yml
- wordpress-deployment.yml
- mysql-service.yml
- mysql-deployment.yml

# Exposer WP via un Ingress

Compléter l'unique fichier 
- ingress.yml

# Gérer la persistence

Compléter les fichiers :
- mysql-pvc.yml
- wordpress-pvc.yml

Ajouter le bloc pour chaque container wordpress et mysql
```
spec: 
  template:
    spec:
      containers:
      - name: # wordpress or mysql
        # ...
        volumeMounts:
        - name: ___________
          mountPath: ___________
        volumes:
        - name: ___________
          persistentVolumeClaim: 
            claimName: ___________
```
