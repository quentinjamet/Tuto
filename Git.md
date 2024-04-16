# Basic git command

## Clone a directory
	```git clone https://github.com/quentinjamet/CDFTOOLS```





## Make a branch for local development
* ```git checkout branchREF```
* ```git branch branchDEV```
* ```git checkout branchDEV```  (to go to the branch)
* to make the upstream copy on origin: ```git push --set-upstream origin branchDEV```


## Save (or hide) local changes without commiting them
very useful to change between branches with ongoing devs
* ```git stash```
* ```[git stash pop to 'reactivate']```
* ```git stash list```
* ```git stash show```

