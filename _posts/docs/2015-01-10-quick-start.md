---
layout: post
title: Quick Start
category : docs
tags : [installation, examples]
---
{% include JB/setup %}

## SINGA setup

Please refer to the
[installation]({{ BASE_PATH }}/docs/installation}) page
for guidance on installing SINGA.

### Starting Zookeeper

SINGA uses [zookeeper](https://zookeeper.apache.org/) to coordinate the
training.  Please make sure the zookeeper service is started before running
SINGA.

If you installed the zookeeper using our thirdparty script, you can
simply start it by:

    #goto top level folder
    cd  SINGA_ROOT
    ./bin/zk-service start

(`./bin/zk-service stop` stops the zookeeper).

Otherwise, if you launched a zookeeper by yourself but not used the
default port, please edit the `conf/singa.conf`:

    zookeeper_host: "localhost:YOUR_PORT"

## Running in standalone mode

Running SINGA in standalone mode is on the contrary of running it using cluster
managers like [Mesos](http://mesos.apache.org/) or [YARN](http://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.html).

{% comment %}
For standalone mode, users have to manage the resources manually. For
instance, they have to prepare a host file containing all running nodes.
There is no restriction on CPU and memory resources, hence SINGA consumes as much
CPU and memory resources as it needs.
{% endcomment %}

### Training on a single node

For single node training, one process will be launched to run SINGA at
local host. We train the [CNN model](http://papers.nips.cc/paper/4824-imagenet-classification-with-deep-convolutional-neural-networks) over the
[CIFAR-10](http://www.cs.toronto.edu/~kriz/cifar.html) dataset as an example.
The hyper-parameters are set following
[cuda-convnet](https://code.google.com/p/cuda-convnet/). More details is
available at [CNN example]({{ BASE_PATH }}/docs/cnn).


#### Preparing data and job configuration

Download the dataset and create the data shards for training and testing.

    cd examples/cifar10/
    make download
    make create

A training dataset and a test dataset are created under *cifar10-train-shard*
and *cifar10-test-shard* folder respectively. An *image_mean.bin* file is also
generated, which contains the feature mean of all images.

Since all code used for training this CNN model is provided by SINGA as
built-in implementation, there is no need to write any code. Instead, users just
execute the running script (*../../bin/singa-run.sh*) by providing the job
configuration file (*job.conf*). To code in SINGA, please refer to the
[programming guide]({{ BASE_PATH }}/docs/programming-guide).

#### Training without parallelism

By default, the cluster topology has a single worker and a single server.
In other words, neither the training data nor the neural net is partitioned.

The training is started by running:

    # goto top level folder
    cd ../../
    ./bin/singa-run.sh -conf examples/cifar10/job.conf


You can list the current running jobs by,

    ./bin/singa-console.sh list

    JOB ID    |NUM PROCS
    ----------|-----------
    24        |1

Jobs can be killed by,

    ./bin/singa-console.sh kill JOB_ID


Logs and job information are available in */tmp/singa-log* folder, which can be
changed to other folders by setting `log-dir` in *conf/singa.conf*.

{% comment %}
One worker group trains against one partition of the training dataset. If
*nworker_groups* is set to 1, then there is no data partitioning. One worker
runs over a partition of the model. If *nworkers_per_group* is set to 1, then
there is no model partitioning. More details on the cluster configuration are
described in the [System Architecture]() page.
{% endcomment %}

#### Asynchronous parallel training

    # job.conf
    ...
    cluster {
      nworker_groups: 2
      nworkers_per_procs: 2
      workspace: "examples/cifar10/"
    }

In SINGA, [asynchronous training]({{ BASE_PATH }}/docs/architecture) is enabled
by launching multiple worker groups. For example, we can change the original
*job.conf* to have two worker groups as shown above. By default, each
worker group has one worker. Since one process is set to contain two workers.
The two worker groups will run in the same process.  Consequently, they run
the in-memory [Downpour]({{ BASE_PATH }}/docs/frameworks) training framework.
Users do not need to split the dataset
explicitly for each worker (group); instead, they can assign each worker (group) a
random offset to the start of the dataset. The workers would run as on
different data partitions.

    # job.conf
    ...
    neuralnet {
      layer {
        ...
        sharddata_conf {
          random_skip: 5000
        }
      }
      ...
    }

The running command is:

    ./bin/singa-run.sh -conf examples/cifar10/job.conf

#### Synchronous parallel training

    # job.conf
    ...
    cluster {
      nworkers_per_group: 2
      nworkers_per_procs: 2
      workspace: "examples/cifar10/"
    }

In SINGA, [asynchronous training]({{ BASE_PATH }}/docs/architecture) is enabled
by launching multiple workers within one worker group. For instance, we can
change the original *job.conf* to have two workers in one worker group as shown
above. The workers will run synchronously
as they are from the same worker group. This framework is the in-memory
[sandblaster]({{ BASE_PATH }}/docs/frameworks).
The model is partitioned among the two workers. In specific, each layer is
sliced over the two workers.  The sliced layer
is the same as the original layer except that it only has `B/g` feature
instances, where `B` is the number of instances in a mini-batch, `g` is the number of
workers in a group. It is also possible to partition the layer (or neural net)
using [other schemes]({{ BASE_PATH }}/docs/neural-net).
All other settings are the same as running without partitioning

    ./bin/singa-run.sh -conf examples/cifar10/job.conf

### Training in a cluster

We can extend the above two training frameworks to a cluster by updating the
cluster configuration with:

    nworker_per_procs: 1

Every process would then create only one worker thread. Consequently, the workers
would be created in different processes (i.e., nodes). The *hostfile*
must be provided under *SINGA_ROOT/conf/* specifying the nodes in the cluster,
e.g.,

    logbase-a01
    logbase-a02

The running command is the same as for single node training:

    ./bin/singa-run.sh -conf examples/cifar10/job.conf

## Running with Mesos

*in working*...


## Where to go next

The [programming guide]({{ BASE_PATH }}/docs/programming-guide) pages will
describe how to submit a training job in SINGA.
