## Deep learning project template
A convenient starting template for most deep learning projects. Built with <b>PyTorch Lightning</b> and <b>Weights&Biases</b>.<br>
Click on <b>"Use this template"</b> button above to initialize new repository.<br>

## Features
- All advantages of PyTorch Lightning
- Predefined folder structure
- Storing project configuration in a convenient way ([project_config.yaml](project/project_config.yaml))
- Storing many run configurations in a convenient way ([run_configs.yaml](project/run_configs.yaml))
- Automates the whole training process and initialization, you only need to create `model` and `datamodule` and specify them in [run_configs.yaml](project/run_configs.yaml)
- Automatically stores all relevant code, configs and model checkpoints in Weights&Biases cloud
- Hyperparameter search with Weights&Biases sweeps ([execute_sweep.py](project/utils/execute_sweep.py))
- Scheduling execution of many experiments ([execute_all_runs.py](project/utils/execute_all_runs.py))
- Built in requirements ([requirements.txt](requirements.txt))
- Built in conda environment initialization ([conda_env.yaml](conda_env.yaml))
- Built in package setup ([setup.py](setup.py))
- Example with MNIST digits classification
- Useful example callbacks ([callbacks.py](project/utils/callbacks.py))
<br>


## Project structure
The directory structure of new project looks like this: 
```
├── project
│   ├── data                    <- Data from third party sources
│   │
│   ├── logs                    <- Logs generated by Weights&Biases and PyTorch Lightning
│   │
│   ├── notebooks               <- Jupyter notebooks
│   │
│   ├── utils                   <- Different utilities
│   │   ├── callbacks.py            <- Useful training callbacks
│   │   ├── execute_sweep.py        <- Special file for executing Weights&Biases sweeps
│   │   ├── execute_all_runs.py     <- Special file for executing all specified runs one after the other
│   │   ├── init_utils.py           <- Useful initializers
│   │   └── predict_example.py      <- Example of inference with trained model 
│   │
│   ├── data_modules            <- All your data modules should be located here!
│   │   ├── example_datamodule      <- Each datamodule should be located in separate folder!
│   │   │   ├── datamodule.py           <- Contains 'DataModule' class
│   │   │   ├── datasets.py             <- Contains pytorch 'Dataset' classes (optional file)
│   │   │   └── transforms.py           <- Contains data transformations (optional file)
│   │   ├── ...
│   │   └── ...
│   │
│   ├── models                  <- All your models should be located here!
│   │   ├── example_model           <- Each model should be located in separate folder!
│   │   │   ├── lightning_module.py     <- Contains 'LitModel' class with train/val/test step methods
│   │   │   └── models.py               <- Model architectures used by lightning_module.py (optional file) 
│   │   ├── ...
│   │   └── ...
│   │
│   ├── project_config.yaml     <- Project configuration
│   ├── run_configs.yaml        <- Configurations of different runs/experiments
│   └── train.py                <- Train model with chosen run configuration
│
├── .gitignore
├── LICENSE
├── README.md
├── conda_env.yaml
├── requirements.txt
└── setup.py
```
<br>


## Project config parameters ([project_config.yaml](project/project_config.yaml))
Example project configuration:
```yaml
num_of_gpus: -1             <- '-1' to use all gpus available, '0' to train on cpu

loggers:
    wandb:
        project: "project_name"     <- wandb project name
        entity: "some_name"         <- wandb entity name
        log_model: True             <- set True if you want to upload ckpts to wandb automatically
        offline: False              <- set True if you want to store all data locally

callbacks:
    checkpoint:
        monitor: "val_acc"      <- name of the logged metric that determines when model is improving
        save_top_k: 1           <- save k best models (determined by above metric)
        save_last: True         <- additionaly always save model from last epoch
        mode: "max"             <- can be "max" or "min"
    early_stop:
        monitor: "val_acc"      <- name of the logged metric that determines when model is improving
        patience: 5             <- how many epochs of not improving until training stops
        mode: "max"             <- can be "max" or "min"

printing:
    progress_bar_refresh_rate: 5    <- refresh rate of training bar in terminal
    weights_summary: "top"          <- print summary of model (alternatively "full")
    profiler: False                 <- set True if you want to see execution time profiling
```
<br>


