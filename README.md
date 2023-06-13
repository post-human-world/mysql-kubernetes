# MySQL High availability cluster

## MySQL Operator

Install MySQL operator and create namespace

```bash
# version: 8.0.33-2.0.10
helm repo add mysql-operator https://mysql.github.io/mysql-operator/
helm repo update
helm install mysql-operator mysql-operator/mysql-operator --namespace mysql-cluster --create-namespace
```

MySQL InnoDBCluster need enough privilege to access namespace and manage router, backup, cluster modification.

```bash
kubectl apply -f rbac.yml
```

Create backup volume and InnoDBCluster. Remember to watch statefulset log and check whether it shows you dont have enough permission, normally i already gave enough privilege, but i am not sure if it will change in later version.

```bash
kubectl apply -f backup-pvc.yml
kubectl apply -f mysql.yml
```

Connect to InnoDBCluster

```bash
kubectl run --rm -it myshell --image=container-registry.oracle.com/mysql/community-operator -- mysqlsh root@mysql-innodb.mysql-cluster:3306  --sql
# If you don't see a command prompt, try pressing enter.
# ****************************** # <-pwd: <password-of-root-user-jecawe>
```

### Conclusion

The main disadvantage of MySQL Operator is it needs long time to delete cluster. In some cases, you have to clean its finalizers even you had set correct settings. The reason is deletion of secret before pod terminating.

If you wanna deploy a middle size cluster this should be a good choice. Comparing to Elastic Operator, MySQL Operator is lacks of custom settings, it can not separate primary and secondary pod settings.

## Mention

## Can't delete MySQL objects even applying force delete - MySQL Operator

[Not able to completely remove Kubernetes CustomResource](https://stackoverflow.com/questions/52009124/not-able-to-completely-remove-kubernetes-customresource)

```bash
# check what kind of resource is stucking
kubectl api-resources --verbs=list --namespaced -o name | xargs -n 1 kubectl get --show-kind --ignore-not-found -n mysql-cluster

# if it shows `innodbclusters.mysql.oracle.com/mysql-innodb` is pending forever, then run the command below
kubectl patch innodbclusters.mysql.oracle.com/mysql-innodb  -p '{"metadata":{"finalizers":[]}}' --type=merge -n mysql-cluster

# if shows pod is terminating long time
kubectl patch pods mysql-innodb-0 -n mysql-cluster -p '{"metadata":{"finalizers":null}}'
```

You have to manually clean resource after you set finalizers to empty

> You can also control how and when garbage collection deletes resources that have owner references using Kubernetes finalizers - [Cascading deletion](https://kubernetes.io/docs/concepts/architecture/garbage-collection/#cascading-deletion)

## This example does not simulate ClusterSet - MySQL InnoDB ClusterSet

After i read [Deploy a stateful MySQL cluster on GKE](https://cloud.google.com/kubernetes-engine/docs/tutorials/stateful-workloads/mysql), i found MySQL InnoDBCluster should have ability to configure ClusterSet, however i don't have enough knowledge to complete.

- [MySQL InnoDB ClusterSet](https://dev.mysql.com/doc/mysql-shell/8.0/en/innodb-clusterset.html)
- [Deploying InnoDB ClusterSet](https://dev.mysql.com/doc/mysql-shell/8.0/en/innodb-clusterset-deploy.html)