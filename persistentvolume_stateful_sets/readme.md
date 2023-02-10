# Persistent Volumes Demo

<a href="https://youtu.be/ZxC6FwEc9WQ" title="k8s-pv"><img src="https://i.ytimg.com/vi/ZxC6FwEc9WQ/hqdefault.jpg" width="20%" alt="k8s-pv" /></a>

## Container Storage

By default containers store their data on the file system like any other process.
Container file system is temporary and not persistent during container restarts
When container is recreated, so is the file system

```
# run postgres
docker run -d --rm -e POSTGRES_DB=postgresdb -e POSTGRES_USER=admin -e POSTGRES_PASSWORD=admin123 postgres:10.4

# enter the container 
docker exec -it <container-id> bash

# login to postgres
psql --username=admin postgresdb

#create a table
CREATE TABLE COMPANY(
   ID INT PRIMARY KEY     NOT NULL,
   NAME           TEXT    NOT NULL,
   AGE            INT     NOT NULL,
   ADDRESS        CHAR(50),
   SALARY         REAL
);

#show table
\dt

# quit 
\q
```

Restarting the above container and going back in you will notice `\dt` commands returning no tables.
Since data is lost.

Same can be demonstrated using Kubernetes

```
cd .\kubernetes\persistentvolume\

kubectl create ns postgres
kubectl apply -n postgres -f ./postgres-no-pv.yaml
kubectl -n postgres get pods 
kubectl -n postgres exec -it postgres-0 bash

# run the same above mentioned commands to create and list the database table

kubectl delete po -n postgres postgres-0

# exec back in and confirm table does not exist.
```

# Persist data Docker

```
docker volume create postgres
docker run -d --rm -v postgres:/var/lib/postgresql/data -e POSTGRES_DB=postgresdb -e POSTGRES_USER=admin -e POSTGRES_PASSWORD=admin123 postgres:10.4

# run the same tests as above and notice
```

# Persist data Kubernetes

```
kubectl create ns postgres

kubectl apply -f persistentvolume.yaml

kubectl apply -n postgres -f persistentvolumeclaim.yaml

kubectl apply -n postgres -f postgres-with-pv.yaml

kubectl -n postgres get pods

```


# Kubernetes client

## retrieve the postgres password in an eviornment variable
```
export POSTGRES_PASSWORD=$(kubectl get configmap postgres-config -n postgres -o jsonpath="{.data.POSTGRES_PASSWORD}")            
```

## use another Docker container to connect through the psql command:
```
kubectl run postgres-client --rm --tty -i --restart='Never' --image postgres:11 --env="PGPASSWORD=$POSTGRES_PASSWORD" -n postgres --command -- psql -h postgres -U admin postgresdb
```

```
INSERT INTO COMPANY (ID, NAME, AGE, ADDRESS, SALARY)
VALUES (2, 'Test Doe', 50, '456 Main Street', 100000);

INSERT INTO COMPANY (ID, NAME, AGE, ADDRESS, SALARY)
VALUES (3, 'Bob Johnson', 35, '789 Pine Boulevard', 55000);

INSERT INTO COMPANY (ID, NAME, AGE, ADDRESS, SALARY)
VALUES (4, 'Samantha Brown', 28, '246 Maple Drive', 40000);
```
