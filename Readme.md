# What do we want to be able to do?

* Run a basic nextflow script
* Run an Rscript (within Nextflow)
* Run the R script in parallel so that each one receives a different input argument
* Use NF-Tower with that R script 
* Run an R script (by calling a file)
* Run the process which calls the R script using a docker image
* Demonstrate that the R script really is running in the docker image by using an image without R
* Push our script to github and run it from our github repos
* Run an R function (using a JSON file to define inputs)
* Run an R function (passing a channel as inputs... which Nurun said is different to just passing arguments)
* Use renv with singularity to avoid having to create a singularity image containing all R dependencies
* Creating the above script within an nf-core template.... and explain why you would do this

# Intro pages about NextFlow
<details>
    <summary>
        Click to expand
    </summary><br>

    Start of doing the Quickstart: https://www.nextflow.io/
</details>

# Write a basic nextflow script
<details>
    <summary>
        Click to expand
    </summary><br>
    
Try running the first script from here: https://www.nextflow.io/docs/latest/getstarted.html#your-first-script

If you try running these on the HPC you will get an error. This is because by default it tries submitting the jobs to the HPC: tell it to use the local executor.

Create this file as tutorial.nf:

```
#!/usr/bin/env nextflow
params.str = 'Hello world!'
process splitLetters {
    executor ='local'
    output:
    file 'chunk_*' into letters
    """
    printf '${params.str}' | split -b 6 - chunk_
    """
}
process convertToUpper {
    executor = 'local'
    input:
    file x from letters.flatten()
    output:
    stdout result
    """
    cat $x | tr '[a-z]' '[A-Z]'
    """
}
result.view { it.trim() }
```

The first part of the script puts a variable called params.str into the global workspace

Nextflow then runs all 'process' functions in the order that they appear in the script

The first function, "splitLetters" does not have any inputs (but uses the global params variables)

Functions try running all code within the quote marks as the functions code... everything outside that explains inputs/outputs/environmments etc

The first function prints out the content of params.str and pipes this to the unix function "split"

The split function is explained here (http://www.theunixschool.com/2012/10/10-examples-of-split-command-in-unix.html)

It's splitting it's input into chunks of 6 bytes... then outputting them as seperate files. 
</details>

<details>
    <summary>
# Running the simple script so that it produces reports
    </summary>
```
/nextflow run ./tutorial.nf -with-report -with-timeline -with-dag flowchart.png

```

If you then connect to the server with SMB (smb://rds.imperial.ac.uk/rds/user/nskene/home) then you can click report.html to see how the run went
</details>

# Run an R script in parallel by submitting jobs with PBS
<details>
    <summary>
        Click to expand
    </summary><br>
Here's a typical template... inside the base directory where NF is run.. create a "bin" folder and put R scripts in there and run "chmod +x" on each of them... e.g. try to save this R script as save_dataset.R in the /bin/ folder: -

```
#!/usr/bin/env Rscript
args = commandArgs(trailingOnly=TRUE)
if (length(args) == 0) {
  stop("No dataset was specified.")
} else {
    dataset <- args[1]
}
sprintf("Loading dataset: %s", dataset)
do.call(data, list(x = eval(dataset)))
write.table(
    dataset, 
    file = paste0(dataset, ".tsv"), 
    sep = "\t", 
    col.names = TRUE, row.names = FALSE)
```

then run chmod +x save_dataset.R  on it

then the tutorial.nf becomes: -

```
#!/usr/bin/env nextflow
params.datasets = ['iris', 'mtcars']
process writeDataset {
    module 'R/3.4.0'
    executor = 'pbspro'
    clusterOptions = '-lselect=1:ncpus=1:mem=1Gb -l walltime=24:00:00 -V'
    tag "${dataset}"
    publishDir "$baseDir/data/", mode: 'copy', overwrite: false, pattern: "*.tsv"
    input:
    each dataset from params.datasets
    output:
    file '*.tsv' into datasets_ch
    """
    save_dataset.R ${dataset}
    """
}
```

The script runs the R script and also passes it a parameter from NF

For larger/more complex R scripts with multiple parameters, it's better to use the argparse package in R
</details>

# Run the process which calls the R script using a Docker / Singularity image
<details>
    <summary>
        Click to expand
    </summary><br>
The simplest way to do this is to tell it to use a docker hub image when you call the nextflow code, using an argument, i.e.

```
nextflow run test.nf -with-singularity "continuumio/miniconda"
```

Note that this pulls a docker image and converts it directly to a singularity image.


Alternatively,  create a file named ```nextflow.config``` in the current directory with:

```
process.container = 'continuumio/miniconda3' # the name of the image
singularity.enabled = true
singularity.cacheDir = 'work/singularity' #path to save the singularity images. can be changed to a shared folder
```

If you have access to the neurogenomics-lab shared workspace, then keep your images there so others can access them:

```
process.container = 'continuumio/miniconda3' # the name of the image
singularity.enabled = true
singularity.cacheDir = '~/projects/neurogenomics-lab' #path to save the singularity images. can be changed to a shared folder
```
</details>
