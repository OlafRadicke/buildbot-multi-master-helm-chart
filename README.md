BuildBot multi master helm chart
================================

About BuildBot
--------------

BuildBot is a CI/CD framwork. It is written in python but usable by all
programming languages. One spcele of BuildBot is, that he can support
multiple master in your setup. That is fundamental  for a high scalability
build farm. Read more about BuildBot on there own Site: 
***[buildbot.net](https://buildbot.net/)***

The concept
-----------

The communication between the the masters and the workers is opened of site
the worker.  The workers is not knowed that he talk with a load balancer. 
The masters is syncing his state about the database. So it is not important
which master is answer the worker. But for the master is all worker a
unique client. so If comes two difference worker with the same login
credentials, than it is a in the ayes of the master a duplicate.  That is
the reasons , that we can't have ReplicaSet of worker.

| ![multi-master-concept.png](multi-master-concept.png) |
|-------------------------------------------------------|

If we still want ReplicaSets for workers, we have to put magic in the login
credentials handling. At the moment this is not implemented. But I am open
for suggestions and discussions about this!


Get and configure the chart variables
-------------------------------------

Get variables of the chart:

```bash
helm show values ./helm-charts/buildbot/
```

Set variables of the chart:

Copy origine configuration:

```bash
helm show values ./helm-charts/buildbot/  > config.yaml
```

Modifide the configuration and than:

```bash
$ echo '{mariadbUser: user0, mariadbDatabase: user0db}' > config.yaml
$ helm install -f config.yaml stable/mariadb --generate-name

helm install \
    -f config.yaml \
    --create-namespace \
    -n my-testenv \
    my-test-setup ./helm-charts/buildbot/ 
```

In the file [values.yaml](helm-charts/buildbot/values.yaml) can you find the 
example configuration. It will be create a master with tow replicas and three
different worker.

Bebuging
--------

If you have trouble with this chart you can use the chart linter:

```bash
[or@augsburg02 buildbot-multi-master-helm-chart]$ helm lint ./helm-charts/buildbot/
==> Linting ./helm-charts/buildbot/

1 chart(s) linted, 0 chart(s) failed
```

Notes and knowed issues
-----------------------

* The volume mount for the master configuration is readonly but the BuildBot 
master needed the write exess to his configuration. So he get only a copy of 
the configuration from the configuration of the volume mount.  This copy will 
be created by the creating of the pod, only one time (without updates in his 
life time).
* No support for worker ReplicaSets in thes moment. See section *The concept*.

License note
------------

This code is under MIT license. See [LICENSE](LICENSE)
