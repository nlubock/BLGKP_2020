The code in this directory provides a docker image for running the bcl2fastq + starcode workflow used for the validation in Figure 3. It is license under the Apache v2.0. license (see LICENSE file).

To reproduce the data please take the following steps: First, pull the docker image from docker-hub

```
docker pull octant/swab-seq:reproduce
```

Alternatively, you can build the image locally

```
docker build --tag swab-seq:reproduce ./docker
```

Next, start the docker container

```
docker run -d --rm \
  -p 8787:8787 \
  -p 8888:8888 \
  -v /PATH/TO/BLCSBGLKP_2020:/home/rstudio/reproduce \
  -e USERID="$(id -u [YOUR_USER_NAME])" \
  -e GROUPID="$(id -g [YOUR_USER_NAME])" \
  -e UMASK=0002 \
  -e PASSWORD=[A_PASSWORD] \
  --name [CONTAINER_NAME] \
  swab-seq:reproduce \
```

The flags in the command above perform the following

```
-d -> run the container in the background
--rm -> remove the container when it exits
-p 8787:8787 -> expose port 8787 for Rstudio
-p 8888:8888 -> expose port 8888 for JupyterLab
-v -> mount [TARGET DIR]:[PATH DIR] (effectively exposes [TARGET DIR] to the container)
-e USERID=... -> ensures files created by docker image are owned by you
-e GROUPID=... -> ensures files created by docker image are owned by your group
-e UMASK=0002 -> keep the UMASK of the directory
-e PASSWORD=[A_PASSWORD] -> set a pasword for the Rstudio log in
--name [CONTAINER_NAME]. -> name your container something
swabseq:reproduce -> run the version of the swabseq container for reproducing these data
```

Now let's drop into our docker container and reproduce our data.

```
docker exec -it --user rstudio [CONTAINER_NAME] /bin/bash
```

Remember to substitute `[CONTAINER_NAME]` for whatever you assigned it in the previous step. This will expose a bash shell where you can navigate to the `reproduce` folder

```
cd reproduce/bcl2fastq_starcode
```

Now we can process our data with `make`

```
make all
```

When the data is finished processing, you can exit the shell with `ctrl-d`. Your container will remain running until you kill it with

```
docker kill [CONTAINER_NAME]
```

## Using Rstudio

If you would like to use R to analyze the resulting data, you can point your browswer to `localhost:8787`. You should see a login screen that you can access with

```
username: rstudio
password: [SOME_PASSWORD]
```

If you are running the docker image on a remote instance, you can access RStudio either by pointing your browser to `your.ip.address:8787` or by setting up an `ssh` tunnel from your local machine

```
ssh -N -L 8787:localhost:8787 [USER_NAME]@[your.ip.address]
```

and then pointing your browser to `localhost:8787`.

## Using JupyterLab

If you would like to use Python instead of R, we've also provided JupyterLab. To start the notebook server, we will once again drop into our container:

```
docker exec -it --user rstudio [CONTAINER_NAME] /bin/bash
jupyter lab
```

and follow the directions.

If you are planning on running JupyterLab from a server, add the `--no-browser` and ` --ip=0.0.0.0` arguments to the above command. If you are having trouble accessing the interface try the suggestions [in this thread](https://github.com/ipython/ipython/issues/6193). You will also likely need to set up an ssh tunnel as we did before for Rstudio (this time specifying port 8888):

```
ssh -N -L 8888:localhost:8888 [USER_NAME]@[your.ip.address]
```
