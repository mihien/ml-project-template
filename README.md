# Machine learning project template
This template was prepared to facilitate the routine of Docker image preparation for a typical deep learning project. Core idea of this template is usability. You need to do just a few steps and you are ready for running your experiments!

If you need to share the container you can share this folder evolved and Docker tarred image (`docker save my-image:latest > my-image.tar`), then your counterpart can easily run it with `bash docker_start.(sh | ps1)` and, voila!, they get the same enviroment as you!

This version of the template is based on [condaforge/miniforge3:4.12.0-0](https://hub.docker.com/layers/condaforge/miniforge3/4.12.0-0/images/sha256-784f8a0484efe4036ec185b469c656c6bb61d3bb5cc5dddedc06e95083db8f03?context=explore)


## Requirements:
Linux:
* [Docker with GPU support on Linux](https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html)
* Optionally: [Rootless Docker](https://docs.docker.com/engine/security/rootless/)

Windows:
* [Docker with GPU support on Windows 10/11](./docs/WINDOWS_DOCKER_GPU.md)

## Build image

1. Add proxy setting in the `~/.docker/config.json`:

        {
            "proxies": {
                "default": {
                    "httpProxy": "http://address.proxy.com:8888/",
                    "httpsProxy": "http://address.proxy.com:8888/"
                }
            }
        }
1. **Rename or add planned packages** `./src/package_name0` into a custom name. After the build you can import this module in python. You can add as many modules in `./src` as you want **before the build**. Do not forget, that each module should include `__init__.py` to be taken into account.
1. **Add pip install arguments** into `requirements.txt`. The file will be used with a command: `xargs -L 1 pip install --no-cache-dir < requirements.txt`. This means that each line will be executed as `pip install <line in requirements.txt>`
1. Add Pyton-installable libs into `./libs`. They will be installed during the build with `pip install -e <lib>` and can be imported in python directly.

1. **Build image**
* In Linux shell or WSL: `bash docker_build.sh`
* Follow prompts. Workspace dir is a directory on the host machine. Provide the full path, please.
    * To avoid interactive session you can provide the following arguments in the command: 
    
        ```bash
        bash docker_build.sh \
            --docker_image_name ml-project-template:tagged \
            --jupyter_password JUPYTER_PASSWORD \
            --ssh_password SSH_PASSWORD
        ```

## Start container
* In Linux shell or WSL: `bash docker_start_interactive.sh`
* **Follow prompts**. You will be asked to define IMAGE_NAME, CONTAINER_NAME, JUPYTER_PORT, TENSORBOARD_PORT, SSH_PORT. The ports you are asked to set-up are the **host** ports, advice available ports to your system admin if you work on remote server, or specify free ports if you work on local machine. 

    - Jupyter Lab is available at: `http://localhost:<JUPYTER_PORT>/lab  `
    - Jupyter Notebook is available at: `http://localhost:<JUPYTER_PORT>/tree`
    - Tensorboard is available at: `http://localhost:<TENSORBOARD_PORT>`, monitoring experiments in $tb.
    - Connect to container via SSH: `ssh -p <SSH_PORT> root@localhost` (if you are under proxy, no connection to outer world => no package installation possible)
    - Inspect the container: `docker exec -it <CONTAINER_NAME> bash` (if you are under proxy, install packages inside in this mode)
    - Stop the container: `docker stop <CONTAINER_HASH>`
    - Inside the container $ws will be available at /ws
* If you want to define additional docker run parameters, just provide them after the command.  
For example: `bash docker_start_interactive.sh -p 9898:9898`
### Custom start
* Refer to `docker_start_noninteractive.sh`

## Connect IDE to the running container:
* VSCode: [Docker with GPU support on Windows 10/11](./docs/VSCODE.md)
* PyCharm: TBD

## Update image
* You can access container by the comand: `docker exec -it <CONTAINER_NAME> bash`  
* Then install as many pip packages as you want (do not forget to add them into `requirements.txt`)  
* At the end you can update the image with the command: `docker commit --change=CMD ~/init.sh <CONTAINER_NAME> <IMAGE_NAME>`

## Share image
* Share the repo and then either build the image on new machine, or compress and decompress the image on a new machine :
* `docker save <IMAGE_NAME>:latest > my-image.tar`
* `docker load < my-image.tar`
  
## Notes:
- On Windows machine. If PowerShell says "execution of scripts is disabled on this system", you can  run in powershell with admin rights: `Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope LocalMachine` to allow scripts execution. But do it with caution, since some scripts can be vulnerable. For the details follow the [link](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.security/set-executionpolicy?view=powershell-7.2).
- You can attach VSCode to a running container: [quick tutorial](./docs/VSCODE.md), [documentation](https://code.visualstudio.com/docs/remote/containers)
- To commit updates from a running container to the built image use:  
    `docker commit --change='CMD ~/init.sh' updated_container_name_or_hash docker_image_name`  
    (_not recommended as a daily practice, good practice is to update the environment.yaml, requirements.txt or Dockerfile_)

## Project structure and philosophy behind

The idea behind this template is to be able to store lightweight code and heavy model artifacts and data in different places.

```
  # Code folder. Available under `/code` inside the container
  template-ml-project/
  ├── libs/
  |   ├── external_lib_as_submodule1/
  |   └── external_lib_as_submodule1/
  ├── src/
  │   ├── custom_module1/
  │   │   └── __init__.py
  |   └── custom_module2/
  |       └── __init__.py
  ├── notebooks
  │   └── jupyter_notebook_example.ipynb
  ├── .gitignore
  ├── Dockerfile
  ├── README.md
  ├── docker_build.ps1
  ├── docker_build.sh
  ├── docker_run.ps1
  ├── docker_run.sh
  ├── environment.yaml
  ├── requirements.txt
  ├── set_jupyter_password.py
  └── setup.py
  
  # Workspace folder. Available under `/ws` inside the container.
  template-ml-project-workspace/
  ├── data_raw
      └── file.dcm
  ├── data_processed
      └── file.npz
  ├── artifacts
      ├── segmentation_masks
      |   └── mask.jpg
      └── checkpoints
  ├── configs/
  └── etc/
 ```
