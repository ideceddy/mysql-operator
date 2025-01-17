The test suite of MySQL Operator for Kubernetes
=============================

Requirements:
0) python3
1) module: kubernetes
2) module: mysql-connector-python (CAUTION! not mysql-connector)


The test-suite is configured in three stages:
0) defaults
1) environment variables
2) command-line options

The defaults may be overridden by envars, in turn, they can be overridden by cmd-line options.

Ad 0) defaults
All defaults are located in
./tests/setup/defaults.py

Ad 1) envars
The following envars are supported:
OPERATOR_TEST_REGISTRY
OPERATOR_TEST_REPOSITORY

OPERATOR_TEST_IMAGE_NAME
OPERATOR_TEST_EE_IMAGE_NAME
OPERATOR_TEST_VERSION_TAG
OPERATOR_TEST_PULL_POLICY
OPERATOR_TEST_GR_IP_WHITELIST

OPERATOR_TEST_SKIP_OCI
OPERATOR_TEST_BACKUP_OCI_APIKEY_PATH
OPERATOR_TEST_RESTORE_OCI_APIKEY_PATH
OPERATOR_TEST_BACKUP_OCI_BUCKET

Ad 2) command-line options
--env=[k3d|minikube]
    set the k8s environment

--kube-version
    set the kubernetes version to use, supported for minikube only
    if not set, the default depends on the installed minikube

--nodes
    points out the nodes to use, supported for minikube only
    if not set, any available will be used

--verbose|-v
    verbose logs

-vv
    more verbose

-vvv
    even more verbose

--debug|-d
    to work with py debugger

--trace|-t
    enable tracer

--nosetup|--no-setup
    disable setup of the environment and creation of cluster / an existing cluster will be used
    CAUTION! if not set the default cluster will be deleted (depending on chosen k8s environment - k3d or minikube)

--noclean|--no-clean
    Do not delete the cluster after the tests completed. By default it is deleted.

--load
    obsolete! used to load images
    at the moment the use of a local registry is strongly recommended
    that option probably will be removed in the future version

--nodeploy|--no-deploy
    do not deploy operator
    by default there is used operator that is generated in the method:
    BaseEnvironment.deploy_operator, file ./tests/utils/ote/base.py

--dkube
    more verbose logging of kubectl operations

--doperator
    set the operator debug level to 3

--mount-operator|-O
    mount operator sources in the mysql-operator pod, very useful to test patches without building an image
    the sources are drawn from
        .${src_root}/mysqloperator
    where tests run from
        .${src_root}/tests
    according to our standard git repo structure

--registry
    set the images registry, e.g. registry.localhost:5000

--registry-cfg
    supported only for k3d, the path to a registry config
    if k3d is used and a registry is set, but this path is not set, then the cfg file will
    be generated according to the following template:
mirrors:
  $registry:
    endpoint:
      - http://$registry

e.g.
mirrors:
  registry.localhost:5000:
    endpoint:
      - http://registry.localhost:5000

--repository
    the image repository, the default value is "mysql"

--operator-tag
    set the operator tag, e.g. 8.0.28-2.0.3 or latest, by default it is the currently developed version

--operator-pull-policy=[Never|IfNotPresent|Always]
    set the pull policy, the default can be found in defaults.py

--skip-oci
    force to skip all OCI tests even if OCI is properly configured, by default it is false

--oci-backup-apikey-path
--oci-restore-apikey-path
--oci-backup-bucket
    used to configure OCI, by default all are empty, which results in skipping tests related to OCI

arguments:
    gtest style test filter like
        include1:include2:-exclude1:exclude2

e.g.
e2e.mysqloperator.cluster.*
e2e.mysqloperator.cluster.cluster_badspec_t.*
e2e.mysqloperator.cluster.*:-e2e.mysqloperator.cluster.cluster_badspec_t.*

It may also be a full path to a test case or a list of test cases:
e2e.mysqloperator.backup.dump_t.DumpInstance
e2e.mysqloperator.cluster.cluster_enterprise_t.ClusterEnterprise e2e.mysqloperator.cluster.cluster_t.TwoClustersOneNamespace

To run all tests use:
e2e.*
e2e.mysqloperator.*
or just pass nothing


Everything runs by the script ./tests/run. It requires python3. Go to the ./tests directory and execute e.g.
CAUTION! To avoid deleting an existing (default) cluster use option --nosetup.


```sh
./run --env=minikube e2e.mysqloperator.backup.dump_t.DumpInstance
```

It will delete the existing default minikube cluster and set up a brand new one, then run the backup-dump-instance test case.


```sh
./run --env=k3d -vvv -t --dkube --doperator e2e.mysqloperator.cluster.cluster_t.Cluster1Defaults
```

It will delete the existing default k3d cluster. Then It will set up a brand new default k3d cluster and
run tests against the 1-instance default cluster. The logs will be very verbose.



```sh
./run --env=minikube --registry=registry.localhost:5000 --repository=qa e2e.mysqloperator.cluster.cluster_t.Cluster3Defaults
```

It will set up a brand new minikube cluster (the previous one will be deleted) and run tests against the 3-instances default
cluster. The operator will pull images from registry.localhost:5000/qa.


```sh
./run --env=k3d --registry=local.registry:5005 --registry-cfg=~/mycfg/k3d-registries.yaml --noclean \
    e2e.mysqloperator.cluster.cluster_enterprise_t.ClusterEnterprise
```

It will set up a brand new k3d cluster and run tests against the enterprise edition. The operator will
pull images from local.registry:5005/mysql.



The exact group of testcases can be run in the following way (here tabs are used to separate the names):

```sh
read -r -d '\t' TEST_SUITE << EOM
	e2e.mysqloperator.cluster.cluster_badspec_t.ClusterSpecAdmissionChecks
	e2e.mysqloperator.cluster.cluster_badspec_t.ClusterSpecRuntimeChecksCreation
	e2e.mysqloperator.cluster.cluster_t.Cluster1Defaults
	e2e.mysqloperator.cluster.cluster_t.ClusterCustomConf
	e2e.mysqloperator.cluster.cluster_upgrade_t.UpgradeToNext
	e2e.mysqloperator.cluster.initdb_t.ClusterFromDumpOCI
EOM

./run --env=k3d --mount-operator --noclean $TEST_SUITE
```

It will set up a brand new k3d cluster and run the chosen list of tests against the mounted (patched) operator. After tests
are completed the cluster will not be deleted.


```sh
./run --env=minikube --registry=myregistry.local:5000 -v e2e.*
```

It will set up a brand new minikube cluster (the previous one will be deleted) and run all tests matching e2e.* pattern.
The operator will pull images from myregistry.local:5000/mysql. The logs will be verbose (level 1).


```
./run --env=k3d --registry=registry.localhost:5000 --nosetup --nodeploy -vv
```

It will use an existing k3d cluster (nosetup) with an already deployed operator (nodeploy). The operator will pull
images from registry.localhost:5000/mysql. The logs will be verbose (level 2). As no filter was passed, all tests
will be executed.