## Run config parameters ([run_configs.yaml](project/run_configs.yaml))
You can store many run configurations in this file.<br>
Example run configuration:
```yaml
MNIST_CLASSIFIER_V1:
    trainer:    # these parameters will be passed directly to PyTorch Lightning 'Trainer' object
        max_epochs: 5
        gradient_clip_val: 0.5
        accumulate_grad_batches: 1
        limit_train_batches: 1.0
    model:      # these parameters will be passed to 'LightningModule' object as 'hparams' dictionary
        model_folder: "simple_mnist_classifier"
        lr: 0.001
        weight_decay: 0.000001
        input_size: 784
        output_size: 10
        lin1_size: 256
        lin2_size: 256
        lin3_size: 128
    dataset:    # these parameters will be passed to 'LightningDataModule' object as 'hparams' dictionary
        datamodule_folder: "mnist_digits_datamodule"
        batch_size: 256
        train_val_split_ratio: 0.9
        num_workers: 1
        pin_memory: False
    callbacks:
        ConfusionMatrixLoggerCallback:
            class_names: ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9"]
    wandb:
        group: ""
        tags: ["v1", "uwu"]
    resume_training:
        checkpoint_path: "path_to_checkpoint/last.ckpt"
        wandb_run_id: None
```
Each run configuration needs to contain sections `trainer`, `model` and `dataset`. Sections `callbacks`, `wandb` and `resume_training` are optional and can be removed.<br>

Section `model` always needs to contain `model_folder` parameter (name of the folder from which `lightning_module.py` will be loaded, which should contain `LitModel` class).<br>
Section `dataset` always needs to contain `datamodule_folder` parameter (name of the folder from which `datamodule.py` will be loaded, which should contain `DataModule` class).<br>

Every parameter in `model` section will be passed to your model class and can be retrieved through `hparams` dictionary (see example with [simple_mnist_classifier](project/models/simple_mnist_classifier/lightning_module.py)).<br>
Every parameter in `dataset` section will be passed to your datamodule class and can be retrieved through `hparams` dictionary (see example with [mnist_digits_datamodule](project/data_modules/mnist_digits_datamodule/datamodule.py)).<br>
<br>


## Workflow
1. Add your model to `project/models` folder. You need to create folder with `lightning_module.py` file containing `LitModel` class
2. Add your datamodule to `project/data_modules` folder. You need to create folder with `datamodule.py` file containing `DataModule` class
3. Create new run config in [run_configs.yaml](project/run_configs.yaml) (specify there folders containing your model and datamodule)
3. Configure [project_config.yaml](project/project_config.yaml)
4. Run training with chosen run config<br>
    ```bash
    python train.py --run_config MNIST_CLASSIFIER_V1
    ```
<br><br>


### DELETE EVERYTHING ABOVE FOR YOUR PROJECT  
 
---

<div align="center">    
 
# Your Project Name     

[![Paper](http://img.shields.io/badge/paper-arxiv.1001.2234-B31B1B.svg)](https://www.nature.com/articles/nature14539)
[![Conference](http://img.shields.io/badge/NeurIPS-2019-4b44ce.svg)](https://papers.nips.cc/book/advances-in-neural-information-processing-systems-31-2018)
[![Conference](http://img.shields.io/badge/ICLR-2019-4b44ce.svg)](https://papers.nips.cc/book/advances-in-neural-information-processing-systems-31-2018)
[![Conference](http://img.shields.io/badge/AnyConference-year-4b44ce.svg)](https://papers.nips.cc/book/advances-in-neural-information-processing-systems-31-2018)  

</div>

## Description   
What it does   

## How to run
First, install dependencies
```bash
# clone project
git clone https://github.com/YourGithubName/your-repo-name
cd your-repo-name

# optionally create conda environment
conda update conda
conda env create -f conda_env.yaml -n your_env_name
conda activate your_env_name

# install requirements
pip install -r requirements.txt
```

Next, you can train model without logging
```bash
# train model without Weights&Biases
# choose run config from project/run_configs.yaml
cd project
python train.py --no_wandb --run_config MNIST_CLASSIFIER_V1
```

Or you can train model with Weights&Biases logging
```yaml
# set project and entity names in project/project_config.yaml
loggers:
    wandb:
        project: "your_project_name"
        entity: "your_wandb_username_or_team"
```
```bash
# train model with Weights&Biases
# choose run config from project/run_configs.yaml
cd project
python train.py --run_config MNIST_CLASSIFIER_V1
```

Optionally you can install project as package with [setup.py](setup.py)
```bash
pip install -e .
```
<br>


#### PyCharm setup
- open this repository as PyCharm project
- set project interpreter:<br> 
`Ctrl + Shift + A -> type "Project Interpreter"`
- mark folder "project" as sources root:<br>
`right click on directory -> "Mark Directory as" -> "Sources Root"`
- set terminal emulation:<br> 
`Ctrl + Shift + A -> type "Edit Configurations..." -> select "Emulate terminal in output console"`
- run training:<br>
`right click on train.py file -> "Run 'train'"`

#### VS Code setup
- TODO
