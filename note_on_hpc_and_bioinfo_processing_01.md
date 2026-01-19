<!-- title: Bash -->
**Notes on HPC and bioinformatics**

These are a series of notes that I made after completing my thesis project. I wish to record the packages and some of the scripts that I frequently used throughout the project. Please note that the information may not be 100% correct so please use the code with caution.

Also for myself: [which types of languages get highlighted by md](https://github.com/highlightjs/highlight.js/blob/main/SUPPORTED_LANGUAGES.md)?

# Section 1: Bash 

**Table of Contents**
- [Section 1: Bash](#section-1-bash)
  - [1.1 Using bash / linux in Windows](#11-using-bash--linux-in-windows)
  - [1.2 Basic bash commands](#12-basic-bash-commands)
    - [1. *Navigating around the system | `cd`, `pwd`, `ls`, `ll`, `du`*](#1-navigating-around-the-system--cd-pwd-ls-ll-du)
    - [2. *Printing something out | `echo`, `cat`*](#2-printing-something-out--echo-cat)
    - [3. *Creating and reading a file | `touch`, `>`, `nano`, `cp`, `mv`, `head`, `tail`, `cat`, `less`, `read`, `readarray`*](#3-creating-and-reading-a-file--touch--nano-cp-mv-head-tail-cat-less-read-readarray)
  - [1.3 (Slightly) Advanced bash commands](#13-slightly-advanced-bash-commands)
    - [1. *Changing file contents, searching, etc. | `wc`, `cut`, `sed`, `sort`, `grep`*](#1-changing-file-contents-searching-etc--wc-cut-sed-sort-grep)
    - [2. *Performing complex  row-wise behaviours | `awk`*](#2-performing-complex--row-wise-behaviours--awk)
    - [3. *Transferring files between local, cluster or cloud | `rsync`, `rclone`*](#3-transferring-files-between-local-cluster-or-cloud--rsync-rclone)
    - [4. *Creating and removing temporary files | `mktemp`, `trap`*](#4-creating-and-removing-temporary-files--mktemp-trap)
  - [1.4 Other useful features in Bash](#14-other-useful-features-in-bash)
    - [1. Parameter expansion](#1-parameter-expansion)
    - [2. Process substitutions `<()` and command substitutions `$()`](#2-process-substitutions--and-command-substitutions-)
  - [1.5 Brief notes on HPC](#15-brief-notes-on-hpc)
  - [1.6 Using conda](#16-using-conda)

## 1.1 Using bash / linux in Windows

Unlike Mac users, Windows users need to manually install Linux system.

**1. Installing ubuntu**\
For a windows system, first install the `WSL` (windows subsystem for linux) or `ubuntu` following this [link for WSL](https://learn.microsoft.com/en-us/windows/wsl/install) and this [link for ubuntu](https://ubuntu.com/desktop/wsl). Then you can access the operating system by typing `ubuntu` in the search bar.

If the installation is successful, you can access in two ways:
1. Go to command prompt, and type: `wsl`
2. Search "ubuntu" and open the ubuntu terminal

**2. I want to access files in an external hard drive**\
Sometimes you want to access data in an external hard drive, which cannot be recognised unless being "mounted". This is an example script I use to mount a drive `D:` and avoid the permission issues: 
```bash
sudo mount -t drvfs D: /mnt/d -o uid=1000,gid=1000
```
 (I cannot remember where I copied this - apologies). You can check if the drive is mounted or not by going to the `/mnt` directory using `cd` command. If successful the drive should be accessible and you can view all files using `ls`.

To remove the drive after you finish, use: 
```bash
sudo umount /mnt/d
```

**3. I forget my linux password**\
Using `sudo` commands requires a password. If you have forgotten the password, it is possible to reset it following these instructions, based on the discussion [here](https://askubuntu.com/questions/931940/unable-to-change-the-root-password-in-windows-10-wsl).

1. Open the windows terminal, `cmd.exe`
2. In the terminal, type the following, line by line: 

```cmd
wsl -u root
passwd username 
:: Now you shall be prompted to change your password
exit
``` 

Username is the string coming before `@computer` when you use `ubuntu`. The password should be changed after these steps.

## 1.2 Basic bash commands

There are a lot of useful commands in Bash. I am only familiar with very few of them. I have categorised them according to my subjective rating for their "usefulness". If you want to receive help on a command, in bash you can type its name and: `--h` or `-h`.

These are the most basic commands for viewing, navigating, and creating files. Most of them are simple to use, and all of them are frequently used in data processing.

---

### 1. *Navigating around the system | `cd`, `pwd`, `ls`, `ll`, `du`*

1. `cd directory` and `pwd`: `cd` will let you go to a specified location, whereas `pwd` prints the current directory.

2. `ls location` and `ll location`: shows the names and information of files listed in a named directory. They can also be combined with *globbing characters* to specifically look for files with certain pattern. 

```bash
ls myprotein_[abc].pdf
```
> More on **globbing characters / glob**: These are useful in matching **filenames** in Linux, i.e. they helps to match filenames containing certain patterns. Many useful links explain them, like this [one](https://www.scaler.com/topics/glob-linux/). 
> 
> Some commonly used are:
> - `*` for 0+ characters
> - `?` for exactly one characters
> - `[123abc]` match any one character from bracket
> - `[1-3]` match anything in the range
> - `^!` excludes characters
>
> Note that glob can be used with many commands that requires a filename. When used, they cannot be quoted. Finally, glob is different from **regular expression**.

3. `du location`: Less about navigating, this commands prints the size of files in specified directory. Use `du -h` so results are human-readable.

---

### 2. *Printing something out | `echo`, `cat`*

1. `echo something`: print something, useful when providing a message like current file being processed.

2. `cat filename`: it requires *files* as input and displays the contents. For example, you can use `cat file1 file2` to combine multiple files, or use `cat -A file` to show the non-printing characters.\
`-A` flag is particularly useful to check: **1)** if a tab separated file (tsv) is correctly separated, with `^I` between columns; **2)** if there are any *carriage return* `\r` at the end of each row (which is a feature of file created in Windows). Do this checking when `sed` or `cut` do something weird.

---

### 3. *Creating and reading a file | `touch`, `>`, `nano`, `cp`, `mv`, `head`, `tail`, `cat`, `less`, `read`, `readarray`*

There are many different ways to create a new file.

1. `touch filename`: This will create an empty file with the specified name. If the file exists, the modification time will change but content will not be touched.

2. `> filename`: **Output redirection** -- it will create an empty file, or empty an existing file under the name without deleting it. Use it with caution when there are already files under the name!

3. `nano filename`: If file with the specified name has not been created, this can be used generate a new file. The file has to be **manually saved** using `ctrl + O`, then use `ctrl + X` to exit. It can be used to edit an existing file. I also find `ctrl + K` useful: this allows you to quickly cut rows to remove them or paste them elsewhere. `alt - U` to undo.

4. `... >> filename`: **Append redirection** -- it will append new information to an existing file and is safer to use when you are working on important data. **Do not confuse it with output redirection, which wipes out your content!**

5. `cp filename newfilename`, `mv filename newfilename`: These commands will create a file named `newfilename` with same information as the original file, `filename`. As name suggests, `cp` will retain new file, whereas `mv` will rename the old file to its new name.

After creating new files, you now can read a file! Depending on **your goals**, you may want to use different ways to read it.

1. To have a glimpse of the file: Use `head filename` or `tail filename`: by default, they will only show 10 rows at the start or end (change using `-n num`). This is useful to know what information is kept in the file, or checking if a work has successfully completed by inspecting the end of a log.
   
2. Checking structure of the file: Use `cat -A` with `head` / `tail` to find out if columns are properly separated or there is no trailing `\r` is important for processing data. If your file is zipped, then use `zcat`.
```bash
cat -A something | head
```
See previous section on `cat`.
   
3. View / scroll through the file, especially when the file is large: Use `less filename`. Once inside, you can search for strings using `/pattern`. I am not very familiar so please read [here](https://unix.stackexchange.com/questions/31/list-of-useful-less-functions) for more useful functions. `less` is extremely powerful! If your file is zipped, then use `zless`.
   
4. Read and edit the file: Use `nano`. This is the text editor we have discussed a while ago.
   
5. Do something based on file rows: Use `while` loop + `read` command (read discussion [here](https://stackoverflow.com/questions/62668968/how-read-line-in-while-loop-works)), which stores each row as a **variable**. 

```bash
while IFS= read -r LINE; do 
    # Do something for each row ($LINE)
    value=$(echo $LINE | cut -f1)
done < infile 
```

Alternatively, use `readarray -t`, for example:
```bash
readarray -t myarray < infile
LINE1=${myarray[0]} # each element is a row
```
These are useful when the `infile` contains some metadata, for example, and you want to do something according to different values appearing on each row. 
`readarray` is more efficient than `while + read` loop for small files. Sometimes I will create single-column files storing sample ID and read them into a script using `readarray`. I find it cleaner compared to typing an array inside the script and makes the file more flexible.

```bash
# Do this in command line
suffix="_original.bed"
ls *${suffix} | sed "s|${suffix}$||" > examplearr.txt

# Then write these in PBS job
suffix_in="_original.bed"
suffix_out="_modified.bed"
filelist="examplearr.txt"

readarray -t myarray < $filelist
fileproc=${myarray[$PBS_ARRAY_INDEX]}

infile="${fileproc}${suffix_in}"
outfile="${fileproc}${suffix_out}"
echo "Currently processing: ${fileproc}"
```

---

## 1.3 (Slightly) Advanced bash commands

### 1. *Changing file contents, searching, etc. | `wc`, `cut`, `sed`, `sort`, `grep`*

These commands will do something to files. Here is only a basic introduction so I will attach links from w3schools which is far more informative than mine, but I will try to include how commands might be useful.

1. `wc filename`: This returns the length of the file, in terms of the character, word, and row count. As this command has a habbit of printing filename with its output, you can mute this behaviour as follows:

```bash
### Muting filename from wc
# file as stdin
wc -l < filename

# piping input from other commands.
grep "pattern" filename | wc -l
```

**Example:** I sometimes use this command to roughly understand the scale of your outputs, such as using `wc -l < filename` on `bed` files to study the number of peaks from Chip-seq experiment.

2. `cut -f1 -d, filename`: This allows you to select some columns. First by some delimiter using `-d`, and retain some columns based on their location (counting from 1) using `-f`. Such as: `cut -d, -f2-4 something.csv` for a csv file. You do not need to specify delimiter if tab-separated (see if there is `^I` between columns). `cut` is simple and quick to use but it is also functionally simple. If `cut` is not working or behaving weirdly, use `awk` instead (see below); additionally, if you wish to reorder columns, use `awk`.\
**Example:** This is useful if you want to find data from a specified column (i.e. a specific variable), for example if you only want the gene symbol from a gtf file. 

3. `sed "s|pattern1|replacement|behaviour" filename`: This is used to replace something with particular pattern, either in place (with flag `-i`) or redirect to a new file (`sed "s|p1|r1|g" filename > newfile`; `g` stands for global and ensures substitution is made for all matched patterns). See more [here](https://www.w3schools.com/bash/bash_sed.php). You can alternatively separate the delimiter using `/` or `:` just ensure it is consistent.

4. `sort`: It is often used with the `-k` flag to sort a file based on specific columns, or use `-u` to remove duplicated rows. See more [here](https://www.w3schools.com/bash/bash_sort.php).\
**Example:** This command is frequently used with `bed` or `bedpe` (genomic coordinates) file, where rows need to be ordered based on their positions in the genome. It functions similarly as [`bedtools sort`](https://bedtools.readthedocs.io/en/latest/content/tools/sort.html). The most often usage is: 

```bash
# Bed: sort based on chr and start coordinate
sort -k 1,1 2,2n file1.bed > file1_sortByCoord.bed
# Bedpe: sort based on chrs and start coordinates from left, then right feature
sort -k 1,1 2,2n 4,4 5,5n file2.bedpe > file2_sortByCoord.bedpe
```

5. `grep "pattern" filename`: This is used for searching a specific pattern in a file, and is poswerful when using in combination with regular expressions. See more [here](https://www.w3schools.com/bash/bash_grep.php).\
**Example:** For instance, when you want to subset a dataset based on presence of a specific pattern, or selecting rows matching an ID. You can also remove specific rows using `-v` flag, like `grep -v "^#"` to remove annotation / header rows in a gtf file.

---

### 2. *Performing complex  row-wise behaviours | `awk`*

`awk` should belong to the previous section. However, as this tool is **exteremely useful** during my processing I want to write a bit more. Note that `awk` has a slightly different grammar than `bash` and its own functions.

Some of `awk`'s functionalities overlaps with the commands we have discussed before. For example, you can select columns using their index in awk: `awk -F, '{print $2, $3}'` which equals to `cut -d, -f2,3`. (By the way, `$0` is the full row). However, `awk` is more flexible: you can simply change order of columns using `'{print $3, $2}'` inside the quotes, which is not achieveable in `cut`!

In brief, many command line tools are highly specified and efficient. `awk` is more flexible but is slightly slower than these more devoted tools.You can use `awk` to achieve the following, but there are many additional applications:

1. **Changing the file separator**: A brief explanation: after `-F` you specify the file separator of the input file (the same as `'BEGIN{FS = ","}'`). `OFS` is a variable name reserved for the **o**utput **f**ile **s**eparator.
   
```bash
awk -F, -v OFS="\t" "{print $1,$2,$3}" ex.csv > ex.tsv 
```
   
2. **Filtering rows based on a specific value and column**: For example the following script returns all rows (`{print $0}` is default behaviour) whose first column equals to the specified variable, `mychrom` (do not need `$`, unlike in bash). Conditionals outside the `{}` are used to filter rows, and operations inside `{}` will be performed on all rows meeting this filter.

```bash
awk -v mychrom="chr1" '$1 == mychrom' example.bed > example_chr1.bed
```
   
3. **Creating IDs by combining existing values**, printing formatted strings: `awk` supports the use of `printf` and concatenate its output, which can be used to create ID columns.
```bash
awk '{print $1, $2, $3, $4, $5, $6, "loop_" NR}' exampleLoop.bedpe > exammpleLoop_wID.bedpe
``` 
   
4. **Math and creating summary statistics**: This involves using `BEGIN` and `END` blocks, which are only processed [once](https://www.gnu.org/software/gawk/manual/html_node/Using-BEGIN_002fEND.html). For example, the following will calculate the mean using non-zero values in column 2 and store it to a variable called `mean_no_zero`.

```bash
mean_no_zero=$(awk 'BEGIN {nrow = 0; sum = 0} {if ($2 != 0) {nrow ++; sum += $2}} END {if (nrow > 0) print sum / nrow; else print "0"}')
```
   
5. **Do something for one file relative to another**: For example, you can use values from the first file to filter the second file. In the following example, column 1 of `file1` is used as **key** to filter `file2`. `NR` records number of all processed record whereas `FNR` is current record number, so the conditional `NR == FNR` will only be met for the first file (see [here](https://stackoverflow.com/questions/32481877/what-are-nr-and-fnr-and-what-does-nr-fnr-imply) for a discussion!).

```bash
# Imagine we have two lists of differentially expressed genes
# We want to keep DEGs in cell2 that are differentially expressed in cell1

awk -F, -v OFS="," 'NR == FNR {a[$1]; next} $1 in a {print $0}' cell1_DEG.csv cell2_DEG.csv > cell2_DEG_in_cell1.csv
```
    
6. **Take advantage of `awk` functions to achieve more complicated behaviour**. For example, combining `split()` and `match()` functions in `awk` to obtain data from complex tables in a clear and readable manner. Let's say we want to separate the column 9 in a [Gencode GTF annotation file](https://www.gencodegenes.org/human/). This can be achieved using the following:

```bash
# If I want a cleaned bed file of TSS from gencode "genes"
awk -v OFS="\t" '
BEGIN { FS = "\t" }
{
    # Check feature is gene
    if ($3 == "gene") {
        chr = $1; raw_start = $4; raw_end = $5; strand = $7;  description = $9; # You can assign column to variables

	# Parse description for ID / type / Symbol
	split(description, description_split, "; ")
	match(description_split[1], /"([^"]*)"/, gene_id)
	match(description_split[2], /"([^"]*)"/, gene_type)
	match(description_split[3], /"([^"]*)"/, gene_name)

	# Depending on strand, get start and end sites
	if (strand == "+") {
	    start = raw_start - 1
	    end = raw_start
	} else if (strand == "-") {
	    start = raw_end - 1
	    end = raw_end
	}

	# Report all information
	print chr, start, end, gene_id[1], gene_name[1], gene_type[1]
    }
}
' $gencode_gtf_file > $output_file
```

You can find a list of the functions [here](https://www.gnu.org/software/gawk/manual/html_node/Built_002din.html).

There are definitely other exciting applications of `awk` that I missed at present, and I am happy to learn them from you. Overall it is a very flexible and quick command line tool that I really recommend using, whenever you found the intended behaviour is more complicated that basic bash commands but does not require Python / R logic.

---

### 3. *Transferring files between local, cluster or cloud | `rsync`, `rclone`*

These two functions allows you to move files around and access remote data, such as those stored in a Google drive or onedrive.

1. `rsync`: This command is useful in copying large file and prevents data loss. It can be used to transfer data within linux or between **cluster** (like HPC):

```bash
rsync -avP username@address.of.hpc:some/remote/location/filename* local/location
```

This command recursively retrieve all files at `some/remote/location` and store them at `local/location`, while tracking progress and providing a progress bar. If you want to access some files ending only in some suffix like `.bam`, or ignore some files like `.html`, use `--include` and `exclude` flags. Multiple flags can be used, but they need to be placed in a certain order because `rsync` execute the early appearing rules first and exits if conditions meet. This is a bit like the logic `CASE` in SQL or other languages, where you usually want early-appearing flags to match specific conditions and the last flag catching all remaining cases.

```bash
# If I want to download all but html from the source
# Logic: if a file ends in *.html, exclude. For all remaining, include.
rsync -avP --exclude="*.html" --include="*"  username@address.of.hpc:some/remote/location/ local/location/

# If I want to download only the .bam files
# --include="*/" ensures rsync looks inside all folder
# -m removes empty folder
# Logic: if it is a folder, keep it and look inside; then, if a file ends in *.bam, include. For all remaining, exclude.
rsync -avP -m --include="*/" --include="*.bam" --exclude="*"  username@address.of.hpc:some/remote/location/ local/location/
```

2. `rclone`: I use this command to access data from **cloud**, like Google drive or onedrive. It is quick and useful for backing up data.

Before start, use `rclone config` to create a remote.
- To set up a remote for Google drive, follow this [guide](https://rclone.org/drive/). When setting up for Google drive, you will be prompted to configure it as personal or team drive (which allows you to access team drive and folders shared with you). 
- To set up a remote for Onedrive, follow this [guide](https://rclone.org/onedrive/)

Once this step finishes, you can now access files in the remote. For example, if the remote's name is `myremote`:

```bash
# Check the directory, and files
rclone lsd myremote:
rclone ls myremote:
# Or list files inside a specified location
rclone ls myremote:my/own/directory
```
You can then copy data into specified location.

```bash
rclone copy -P --transfers=2 --checkers=8 --multi-thread-streams=8 --retries=5 myremote:some/remote/location/filename* local/location 
```

You can use `--dry-run` in `rclone`, or `-n` in `rsync` to launch a dry run: no files will be actually transferred, and you can check if the commands are correct. 

---
### 4. *Creating and removing temporary files | `mktemp`, `trap`*

In some cases, it might be necessary to create temporary files, when the output cannot be easily passed using *process or command substitution* (see below).

1. `mktemp`: This will create a temporary file or directory (using `-d` flag), by default at the temporary directory, and returns the names of file being created. You can capture the name using command substitution as follows:
```bash
mytmp=$(mktemp)
# do something and save output to $mytmp
```
Using `mktemp` avoids duplication of file names and is safe and easy to use.

2. `trap command signal`: Using `trap` to remove temporary files. `trap` allows you to execute a command whenever a *[signal](https://man7.org/linux/man-pages/man7/signal.7.html)* has been captured. In bash, there is a pseudo signal `EXIT`, which is triggers whenever the script stops or completes. Read a blog [here](https://www.linuxjournal.com/content/bash-trap-command).
  
Combining `mktemp` and `trap` allows us to create a temporary file, and remove it automatically when finishing the script.

```bash
mytmp=$(mktemp) # Create tmp file
trap "rm -f $mytmp" EXIT
# Do anything with mytmp below...
```

The command of `trap` can also be a function, allowing you to do something slightly more complicated.

```bash
function myfunction() {
  local exit_status=$?
  echo "Job completed! Exit status of previous command is ${exit_status}"
  times # CPU runtime
  rm -f $mytmp1 $mytmp2 $mytmp3
}
trap myfunction EXIT
```

## 1.4 Other useful features in Bash

There are too many useful features and logics in bash. Here are a few things I found handy when working with the code.

### 1. Parameter expansion

[Parameter expansion](https://www.gnu.org/software/bash/manual/html_node/Shell-Parameter-Expansion.html) does something to the parameters / variables. There are many examples of [string manipulations](https://mywiki.wooledge.org/BashFAQ/100), but I used the following more frequently (And I am only familiar with them!).

> Note: `${}`, apart from being used in parameter expansion, can also be used in strings to avoid ambiguous interpretation of variables. If you have a variable `$fruit` and an array `$fruits`, isolating the variable like `"These ${fruit}s were harvested"` is clearer and less likely to make errors.

1. **Removing suffix (`%`) / prefix (`#`)**: This is helpful when your variables contain strings in some strict format, and provides a quicker way for parsing the information. Using two symbols `%%` and `##` stands for greedy approach -- they will remove the longest, rather than the shortest match. Note that the pattern is, again, *globbing characters* but not *regex*.\
**Example:** When you want to create an output file based on  names of input files. For instance, I find it helpful to document the processing steps as suffix in the file name, so using parameter expansion or `basename` (see below) will be  helpful. `basename` has its own limitations: it starts a subprocess (slightly slower than expansion); does not remove prefix; and does not support glob characters when specifying suffix (i.e., the suffix must be known). There is a discussion [here](https://unix.stackexchange.com/questions/253524/dirname-and-basename-vs-parameter-expansion) on `basename`, expansion, and `dirname`.
    
```bash
# Let's say we want to extract the sample ID and make a new name
filename="sample1_trimmed_aligned_filtered_blacklist_rm.bam"
fileID="${filename%%_*}" # Removes everything after first "_"
newfilename="${fileID}_peaks.bed"

# What if you have an array of filenames?
# Using expansion avoids the for loop
readarray -t myarray < <(printf "%s\n" *.bam)
cleanarray=( "${myarray[@]%%_*}_peaks.bed" )

# Alternatively, you can use basename command to remove directory and a specified suffix
filename2="some/location/sample2_trimmed.bam"
fileID2=$(basename "$filename2" "_trimmed.bam")

# Same as the following two-step process:
step1="${filename2##*/}"; fileID2="${step1%%_*}"

```

2. **Changing to uppercase (`^`) / lowercase (`,`)**: This is useful when the variables or a list of text has not been formatted in a consistent way. Using `^^` or `,,` converts all characters mattching a pattern (any character if left blank).\
**Example:** Across databases, gene symbols might have different formats. For example, some will make all characters uppercase (for proteins) whereas others only make the first letter uppercase (for genes). You can use the following to make all but first character in lowercase.

```bash
inarray=("egfr" "Myc" "BRCA1")
inarray_all_lower=( "${inarray[@],,}" )
inarray_first_up=( "${inarray_all_lower[@]^}" )
```

3. **Replacing parts of the variable using `/` and `//`**: This is helpful for replacing parts of the variable in a quick and efficient ways. Using the format `${variable/feature/replacement}` only replaces once, whereas `${variable//feature/replacement}` will replace all matches.
**Example**: For example, when you want to switch between "chr" and "chromosome", or only changing suffix part of a file name. Replacement makes me thinks of `sed`, which has similar functionality. I think `//` is simplier, since you need to `echo` the variable as standard input in order to use `sed`, which is a two-step process. On the other hand, `sed` is useful for massive files or texts, whereas `//` only copes with small strings.

```bash
# Let's say this is the file and we want to make it "trimmed"
filename="some/location/sample1_raw.fasta"
newname="${filename/raw/trimmed}"

# This is the same as...
newname=$(sed "s|raw|trimmed|" <<< "$filename")
newname=$(echo "$filename" | sed "s|raw|trimmed|")

# Final note: ${file//raw/trimmed} is the same as "s|raw|trimmed|g" in sed
```

### 2. Process substitutions `<()` and command substitutions `$()`

These are useful in storing the output of commands without creating temporal files. Command substitution stores the printed output as single **variable**, whereas process substitution creates a file that can be passed to commands expecting a **filename**.

There are already many instances of both substitutions earlier in this notebook. As another example:

```bash
# Process substitution is used in commands expecting filenames
bedtools intersect -a <(grep "^chr1" "$bedfile1") -b "$bedfile2" > intersections.bed
# Essentially, the same as:
grep "^chr1" $bedfile1 > $tmpfile
bedtools intersect -a $tmpfile -b $bedfile2 > intersections.bed
rm $tmpfile
# There is also output process substitution >()

# Command substitution can be used in commands expecting variables
echo "The total number of loops in this file is $(wc -l < $loopfile)"
```

## 1.5 Brief notes on HPC

HPC stands for high performance computing. These notes are specific for the HPC cluster at [Imperial](https://www.imperial.ac.uk/computational-methods/hpc/).

There is a very detailed guideline on how to access the cluster [here](https://icl-rcs-user-guide.readthedocs.io/en/latest/hpc/). In brief:

1. Login to a login node using `ssh`: `ssh username@login.cx3.hpc.imperial.ac.uk` Notice, the `username@address` shall also be used when you want to download or upload files from HPC. You can use a different address to access different [CPUs](https://icl-rcs-user-guide.readthedocs.io/en/latest/hpc/getting-started/using-ssh/)
  
> **Login node**: only do simple tasks at the login node, like viewing a file using `head`. For more complicated tasks you should submit them as a **PBS job**

2. Submitting a job with PBSPro workload manager. The syntax is simple and it will return a job ID.
```bash
qsub jobname.pbs
# jobID.pbs
```
For a pbs job, it needs to have a header specifying details and requirement, starting with `#PBS` on each row. Here are some commonly used headers. Delete everything on the right of `# <--` if you want to copy this. 
```bash
#!/bin/bash    # <-- she-bang: specifying interpreter
#PBS -l walltime=HH:MM:SS # <-- Max time the job shall run
#PBS -l select=1:ncpus=N:mem=Mgb # <-- Usually only change N and M and select one node
#PBS -N the_name_of_my_job # <-- Optional: Job name
#PBS -J start-end # <-- Optional: Submitting as job array
```

Some of the specifications are required (`-l`), while others are optional (`-N -J` etc.). 

`-l`: The amount of resoures required depend on the complexity of your script / job. For example a large sequencing data needs longer time to process. [Job sizing](https://icl-rcs-user-guide.readthedocs.io/en/latest/hpc/queues/job-sizing-guidance/) also affects which queue the job enters. I think *small* jobs generally takes shorter queue time than *medium* jobs. 

The goal is to estimate an appropriate runtime and resource to balance the following: **1)** The job doesn't get killed halfway or due to lack of memory; **2)** The queue time is acceptable and you do not overestimate resources too much. If you ask for `ncpus=128` and `walltime=72:00:00` for a simple task, then certainly it can be completed but waiting time would be very, very long.

`-J`: This allows you to access a variable, `$PBS_ARRAY_INDEX` inside your file. In the header you will specify a range (`start` to `end`). By submitting the script you will launch an array of jobs, each with an array index taken from the range. You can also use stepping factor like `start-end:step` or limit number of jobs running at same time using `start-end%max_at_same_time`. Array job is useful when you have many samples and you wish to process them in the same manner, see discussions on `readarray` and `read` + `$PBS_ARRAY_INDEX`. 

**Personally**, I feel that the array queue takes much longer than the normal queue. If you do not have *too many jobs* like only have 4-8 samples that takes 72 hours to complete and many cpus, I would say you can try to submit them as normal jobs by creating a submission script and passing the variable using `-v`. I am not encouraging to never submit jobs as queue jobs!

```bash
# Create a script with resource requirement
# It has a variable, $input, for input file paths
your_pbs_script="my_long_script.pbs"

# You can pass file paths like this
for (path in *.fasta); do
    fileID=$(basename "$path" ".fasta")
    qsub -N "log_${fileID}" -v input="$path" "$your_pbs_script"
done
```

`-W`: This allows the establishment of *dependency* -- for example, you can ask a job to submit only after a previous job enters certain condition. It is useful to process data sequentially if you have all scripts ready. For instance:

```bash
qsub trim_fasta.pbs 
# job1ID.pbs  ## Submitting job1 will return an ID

qsub -W depend=afterok:job1ID align_trimmed.pbs
# job2ID.pbs  ## This will also return an ID, but job2 is waiting for job1 to complete

qsub -W depend=afterok:job2ID align_call_peak.pbs
# job3ID.pbs
```
3. Checking job status using `qstat`: Using `qstat` on the new cluster requires a username to be supplied via `-u` flag, like:
```bash
qstat -u myusername
```
For convenience, you can save this command to a shorter name using `alias`, avoiding typing myusername every time:

```bash
# Do in command line
alias qs='qstat -u myusername' 
qs
```
You can use `-q` flag to check the current queues, for example how many are waiting.

4. Removing a job using `qdel jobID`

## 1.6 Using conda

You can use conda in `wsl`, in HPC login node, or in Windows by downloading Anaconda Prompt. Conda is helpful for regulating the environments i.e. the packages you use. For an old tool, you might want to create a specific environment since it requires many dependencies 

To start using conda, run the following script once:
```bash
module load miniforge/3
miniforge-setup
```
This will set up a directory at your `$HOME` for miniforge. Then, each time you would like to use conda, type the following:

```bash
eval "$(~/miniforge3/bin/conda shell.bash hook)"
```

Then, you can create, delete, or download something inside conda environment.
```bash
# View, create, delete environments
conda info --envs # Listing all conda environments
conda create -n myenv # Create new environment
conda remove -n myenv --all # Remove myenv, and all packages inside

# Activate and deactivate
conda activate myenv # Access packages inside "myenv"
conda deactivate # Leaving the environment

# Setting channel priorities to avoid dependency issues
# Conda-forge > bioconda > default
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge
conda config --set channel_priority strict

# Installing inside package
mamba install package=version # Specified version
mamba install channel::package # Specified channel
conda list package # Check existence and version
which package # Show location

# Exporting environment for reproducibility
conda env export > environment.yml # Export to yml
conda env create --name myenv2 -f environment.yml # Recreate from yml
```