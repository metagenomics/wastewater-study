# Metagenomics-tk setup for Global Wastewater Study

The following instructions explain how we setup the metagenomics-tk pipeline for the re-analysis of a global wastewater study.

## Per Sample Analysis

### Prerequisites

This setup has been tested on a SLURM system where all VMs are based on Ubuntu.
You will need to install java on the master node.

### 1. Checkout the repo and download all necessary assets

1. Checkout `git clone  --depth 1 --branch  <tag name> git@github.com:pbelmann/metagenomics-tk.git`

### 2. Download cplex binary

This part only works if you have access to a private bucket hosted on the Bielefeld de.NBI Cloud site.
If this is not the case, you can download the cplex binary from the IBM website.
Note that you will need to register with IBM to obtain an academic license for CPLEX.

1. Create a credentials file for s5cmd in your .aws folder: $HOME/.aws/credentials with the following content:

```
[default]
aws_access_key_id=
aws_secret_access_key=
```
2. Change directory to metagenomics-tk: `cd metagenomics-tk`

3. Place the cplex binary in cplex/docker: 
   `./bin/s5cmd  --endpoint-url https://openstack.cebitec.uni-bielefeld.de:8080 cp s3://wastewater-assets/cos_installer.bin  cplex/docker/`

4. Update builScript.sh script by setting the `DOCKERILE_FOLDER` variable. It should point the folder where the Dockerfile is placed.
   Example: `/vol/spool/final/metagenomics-tk/cplex/docker`


### 3. Configure metagenomics-tk

1. If you want to upload your results to Object storage, you will have to create AWS config file which looks like this:
   
```
aws {

  accessKey = ""
  secretKey = ""


    client {

      s_3_path_style_access = true
      connectionTimeout = 120000
      maxParallelTransfers = 28 
      maxErrorRetry = 10
      protocol = 'HTTPS'
      connectionTimeout = '2000'
      endpoint = 'https://openstack.cebitec.uni-bielefeld.de:8080'
      signerOverride = 'AWSS3V4SignerType'
    }
}
```

2. In case you want to to run more jobs in parallel, you can update the Nextflow queue size (`queueSize`) in the nextflow.config file. 


### 4. Download and prepare GTDB for fragment recruitment

1. Download GTDB

```
mkdir -p gtdb && wget https://openstack.cebitec.uni-bielefeld.de:8080/databases/gtdbtk_r214_data.tar.gz  -O - | tar -xzvf - -C gtdb
```  

2. Prepare paths file

```
echo "PATH" > paths.tsv && find gtdb -name "*.fna.gz" | xargs -I {} readlink -f {} >> paths.tsv
```

### 5. Run the main tool

1. Install nextflow
`cd metagenomics-tk && make nextflow`

2. In addition to the main Nextflow run command you can optionaly add the following parameter:

Specify `-with-weblog http://localhost:8000/run/<token-id>/` if you want to use a logging system like nf-tower or TraceFlow(https://github.com/vktrrdk/nextflowAnalysis).

Final command:

```
./nextflow -c prod_toolkit/aws.config run main.nf \
    -ansi-log false -profile slurm -resume -entry wFullPipeline -params-file default/fullPipeline_illumina_nanpore_without_aggregate.yml \
    --smetana_image=pbelmann/metabolomics:0.1.0  --carveme_image=pbelmann/metabolomics:0.1.0 \
    --steps.metabolomics.beforeProcessScript=/vol/spool/metagenomics-tk/cplex/buildScript.sh  --steps.metabolomics.carveme.additionalParams=''
```

### 6. Postprocess

Once the run is finished,
 * collect the stats produced by the trace file and commit them with the correct name pattern.
 * fill in the form in the google spread sheet.

## Aggregation

## 1. Run the following command for aggregating

