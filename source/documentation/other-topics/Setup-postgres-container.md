### Setup Postgres container

If you need to quickly create DB for test/dev, you can set up a pod in your namespace running postgres in a container. Deploying [Bitnami PostgreSQL][postgresql-chart] as Helm Chart is the easiest way to get started with PostgreSQL on Kubernetes. This chart bootstraps a PostgreSQL deployment on a Kubernetes cluster using the Helm package manager. 

The [django-reference-application][django-app] uses Bitnami PostgreSQL Helm Chart to add a postgres instance, you can use the same Chart to setup postgres in your namespace.

> Note: Even though we are going to install a database within the Kubernetes cluster, it is recommended to use a database as a service offering such as [AWS RDS](https://aws.amazon.com/rds/) if running in production.

##### Requirements
It is assumed you have the following:

 - You have [created an environment for your application][env-create]
 - You have installed [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/) on your local machine.
 - You have [Authenticated][auth-to-cluster] to the cloud-platform-live-1 cluster.
 - You have configured [Helm and Tiller][using-helm]. 

##### Set up
First copy [values.yaml][postgresql-values] file in to your working directory. You now have a [Bitnami PostgreSQL][postgresql-chart] Helm Chart "values.yaml" file. If you need to create your own values for postgresqlUsername,postgresqlPassword and postgresqlDatabase you can update the `values.yaml` or provide those as an argument on our installation command, the `set postgresql values` overwrites the value stored in `value.yaml` file.

Run the following (replacing the `YourName` with your own name and `env-name` with your environment name:

        $ helm install --name <YourName> -f values.yaml stable/postgresql \
          --namespace <env-name> \
          --set postgresqlUsername=postgres,postgresqlPassword=secretpassword,postgresqlDatabase=my-database \
          --tiller-namespace <env-name>

##### Viewing your PostgreSQL DB

You can now check PostgreSQL Helm Chart is deployed sucessfully: 

    $ kubectl get pods --namespace <env-name>

If the Installation was successful you should be seeing something similar to the below:

```
NAME                              READY     STATUS    RESTARTS   AGE
<YourName>-postgresql-0           1/1       Running   0          39m
```
You should have a postgres pod with the status **running**. You can also check the logs of the PostgreSQL pod: 

    $ kubectl --namespace <env-name> logs <YourName>-postgresql-0 

If the PostgreSQL setup was successful you should be seeing tail of log as below:

```
 12:49:28.02 INFO  ==> ** PostgreSQL setup finished! **
``` 

##### Accessing your PostgreSQL DB

PostgreSQL can be accessed via port 5432 on the following DNS name from within your cluster:

    <YourName>-postgresql.<env-name>.svc.cluster.local - Read/Write connection

The `postgresqlPassword` you have set will be stored as a secret in your namespace, to get the password for "postgres" run:

    export POSTGRES_PASSWORD=$(kubectl get secret --namespace <env-name> <YourName>-postgresql -o jsonpath="{.data.postgresql-password}" | base64 --decode)


To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace <env-name> svc/<YourName>-postgresql 5432:5432 &
    PGPASSWORD="$POSTGRES_PASSWORD" psql --host 127.0.0.1 -U postgres -d my-database -p 5432


Congratulations on getting this far. If all went well your postgresql pod is now deployed and you could connect to your database from outside the cluster.


[env-create]: tasks.html#creating-a-cloud-platform-environment
[auth-to-cluster]: tasks.html#authentication
[django-app]: tasks.html#deploying-an-application-to-the-cloud-platform-with-helm
[using-helm]: tasks.html#using-helm
[postgresql-chart]: https://github.com/helm/charts/tree/master/stable/postgresql
[postgresql-values]: https://github.com/ministryofjustice/cloud-platform-reference-app/blob/master/helm_deploy/django-app/charts/postgresql/values.yaml