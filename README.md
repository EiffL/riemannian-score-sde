# score-sde

## Install
This repo requires a modified version of [geomstats](https://github.com/geomstats/geomstats) that adds jax functionality, and a number of other modifications. This can be found [here](https://github.com/oxcsml/geomstats.git ) on the branch `jax_backend`.

Simple install instructions are:
```
git clone https://github.com/oxcsml/score-sde.git
cd score-sde
git clone --single-branch --branch jax_backend https://github.com/oxcsml/geomstats.git 
virtualenv -p python3.9 venv
source venv/bin/activate
pip install -r requirements.txt
pip install -r requirements_exps.txt
pip install -e geomstats
pip install -e .
```

- `requirements.txt` contains the core requirements for running the code in the `score_sde` and `riemmanian_score_sde` packages. NOTE: you ma need to alter the jax versions here to match your setup.
- `requirements_exps.txt` contains extra dependencies needed for running our experiments, and using the `run.py` file provided for training / testing models. 
- `requirements_slurm.txt` contains extra dependencies for using the job scheduling functionality of hydra.
- `requirements_dev.txt` contains some handy development packages.

## Run experiments
Experiment configuration is handled by [hydra](https://hydra.cc/docs/intro/), a highly flexible `ymal` based configuration package. Base configs can be found in `config`, and parameters are overridden in the command line. Sweeps over parameters can also be managed with a single command.

Jobs scheduled on a cluster using a number of different plugins. We use Slurm, and configs for this can be found in `config/server` (note these are reasonably general but have some setup-specific parts). Other systems can easily be substituted by creating a new server configuration.

The main training and testing script can be found in `run.py`, and is dispatched by running `python main.py [OPTIONs]`.

### Logging
By default we log to CSV files and to [Weights and biases](wandb.ai). To use weights and biases, you will need to have an appropriate `WANDB_API_KEY` set in your environment, and to modify the `entity` and `project` entries in the `config/logger/wandb.yaml` file. The top level local logging directory can be set via the `logs_dir` variable.

### S^2 toy
To run a toy experiment on the sphere run:
`python main.py experiment=s2_toy`
This should validate that the code is installed correctly and the the RSGM models are training properly.
### Earth datasets
We run experiments on 4 natural disaster experiments against a number of baselines. To run the full sweeps over parameters used in the paper run:

`RSGM ISM loss`:
```
python main.py -m \
    server=[SERVER] \
    experiment=volcano,earthquake,fire,flood \
    model=rsgm \
    generator=div_free,ambient \
    loss=ism \
    flow.N=20,50,200 \
    flow.beta_0=0.001 \
    flow.beta_f=2,3,5 \
    steps=300000,600000 \
    eps=1e-3 \
    seed=0,1,2,3,4 \
    n_jobs=8
```
`RSGM DSM loss`:
```
python main.py -m \
    server=[SERVER] \
    experiment=volcano,earthquake,fire,flood \
    model=rsgm \
    generator=div_free,ambient \
    loss=dsm0 \
    loss.thresh=0.0,0.2,0.3,0.5,0.8,1.0 \
    loss.n_max=-1,0,1,3,5,10,50 \
    flow.N=200 \
    flow.beta_0=0.001 \
    flow.beta_f=2,3,5 \
    eps=1e-3 \
    seed=0,1,2,3,4 \
    n_jobs=8
```
`Stereo RSGMs:`
```
python main.py -m \
    server=[SERVER] \
    experiment=volcanoe,earthquake,fire,flood \
    model=sgm_stereo \
    generator=ambient \
    loss=ism \
    flow.beta_0=0.001 \
    flow.beta_f=4,6,8 \
    eps=1e-3 \
    seed=0,1,2,3,4 \
    n_jobs=8
```
`Moser flows`:
```
python main.py -m \
    server=[SERVER] \
    experiment=volcanoe,earthquake,fire,flood \
    model=moser \
    loss.hutchinson_type=None \
    loss.K=20000 \
    loss.alpha_m=100 \
    seed=0,1,2,3,4 \
    n_jobs=8
```
`CNF`:
```
python main.py -m \
    server=[SERVER] \
    experiment=volcanoe,earthquake,fire,flood \
    model=cnf \
    generator=div_free,ambient \
    steps=100000 \
    flow.hutchinson_type=None \
    optim.learning_rate=1e-4 \
    seed=0,1,2,3,4 \
    n_jobs=8
```

### SO(3) toy
`python main.py experiment=so3 dataset=wrapped`

