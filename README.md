# TF Graph Neural Network Samples
This repository is the code release corresponding to an article introducing
graph neural networks (GNNs) with feature-wise linear modulation ([Brockschmidt, 2019](#brockschmidt-2019)).
In the paper, a number of GNN architectures are discussed:
* Gated Graph Neural Networks (GGNN) ([Li et al., 2015](#li-et-al-2015)).
* Relational Graph Convolutional Networks (RGCN) ([Schlichtkrull et al., 2016](#schlichtkrull-et-al-2016)).
* Relational Graph Attention Networks (RGAT) - a generalisation of Graph Attention Networks ([Veličković et al., 2018](#veličković-et-al-2018)) to several edge types.
* Relational Graph Dynamic Convolution Networks (RGDCN) - a new variant of RGCN in which the weights of convolutional layers are dynamically computed.
* Graph Neural Networks with Feature-wise Linear Modulation (GNN-FiLM) - a new extension of RGCN with FiLM layers.

The results presented in the paper are based on the implementations of models
and tasks provided in this repository.

This code was tested in Python 3.6 with TensorFlow 1.13.1.
To install required packages, run `pip install -r requirements.txt`.

The code is maintained by the [Deep Program Understanding](https://www.microsoft.com/en-us/research/project/program/) project at Microsoft Research, Cambridge, UK. We are [hiring](https://www.microsoft.com/en-us/research/theme/ada/#!opportunities).

# Running
To train a model, it suffices to run `python train.py MODEL_TYPE TASK`, for
example as follows:
```
$ python train.py RGCN PPI
Loading task/model-specific default parameters from tasks/default_hypers/PPI_RGCN.json.
 Loading PPI train data from data/ppi.
 Loading PPI valid data from data/ppi.
Model has 699257 parameters.
Run PPI_RGCN_2019-06-26-14-33-58_17208 starting.
 Using the following task params: {"add_self_loop_edges": true, "tie_fwd_bkwd_edges": false, "out_layer_dropout_keep_prob": 1.0}
 Using the following model params: {"max_nodes_in_batch": 12500, "graph_num_layers": 3, "graph_num_timesteps_per_layer": 1, "graph_layer_input_dropout_keep_prob": 1.0, "graph_dense_between_every_num_gnn_layers": 10000, "graph_model_activation_function": "tanh", "graph_residual_connection_every_num_layers": 10000, "graph_inter_layer_norm": false, "max_epochs": 10000, "patience": 25, "optimizer": "Adam", "learning_rate": 0.001, "learning_rate_decay": 0.98, "momentum": 0.85, "clamp_gradient_norm": 1.0, "random_seed": 0, "hidden_size": 256, "graph_activation_function": "ReLU", "message_aggregation_function": "sum"}
== Epoch 1
 Train: loss: 77.42656 || Avg MicroF1: 0.395 || graphs/sec: 15.09 | nodes/sec: 33879 | edges/sec: 1952084
 Valid: loss: 68.86771 || Avg MicroF1: 0.370 || graphs/sec: 14.85 | nodes/sec: 48360 | edges/sec: 3098674
  (Best epoch so far, target metric decreased to 224302.10938 from inf. Saving to 'trained_models/PPI_RGCN_2019-06-26-14-33-58_17208_best_model.pickle')
[...]
```
An overview of options can be obtained by `python train.py --help`.

Note that task and model parameters can be overriden (note that every training
run prints their current settings) using the `--task-param-overrides` and
`--model-param-overrides` command line options, which take dictionaries in JSON
form.
So for example, to choose a different number of layers, 
`--model-param-overrides '{"graph_num_layers": 4}'` can be used.

Results of the training run will be saved as well in a directory (by default
`trained_models/`, but this can be set using the `--result_dir` flag).
Concretely, the following three files are created:
* `${RESULT_DIR}/${RUN_NAME}.log`: A log of the training run.
* `${RESULT_DIR}/${RUN_NAME}_best_model.pickle`: A dump of the model weights 
  achieving the best results on the validation set.

To evaluate a model, use the `test.py` script as follows on one of the
model dumps generated by `train.py`:
```
$ python test.py trained_models/PPI_RGCN_2019-06-26-14-33-58_17208_best_model.pickle
Loading model from file trained_models/PPI_RGCN_2019-06-26-14-33-58_17208_best_model.pickle.
Model has 699257 parameters.
== Running Test on data/ppi ==
 Loading PPI test data from data/ppi.
Loss 11.13117 on 2 graphs
Metrics: Avg MicroF1: 0.954
```
`python test.py --help` provides more options, for example to specify a different
test data set.
A run on the default test set can be be automatically triggered after training
using the `--run-test` option to `train.py` as well.

# Experimental Results
Experimental results reported in the accompanying article can be reproduced
using the code in the repository.
More precisely, `python run_ppi_benchs.py ppi_eval_results/` should
produce an ASCII rendering of Table 1 - note, however, that this will take
quite a while.
Similarly, `python run_qm9_benchs.py qm9_eval_results/` should
produce an ASCII rendering of Table 2 - this will take a very long time
(approx. 13 * 4 * 45 * 5 minutes, i.e., around 8 days), and
in practice, we used a different version of this parallelising the runs
across many hosts using Microsoft-internal infrastructure.

Note that the training script loads fitting default hyperparameters for
model/task combinations from `tasks/default_hypers/{TASK}_{MODEL}.json`.

# Models
Currently, five model types are implemented:
* `GGNN`: Gated Graph Neural Networks ([Li et al., 2015](#li-et-al-2015)).
* `RGCN`: Relational Graph Convolutional Networks ([Schlichtkrull et al., 2017](#schlichtkrull-et-al-2017)).
* `RGAT`: Relational Graph Attention Networks ([Veličković et al., 2018](#veličković-et-al-2018)).
* `RGDCN`: Relational Graph Dynamic Convolution Networks - a new variant of RGCN in which the weights of convolutional layers are dynamically computed.
* `GNN-FiLM`: Graph Neural Networks with Feature-wise Linear Modulation - a new extension of RGCN with FiLM layers.

# Tasks
New tasks can be added by implementing the `tasks.sparse_graph_task` interface.
This provides hooks to load data, create a task-specific output layers and
compute task-specific metrics.
The documentation in `tasks/sparse_graph_task.py` provides a detailed overview
of the interface. Currently, four tasks are implemented, exposing different
aspects.

## Citation networks
The `CitationNetwork` task (implemented in `tasks/citation_network_task.py`)
handles the Cora, Pubmed and Citeseer citation network datasets often used
in evaluation of GNNs ([Sen et al., 2008](#sen-et-al-2008)).
The implementation illustrates how to handle the case of transductive graph
learning on a single graph instance by masking out nodes that shouldn't be
considered.
You can call this by running `python train.py MODEL Cora` (or `Pubmed` or
`Citeseer` instead of `Cora`).

To run experiments on this task, you need to download the data from 
https://github.com/kimiyoung/planetoid/raw/master/data. By default, the
code looks for this data in `data/citation-networks`, but this can be changed
by using `--data-path "SOME/OTHER/DIR"`.

## PPI
The `PPI` task (implemented in `tasks/ppi_task.py`) handles the protein-protein
interaction task first described by [Zitnik & Leskovec, 2017](#zitnik-leskovec-2017).
The implementation illustrates how to handle the case of inductive graph
learning with node-level predictions.
You can call this by running `python train.py MODEL PPI`.

To run experiments on this task, you need to download the data from
https://s3.us-east-2.amazonaws.com/dgl.ai/dataset/ppi.zip. By default, the
code looks for this data in `data/ppi`, but this can be changed
by using `--data-path "SOME/OTHER/DIR"`.

### Current Results
Running `python run_ppi_benchs.py ppi_results/` should yield results looking
like this (on an NVidia V100):

| Model        | Avg. MicroF1      | Avg. Time  |
|--------------|-------------------|------------|
| GGNN         | 0.990 (+/- 0.001) |      432.6 |
| RGCN         | 0.989 (+/- 0.000) |      759.0 |
| GAT          | 0.989 (+/- 0.001) |      782.3 |
| GNN-Edge-MLP | 0.992 (+/- 0.001) |      479.2 |
| GNN-FiLM     | 0.992 (+/- 0.000) |      308.1 |

## QM9
The `QM9` task (implemented in `tasks/qm9_task.py`) handles the quantum chemistry
prediction tasks first described by [Ramakrishnan et al., 2014](#ramakrishnan-et-al-2014)
The implementation illustrates how to handle the case of inductive graph
learning with graph-level predictions.
You can call this by running `python train.py MODEL QM9`.

The data for this task is included in the repository in `data/qm9`, which just
contains a JSON representation of a pre-processed version of the dataset originally
released by [Ramakrishnan et al., 2014](#ramakrishnan-et-al-2014).

## VarMisuse
The `VarMisuse` task (implemented in `tasks/varmisuse_task.py`) handles the
variable misuse task first described by [Allamanis et al., 2018](#allamanis-et-al-2018).
Note that we do not fully re-implement the original model here, and so
results are not (quite) comparable with the results reported in the original
paper.
The implementation illustrates how to handle the case of inductive graph
learning with predictions based on node selection.
You can call this by running `python train.py MODEL VarMisuse`.

To run experiments on this task, you need to download the data from
https://aka.ms/iclr18-prog-graphs-dataset and unzip it.
By default, the code looks for this data in `data/varmisuse/`, but this can be 
changed by using `--data-path "SOME/OTHER/DIR"`.

# References

#### Allamanis et al., 2018
Miltiadis Allamanis, Marc Brockschmidt, and Mahmoud Khademi. Learning to
Represent Programs with Graphs. In International Conference on Learning
Representations (ICLR), 2018. (https://arxiv.org/pdf/1711.00740.pdf)

#### Brockschmidt, 2019
Marc Brockschmidt. GNN-FiLM: Graph Neural Networks with Feature-wise Linear
Modulation. (https://arxiv.org/abs/1906.12192)

#### Li et al., 2015
Yujia Li, Daniel Tarlow, Marc Brockschmidt, and Richard Zemel. Gated Graph
Sequence Neural Networks. In International Conference on Learning
Representations (ICLR), 2016. (https://arxiv.org/pdf/1511.05493.pdf)

#### Ramakrishnan et al., 2014
Raghunathan Ramakrishnan, Pavlo O. Dral, Matthias Rupp, and O. Anatole
Von Lilienfeld. Quantum Chemistry Structures and Properties of 134 Kilo
Molecules. Scientific Data, 1, 2014.
(https://www.nature.com/articles/sdata201422/)

#### Schlichtkrull et al., 2017
Michael Schlichtkrull, Thomas N. Kipf, Peter Bloem, Rianne van den Berg,
Ivan Titov, and Max Welling. Modeling Relational Data with Graph
Convolutional Networks. In Extended Semantic Web Conference (ESWC), 2018.
(https://arxiv.org/pdf/1703.06103.pdf)

#### Sen et al., 2008
Prithviraj Sen, Galileo Namata, Mustafa Bilgic, Lise Getoor, Brian Galligher,
and Tina Eliassi-Rad. Collective Classification in Network Data. AI magazine,
29, 2008. (https://www.aaai.org/ojs/index.php/aimagazine/article/view/2157)

#### Veličković et al. 2018
Petar Veličković, Guillem Cucurull, Arantxa Casanova, Adriana Romero, Pietro
Liò, and Yoshua Bengio. Graph Attention Networks. In International Conference
on Learning Representations (ICLR), 2018. (https://arxiv.org/pdf/1710.10903.pdf)

#### Zitnik & Leskovec, 2017
Marinka Zitnik and Jure Leskovec. Predicting Multicellular Function Through
Multi-layer Tissue Networks. Bioinformatics, 33, 2017.
(https://arxiv.org/abs/1707.04638)

# Contributing

This project welcomes contributions and suggestions.  Most contributions
require you to agree to a Contributor License Agreement (CLA) declaring 
that you have the right to, and actually do, grant us the rights to use
your contribution. For details, visit https://cla.microsoft.com.

When you submit a pull request, a CLA-bot will automatically determine 
whether you need to provide a CLA and decorate the PR appropriately (e.g.,
label, comment). Simply follow the instructions provided by the bot.
You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
