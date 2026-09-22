# Lab 5 Evidence

## 1. Pipeline Links
* **Docker Hub Tags:** https://hub.docker.com/r/conflictserum/notes-app/tags
* **GitHub Actions:** https://github.com/basic-hbb/notes-app/actions

## 2. Experiment Answers

**Experiment 2 (Data Persistence): Is your note still there after deleting the db pod? Why? Which object holds the data?**
Yes, the note survived. The data is held in the PersistentVolume provisioned by the PersistentVolumeClaim (`db-data`), which exists independently of the ephemeral database pod. When the new database pod spun up, it automatically reattached to that same storage volume.

**Experiment 3 (Load Balancing): When hitting the scaled-up Service 8 times, how many distinct values did you see in the `served_by` field? What are they?**
There were 4 distinct values. These values correspond to the auto-generated hostnames of the 4 individual `web` pods. This proves the Kubernetes Service is actively load-balancing incoming requests evenly across all available replicas.

**Experiment 4 (Rolling Updates): Did old and new pods overlap? Was there any downtime?**
Yes, they overlapped, resulting in zero downtime. Kubernetes uses a rolling update strategy, waiting for a new pod to pass its `readinessProbe` before terminating an old pod. 
*(Note: During the rollback, the app didn't revert because Kubernetes fetched the `latest` tag again, which the CI/CD pipeline had just overwritten with the newest code. This demonstrates why pinning to specific `sha-` tags is safer for rollbacks than using `latest`!)*

## 3. Kubernetes Status
ismail@Mac notes-app % kubectl get all,pvc
NAME                       READY   STATUS    RESTARTS   AGE
pod/db-6c5c8947cd-l2mh9    1/1     Running   0          19m
pod/web-655dc84b44-lnnc8   1/1     Running   0          5m34s
pod/web-655dc84b44-t9js7   1/1     Running   0          5m40s

NAME          TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)    AGE
service/db    ClusterIP   10.96.255.71   <none>        5432/TCP   3h27m
service/web   ClusterIP   10.96.2.206    <none>        80/TCP     3h23m

NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/db    1/1     1            1           3h27m
deployment.apps/web   2/2     2            2           3h23m

NAME                             DESIRED   CURRENT   READY   AGE
replicaset.apps/db-6c5c8947cd    1         1         1       3h27m
replicaset.apps/web-5bfd7559bc   0         0         0       7m54s
replicaset.apps/web-655dc84b44   2         2         2       23m
replicaset.apps/web-6788b79595   0         0         0       3h23m

NAME                            STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   VOLUMEATTRIBUTESCLASS   AGE
persistentvolumeclaim/db-data   Bound    pvc-c146284f-8feb-4146-843c-0e390b3c69cd   1Gi        RWO            standard       <unset>                 3h27m