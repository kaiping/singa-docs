---
layout: post
title: Checkpoint and Resume
category : docs
tags : [checkpoint, restore]
---
{% include JB/setup %}

SINGA checkpoints model parameters onto disk periodically according to user
configured frequency. By checkpointing model parameters, we can

  1. resume the training from the last checkpointing. For example, if
    the program crashes before finishing all training steps, we can continue
    the training using checkpoint files.

  2. use them to initialize a similar model. For example, the
    parameters from training a RBM model can be used to initialize
    a [deep auto-encoder]({{ BASE_PATH }}/docs/rbm) model.

## Configuration

Checkpointing is controlled by two configuration fields:

* `checkpoint_after`, start checkpointing after this number of training steps,
* `checkpoint_freq`, frequency of doing checkpointing.

For example,

    # job.conf
    checkpoint_after: 100
    checkpoint_frequency: 300
    ...

Checkpointing files are located at *WORKSPACE/checkpoint/stepSTEP-workerWORKERID.bin*.
For the above configuration, after training for 700 steps, there would be
two checkpointing files,

    step400-worker0.bin
    step700-worker0.bin

## Application - resuming training

We can resume the training from the last checkpoint (i.e., step 700) by,

    ./bin/singa-run.sh -conf JOB_CONF -resume

There is no change to the job configuration.

## Application - model initialization

We can also use the checkpointing file from step 400 to initialize
a new model by configuring the new job as,

    # job.conf
    checkpoint : "WORKSPACE/checkpoint/step400-worker0.bin"
    ...

If there are multiple checkpointing files for the same snapshot due to model
partitioning, all the checkpointing files should be added,

    # job.conf
    checkpoint : "WORKSPACE/checkpoint/step400-worker0.bin"
    checkpoint : "WORKSPACE/checkpoint/step400-worker1.bin"
    ...

The training command is the same as starting a new job,

    ./bin/singa-run.sh -conf JOB_CONF

{% comment %}
## Advanced user guide

Checkpointing is done in the [Worker class]({{ BASE_PATH }}/api/classsinga_1_1Worker.html).
Only `Param`s from the first group are dumped into
checkpointing files. For a `Param` object, its name, version and values are saved.
It is possible that the snapshot is separated
into multiple files because the neural net is partitioned into multiple workers.

The Worker's `InitLocalParam` function will initialize parameters from checkpointing files if the
`checkpoint` field is set. Otherwise it randomly initialize them using user
configured initialization method. The `Param` objects are matched based on name.
If a `Param` object is not configured with a name, `NeuralNet` class will automatically
create one for it based on the name of the layer.
The `checkpoint` can be set by users (Application 1) or by the `Resume` function
(Application 2) of the Trainer class, which finds the files for the latest
snapshot and add them to the `checkpoint` filed. It also sets the `step` field
of model configuration to the checkpoint step (extracted from file name).


### Caution

Both two applications must be taken carefully when `Param` objects are
partitioned due to model partitioning. Because if the training is done using 2
workers, while the new model (or continue training) is trained with 3 workers,
then the same original `Param` object is partitioned in different ways and hence
cannot be matched.

{% endcomment %}
