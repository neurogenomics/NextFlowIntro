# What do we want to be able to do?

* Run a basic nextflow script
* Run an R script
* Run the R script in parallel so that each one receives a different input argument
* Run the process which calls the R script using a docker image
* Push our script to github and run it from our github repos

# Intro pages about NextFlow

Start of doing the Quickstart: https://www.nextflow.io/

# Write a basic nextflow script

Try running the first script from here: https://www.nextflow.io/docs/latest/getstarted.html#your-first-script

If you try running these on the HPC you will get an error. This is because by default it tries submitting the jobs to the HPC.

So tell it to use the local executor:

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

# Running the simple script so that it produces reports

```
/nextflow run ./tutorial.nf -with-report -with-timeline -with-dag flowchart.png

```

If you then connect to the server with SMB (smb://rds.imperial.ac.uk/rds/user/nskene/home) then you can click report.html to see how the run went
