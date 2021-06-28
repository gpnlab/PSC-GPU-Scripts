# PSC-GPU-Scripts

# PSC GPU Usage

This document serves as a reference for the basic usage of the PSC GPU allocation. If there is something that is missed please refer to the bridges 2 user guide here: https://www.psc.edu/resources/bridges-2/user-guide-2/.

## Accessing the PSC

Before accessing the supercomputer please contact L.wang@pitt.edu to be added to the GPU allocation. Please do this even if you are already on the CPU allocation since that is different.

### SSH

To access the supercomputer first register an account at https://apr.psc.edu/. This will let you login directly through the PSC node rather than through XSEDE. Once you have done this you may need to wait a while until the server has updated your password. Once that has happened login using the following command 

```
ssh username@bridges2.psc.edu
```

You will then be prompted to enter your password that you set using the link above (note: if you have different XSEDE and PSC passwords, use your PSC password). 

### Navigation

Once you login you will first be placed at the login node of the supercomputer. This is where you can store small scripts and miscellaneous items, but you only have 10 GB of space here. You can use the `ls` command to see all the files and folders in your current directory. The `cd` command will let you navigate to other folders on the server. You can use `cd ..` to move up in the file directory or `cd folder/` to move to a folder that you can see using the ls command. You can also move to folders you don't directly see with the [absolute file path](https://www.linux.com/training-tutorials/absolute-path-vs-relative-path-linuxunix/). In our case, you can use the command 

```
cd /ocean/projects/med200001p/
```

 to move to our project folder. If you are ever get lost you can use the `pwd` command to see what your current location is. 

### Project selection

For those on multiple projects you will need to make sure you are using the right allocation or else you won't be able to use the GPU's/CPU's. To do this use the `change_primary_group -l` command to see all available projects. Then use `change_primary_group account-id` to set the correct project.

## Running jobs and scripts

### SLURM 

Slurm is the main job manager on the supercomputer. We won't go too in depth in this document, but we will eventually want to use this to queue up jobs for training multiple models or for more computationally expensive jobs. The important commands to know for this `sbatch` which lets you queue up jobs, `squeue -u username` which lets you see jobs running under a specific username, and `scancel jobID` which lets you cancel jobs. You can use  `squeue -u username` to look up the jobID that you want to cancel. 

### Interactive mode

We will mainly start with using the Interactive mode to get familiar with the resource. To use this use the command

```
interact -p GPU-shared--gres=gpu:v100-16:1
```

to start an interactive instance on the GPU-shared partition requesting 1 GPU from the v100-16 GPU node. When in doubt use the shared partitions as they are charged at a much lower rate that the standard GPU and CPU partitions (please refer to the user guide for more details). 

## Python and Pytorch

All documentation for the following section is from: https://www.psc.edu/resources/software/jupyter/.

### Conda

Since Pytorch is not pre-built on PSC we will need to set it up ourselves. We will do this by calling the `module load anaconda3`(there are a couple other steps here that I don't entirely remember to setup conda). From here we want to also load CUDA so that our models can run using the GPU with `module load cuda/11.1`. First we will want to create a new virtual environment to install everything by calling `conda create -n "name" -y` this will create a new isolated virtual environment to install everything in named "name". To use the environment call `conda activate name` where "name" is the name of your environment. The default environment is (base) which you will see on the left of you command line, this will change to (name) when you activate your virtual environment. Next run `conda install pytorch torchvision torchaudio cudatoolkit=11.1 -c pytorch -c nvidia` to install Pytorch for GPU. This uses CUDA version 11.1 which is why we also specified the CUDA version above. We will also want to run `conda install jupyter-lab` to use Jupyter-labs to test work with our code in an interactive session. 

### Jupyter Notebooks

Jupyter Notebooks (and lab) are really powerful interactive python tools that let you rapidly develop, test, and present code. To set this up first ensure that you have selected the correct virtual environment then call 

```
jupyter lab --no-browser --ip=0.0.0.0
```

 . You should see a line that looks like 

```
[I 2021-06-28 14:35:04.283 ServerApp] http://v033.ib.bridges2.psc.edu:8888/lab
```

 possibly with a more random text at the end. The `v033` will be different depending on the node you are assigned so adjust this part as needed based on what is displayed.  We will then need to open another terminal window since to connect to Bridges 2 again. This time you will want to call 

```
ssh -L 8889:v033.ib.bridges2.psc.edu:8888 username@bridges2.psc.edu
```

this will connect port 8889 on your computer to port 8888 on the GPU node allowing you to connect to the Jupyter lab instance. Note that the `v033` in this line will change depending on the node. Then just open any browser and go to `localhost:8889` which should connect you to the Jupyter instance. You may be asked to enter a token which will be the random letters and numbers following the `http://v033.ib.bridges2.psc.edu:8888/lab/?token=` you will also be able to assign a password which I recommend for convenience.

### Code considerations

To test you code use the following

```python
import torch
x = torch.rand(5, 3)
print(x)
print(torch.cuda.is_available())
```

you should see some tensor and True as the outputs.

When using a GPU you will need to move your data and your model to your CUDA device usually with the `.to("cuda")` function. Please refer to https://pytorch.org/docs/stable/notes/cuda.html for more specifics. 

Also when using multiple GPUs you will need to design your models and code to split the work accordingly between the GPUs which can be a challenge. I would avoid this for now and only request 1 GPU at a time to avoid wasting computing resources.

### Other tips

If you are using VS Code (which I recommend if you are unfamiliar with vim/emacs) you can connect your workspace to bridges following these instructions: https://code.visualstudio.com/docs/remote/ssh. You can also connect your Jupyter instance to the PSC by setting the address to `localhost:8889`. 

 you 