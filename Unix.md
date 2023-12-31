# Basic Unix command

The following list is extracted from ```https://www.unixtutorial.org/basic-unix-commands```,
where you can find complementary Unix commands.

* **To navigate in a terminal**
  * ```pwd```: get current directory
  * ```ls```: list files and directories
  * ```cd```: change directory
  * ```cp```: copy files (cp -r for directories)
  * ```rm```: remove files (rm -rf for directories and forcing)
  * ```mv```: rename or move files and directories to another location
  * ```mkdir```: make new directory
  * ```grep```: search for patterns in text files (with ```-i``` option to confond upper and lower case letters) (ex: ```grep -i tOtO ./*```)
  * ```history```: show history of previous commands
  * ```env | grep SHELL```: know with SHELL you use

* **To run croco and print the output into a file**
  * ```$ ./croco croc.in &> basin.out```
  * Suspend a process and put it in backgraound, use ```ctrl+z``` then ```bg```
  * Get the PID of a running process and see on how many cores:
    ```
    top
    ```
  * Stop a process :
    ```
    kill -9 PID
    ```
