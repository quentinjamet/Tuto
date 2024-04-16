# Basic git command

<!--- /////////////// -->
* **Clone a directory**
	* ```git clone https://github.com/quentinjamet/CDFTOOLS```
	*  By default, git uses the name ```origin``` for a remote, so if you do ```git clone <url>``` it will by default call that remote ```origin``` 
	* Use ```git remote -v``` to see what origin points to


<!--- /////////////// -->
* **Make a branch for local development**
	* ```git checkout branchREF```
	* ```git branch branchDEV```
	* ```git checkout branchDEV```  (to go to the branch)
	* to make the upstream copy on origin: ```git push --set-upstream origin branchDEV```


<!--- /////////////// -->
* **Diff local repo with ```origin```**
From https://stackoverflow.com/questions/11935633/git-diff-between-a-remote-and-local-repository
	* ```git diff remote/my_branch my_branch```
OR
	* ```git fetch origin your_branch```:  fetch the branch named 'your_branch' from the remote named 'origin' ; then
	* ```git diff --summary FETCH_HEAD```: display diff between ```local``` and ```origin```. You can also use the option ```--stat``` for more details.


<!--- /////////////// -->
* **Update local repo with ```origin```**
	* ```git pull```: update local repo
OR
	* after a ```get fetch``` (see below), complet with ```git merge```


<!--- /////////////// -->
* **Commit and push local devs to origin
	* ```git status```: display local changes as compared to ```origin```
	* ```git add your_file```: will add ```your_file``` to the commit
	* ```git commit -m 'a comment'```: commit change
	* ```git push```: push local commit to ```origin```


<!--- /////////////// -->
* **Save (or hide) local changes without commiting them**

very useful to change between branches with ongoing devs

	* ```git stash```
	* ```[git stash pop to 'reactivate']```
	* ```git stash list```
	* ```git stash show```

