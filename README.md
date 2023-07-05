# Multiscale Run Docker Image

This repository provides the Docker recipe for the multiscale run project, i.e. the project that couples [Neurodamus](https://github.com/BlueBrain/neurodamus), [STEPS](https://github.com/CNS-OIST/STEPS), [AstroVascPy](https://github.com/BlueBrain/AstroVascPy) & [metabolism](https://www.epfl.ch/research/domains/bluebrain/blue-brain/people/our-people/simulation-neuroscience-division/polina-shichkova/).

## Requirements

You must have:

* [Docker](https://www.docker.com/community-edition) installed and running.
* [Docker Compose](https://docs.docker.com/compose) utility installed.
* [Git](https://git-scm.com/)

## Fetch Multiscale Run Project

These steps download a reduced/experimental version of multiscale run project:

* If on MacOS/Linux (**pip required**), `./setup_multiscale_run.sh`, and skip the steps below.
* Download [multiscale run](https://drive.google.com/file/d/1ZgdF4R2UgL_s8TK4lnb8qnSmhxex81gJ/view?usp=sharing) (click on the hyperlink).
* Uncompress the downloaded project under the **notebooks** folder.
* Follow the steps below to build & deploy the docker image.

## Getting Started (MacOS & Linux)

The set of commands below will start a JupyterLab container providing
python modules for all the coupled solvers.

```bash
$ git clone https://github.com/kotsaloscv/multiscale_run_dimage.git
$ cd multiscale_run_dimage
$ echo "DUID=$(($(id -u)+1))\nDGID=$(id -g)\nHOST=$(hostname)" > .env
$ docker-compose build
$ docker-compose up
...
multiscale_run_dimage-lab-1  | [C 2023-07-05 13:59:36.052 ServerApp] 
multiscale_run_dimage-lab-1  |     
multiscale_run_dimage-lab-1  |     To access the server, open this file in a browser:
multiscale_run_dimage-lab-1  |         file:///home/dummy/.local/share/jupyter/runtime/jpserver-29-open.html
multiscale_run_dimage-lab-1  |     Or copy and paste one of these URLs:
multiscale_run_dimage-lab-1  |         http://bb-c02dk0usml85.epfl.ch:8888/lab?token=596536b192733c7041b845a969e980e6814845132625e99e
multiscale_run_dimage-lab-1  |         http://127.0.0.1:8888/lab?token=596536b192733c7041b845a969e980e6814845132625e99e
```

Then open your web browser at the provided HTTP address. In this case
http://127.0.0.1:8888/lab?token=596536b192733c7041b845a969e980e6814845132625e99e

## Files management

Inside the JupyterLab files browser, you will be able to see `STEPS_Example` directory
providing code samples to start with.

You are free to modify the `notebooks` directory from either the container or
your machine. Files created on one side will be visible on the other one, and vice versa!

## Advanced Usage

### Run big simulations in OSX

On OSX, you may have to increase the memory allocated to the Docker containers
to execute important simulations. Default reserved memory in 2GB.
See official documentation [here](https://docs.docker.com/docker-for-mac/#memory)
to increase it.

### How to use traditional Jupyter Notebook

Jupyter Notebook is very lazy when it comes to the syntax of ipynb files compared
to JupyterLab. In JupyterLab, notebooks must be valid JSON files. This may prevent
you to import your notebooks. In this case, you can either:

* Fix JSON issues in your existing notebooks. To detect syntax errors, you can
  use the command below:

    ```bash
    <YOUR_NOTEBOOK.ipynb python -m json.tool
    ```

* Use the `notebook` container provided in the `docker-compose.yml` file:

    ```bash
    docker-compose up notebook
    ```

## Windows support

This Docker image can be run with _Docker Desktop for Windows_. Instructions in the *Getting Started* section above are a bit different though. Instead of executing command `echo "DUID=$(($(id -u)+1))\nDGID=$(id -g)\nHOST=$(hostname)" > .env`, update the `docker-compose.yaml` file as follow:

* **hostname**: hardcode the machine name
* **USER_LOGIN**: hardcode your user name
* **USER_ID**: 42
  * If you get an error similar to `useradd: UID 42 is not unique`, you can change this value to an other arbitrary one, e.g. 43.
* **GROUP_ID**: 42
* **volumes**: replace `$PWD` by the path to this repository.

for instance:

```diff
diff --git a/docker-compose.yml b/docker-compose.yml
index 528e993..64dcca9 100644
--- a/docker-compose.yml
+++ b/docker-compose.yml
@@ -3,15 +3,15 @@ services:
   lab:
     image: ...
     build: recipe
-    hostname: $HOST
+    hostname: my-windows10-machine
     ports:
     - "8888:8888"
     environment:
-    - USER_LOGIN=$USER
-    - USER_ID=$DUID
-    - GROUP_ID=$DGID
+    - USER_LOGIN=YOUR_USER_NAME
+    - USER_ID=42
+    - GROUP_ID=42
     volumes:
-    - $PWD/notebooks:/opt/src/notebooks
+    - C:\Users\YOUR_USER_NAME\Documents\multiscale_run_dimage\notebooks:/opt/src/notebooks
   notebook:
     extends: lab
     command:
```

Before running `docker-compose build`, it is imperative to open `./recipe/entrypoint` with Visual Code (or your favourite text editor), and change the **End of Line Sequence** from `CRLF` to `LF`.

From:
![image](images/crlf.png)

To:
![image](images/lf.png)

## Acknowledgement

The development of this software was supported by funding to the Blue Brain Project, a research center of the
École polytechnique fédérale de Lausanne (EPFL), from the Swiss government’s ETH Board of the Swiss Federal
Institutes of Technology.

Copyright (c) 2005-2023 Blue Brain Project/EPFL

## License

Apache-2.0 license (see LICENSE.txt)
