# kubernetes-experience\

1.) Create a PersistentVolume mysql-pv, its capacity should be 250Mi, set other parameters as per your preference.

2.) Create a PersistentVolumeClaim to request this PersistentVolume storage. Name it as mysql-pv-claim and request a 250Mi of storage. Set other parameter as per your preference.
3.) Create a deployment named mysql-deployment, use any mysql image as per your preference. Mount the PersistentVolume at mount path /var/lib/mysql.

4.) Create a NodePort type service named mysql and set nodePort to 30007.

5.) Create a secret named mysql-root-pass having a key pair value, where key is password and its value is YUIidhb667, create another secret named mysql-user-pass having some key pair values, where frist key is username and its value is kodekloud_top, second key is password and value is B4zNgHA7Ya, 
create one more secret named mysql-db-url, key name is database and value is kodekloud_db8

6.) Define some Environment variables within the container:
a) name: MYSQL_ROOT_PASSWORD, should pick value from secretKeyRef name: mysql-root-pass and key: password
b) name: MYSQL_DATABASE, should pick value from secretKeyRef name: mysql-db-url and key: database
c) name: MYSQL_USER, should pick value from secretKeyRef name: mysql-user-pass key key: username
d) name: MYSQL_PASSWORD, should pick value from secretKeyRef name: mysql-user-pass and key: password


#Below are the task to implement the above ask, code will change based on secret values and other paramater.

#Create Secrets
```sh
kubectl create secret generic mysql-root-pass --from-literal=password=YUIidhb667 
kubectl create secret generic mysql-user-pass --from-literal=username=kodekloud_top --from-literal=password=B4zNgHA7Ya 
kubectl create secret generic mysql-db-url --from-literal=database=kodekloud_db8 
```

```
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
spec:
  capacity:
    storage: 250Mi
  accessModes:
    - ReadWriteOnce
  hostPath: 
   path: /tmp

---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: mysql-pv-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 250Mi

---
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  type: NodePort
  ports:
    - targetPort: 3306
      port: 3306
      nodePort: 30007
  selector:
    app: mysql

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
  labels:
     app: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - image: mysql:5.6
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-root-pass
              key: password
        - name: MYSQL_DATABASE
          valueFrom:
            secretKeyRef:
              name: mysql-db-url
              key: database
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: mysql-user-pass
              key: username
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-user-pass
              key: password
        ports:
        - containerPort: 3306
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage
        persistentVolumeClaim:
          claimName: mysql-pv-claim
```
