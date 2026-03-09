# Processing AHN using Laserfarm workflow on Spider

 This repository contains the following instructions:
 1. Run [Laserfarm](https://github.com/eEcoLiDAR/Laserfarm) on JupyterLab server and Dask cluster on a SLURM system. Please follow [this repository](https://github.com/RS-DAT/JupyterDaskOnSLURM.git) to set up the JupyterLab server on Spider. Examples have been run on the [Spider data processing platform](https://spiderdocs.readthedocs.io) hosted by [SURF](https://www.surf.nl).
 2. Download AHN4 data on Spider.
 3. Data transferring between local system and Spider storage.

## 1. Run Laserfarm on JupyterLab and Dask on SLURM

Log in Spider using Git Bash or Command Prompt by typing in

```shell
ssh lidarac-yshi@spider.surfsara.nl
```

Clone and access the current repository:

```shell
cd /project/lidarac/Software/Yifang/
git clone https://github.com/ShiYifang/AHN4_on_Spider.git .
cd AHN4_on_Spider.git
```

The workflow is run using the Python interpreter from the container image built using [Apptainer](https://apptainer.org/) via [this GitHub action](.github/workflows/build.yml) and based on the [`environment.yaml`](./environment.yml) conda environment file (and a few extra "required" dependencies defined [here](.github/apptainer/environment_default.yml)). The image is published on the [GitHub Container Registry](https://github.com/ShiYifang/AHN4_on_Spider/pkgs/container/ahn4_on_spider).

### Jupyter

Submit a batch job script based on the provided [template](./Scripts/jupyterdask-spider.slurm) to start the Jupyter server and the Dask scheduler on a compute node (one might want to change the node specifications depending on the requirements of the analysis running on the same node):

```shell
sbatch ./Scripts/jupyterdask-spider.slurm
```

Copy and paste the `ssh` command printed in the job stdout (file `JobOut/jupyter-spider.slurm-[JOBID].out`) in a new terminal window on your local machine (modify the path to the private key used to connect to the supercomputer). You can now access the Jupyter session running on the supercomputer from your browser at http://localhost:8889.

### Dask

A Dask cluster (with no worker) is started together with the JupyterLab session (it should be listed in the menu appearing when selecting the Dask tab on the left part of the screen). Workers can be added by clicking the "scale" button on the running cluster instance and by selecting the number of desired workers. Press "shutdown" to kill all workers and the scheduler. A new cluster based on the default configurations can be created by pressing the "+" button. A cluster with different specifications can be created in a Python notebook/console by instantiating a `SLURMCluster()` object with the desired custom features.

### Laserfarm

To run Laserfarm workflow on Spider, the `Client` needs to be set as the scheduler address, e.g. `client = Client("tcp://10.0.2.186:41037")` within each processing steps. Different from running on HPC cloud, the `cluster` is no longer needed here. For AHN4 data, the following steps are needed to produce the final LiDAR products:

* Retiling
* Normalization
* Feature extraction
* Geotiff exportation

Modification might be needed when to choose the classification value at `Feature extraction` step, setting `apply-filters` -> `value` based on the classification code for AHN4 (unclassified (1), ground (2), buildings (6), water (9), wire conductor (14), artificial objects (26), never classified (0)). Features can be defined at `features = []` and the resolution of final products can be defined at `tile_mesh_size`.

To check the statues of submitted job use
```shell
squeue --job[JOBID]
```
The progress of the processing can be checked via the provided dashboard URL (`/proxy/8787/status`). Files with `.out` provide the detailed information of the processing result.

After running all the steps, remember to shutdown all workers and scheduler.

## 2. Download AHN4 data on Spider

After login Spider, go to the folder where AHN4 data will be stored:

```shell
cd /project/lidarac/Data/AHN4
```

Check the `AHN4_downloader.sh` and `urls.txt` are correct and copied at the same directory. Then run:

```shell
chmod +x AHN4_downloader.sh
./AHN4_downloader.sh
```

To check the number of files have been downloaded in the directory:

```shell
find /project/lidarac/Data/AHN4 -maxdepth 1 -type f | wc -l
```

To check the file properties:

```shell
stat file_name
```

To check the usage of the storage:

```shell
getfattr -n ceph.dir.rbytes --absolute-names /project/lidarac/Data
```

## 3. Data transferring

To copy files from local system to spider local storage:

```shell
scp -i /c/Users/jshi/.ssh/id_rsa /D/UVA/LiDAR_Data/ahn4/C_44BZ2.laz lidarac-yshi@spider.surf.nl:/project/lidarac/
Software/Yifang/JupyterDaskOnSLURM/AHN4_test/rawlas/
```

To copy a result folder from spider to local system:

```shell
scp -i /c/Users/jshi/.ssh/id_rsa -r lidarac-yshi@spider.surf.nl:/project/lidarac/
Software/Yifang/JupyterDaskOnSLURM/AHN4_test/Geotiff_veg /D/UVA/LiDAR_Data/ahn4/
```

### Resources

* [RS-DAT: Jupyter and Dask on HPC](https://github.com/RS-DAT/JupyterDaskOnSLURM)
