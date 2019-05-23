# GIT Notes

* ### Change a global configuration setting
    * `git config --global user.name = 'Scott Crowley'`
    * `git config --global user.email = 'scott@scottcrowley.com'`
* ### Create a new repository (locally)
    * `git init` creates a new repository (.git directory) in the existing directory
* ### Various commands
    * `git status` Shows status of all files. Same as `svn status`
    * `git add {filename}` Adds the designated file to the staging area. Same as `svn add {filename}`
    * `git add .` or `git add -A` Adds all unstaged files to the staging area.
    * `git log` Shows all commits. Same as `svn log`
        * add `--oneline` to have to each log entry appear on one line to make it simpler to read
        * add `--graph` to show a graphic representation of all the various commits. Helpful when dealing with branches.
        * add `--decorate` to show you where the `HEAD` is currently pointing
    * `git commit -m 'commit message'` Commits all currently staged files. Same as `svn commit -m "commit message"`
    * `git commit` By itself, will open the commit message in a default editor where a longer message can be added. If adding a long detailed message, it is good practice to add the title of the change on the first line then followed by two new lines and a more detailed description of the commit
    * `git diff {filename}` Shows difference in {filename} since last commit. Same as `svn diff {filename}`
    * `git reset --soft {commit identifier number}` Resets the working directory to the chosen commit but leave any changes, since that commit, unchanged. Allows for rolling back a commit but keeping the changes in the working directory so they can be recommitted later.
    * `git reset --hard {commit identifier number}` **BEWARE**-Resets the working directory to the chosen commit and removes and changes that were made since that commit. This can be used to completely reset the working directory back to the exact state of the designated commit. 
    * `git reflog` Allows you to see a log file of everything that has been done, including any changes that were done using `git rebase -i master`
    * `git fetch` Fetch all changes that have been done on the remote but doesn't modify your working directory. Allows you to see where the remote and local branches stand with each other.
        * add `--all` to fetch all remote branches at one time instead of specify individual remotes.
    * `git pull` Does the same as `git fetch` except it also does a `git merge` on the specified branch, which will update your local working directory with the data on the remote.
* ### Creating branches
    * `git checkout -b feature-name-of-featue` Creates the branch and switches current working directory to that branch. All adds and commits are handled the same.
    * `git checkout master` Changes working directory to the `Master` branch
    * `git merge feature-name-of-featue` If this is done in the Master branch then the designated branch is merged back into the `Master` Branch
    * `git branch -d feature-name-of-featue` Deletes the branch. This should be done after a merge has been performed.
    * To merge the branch back into the master but have all the commits for the feature squashed into a single commit:
        * Switch back to the master branch `git master`
        * Then run the merge to combine the feature back into the master branch 
            * `git merge --squash feature-name-of-feature`
        * If you get a message `Automatic merge went well; stopped before committing as requested` then:
            * `git commit -m "merged feature back into master"` or something more descript
* ### Rebasing
    * When you rebase a branch, it is like taking all of the commits in the branch and reapplying them to the latest commit of the master branch. It is a way to keep the log history clean and simple. Make sure not to do an Rebasing when working with a public repository or when on a team since it can cause a lot of confusion about the commit history.
    * Type this in a branch working copy `git rebase master`
        * `git rebase -i master` Performs an interactive rebase which gives you the option to change commit messages or to combine various commits into one another.
    * The switch to the master branch `git checkout master` or `git master`
    * Then run the merge with the previous branch `git merge feature-name-of-feature`
        * You could also rebase the features commits into the master, at this point, by using `git rebase feature-name-of-feature`
* ### Saving a current working state without making a commit to the repository. (`STASHING`) Stashes can be applied to different branches as well.
    * `git stash` Stashes the current state of the working directory into a temporary holding area. This is useful if you need to work on a different branch but you don't want to commit what you are currently working on. 
    * `git stash list` Shows all the stashes that have been made in the current branch
    * `git stash apply` Retrieves the latest stash and applies it to the working directory
    * `git stash apply {stash identifier}` Retrieves the stash with the chosen identifier (found by running `git stash list`. i.e. stash@{3} ) and applies it to the working directory
    * `git stash pop` Same as apply, but deletes the stash after it has been applied
    * `git stash pop {stash identifier}` Same as apply `{stash identifier}`, but deletes the stash after it has been applied
    * `git stash drop` Drops or deletes the latest stash.
    * `git stash drop {stash identifier}` Drops or deletes the stash with the chosen identifier (found by running `git stash list`. i.e. stash@{3} ).
    * `git stash branch feature-name-of-feature` Retrieves the current stash and applies it to a new branch called `feature-name-of-feature`.
* ### Working with Github
    * A repository needs to be first created in your Github account. Notate the github.com url that is generated. `{GitHub url}`
    * If you have an existing project with git already initialized then you need to do the following (This only needs to be done once per project):
        * `git remote add origin {GitHub url}` This sets the remote path to the repository on GitHub
        * `git push -u origin master` This pushes the master branch to the GitHub repository specified in the previous step
            * this will ask for your GitHub `username` and `password`
    * After the origin has been set (previous steps), then the following commands can be used
        * `git push` Pushes all local branches to the repository that match names already setup
            * This only works if the default `push method` is set to `matching` (default). `git config --global push.default matching`
            * If `git config --global push.default simple` is done, then the push command will only push the current branch to the matching remote branch
        * `git clone {GitHub url}` Copies the selected repository to the local machine
* ### Pull Request Process
    * This example will cover forking someone elses project to allow us to perform our changes and then open a pull request to have the changes added to the originators code base. 
        * The process is similar for when you are working on a project that you are a member of. In this case you would create a branch and then create a pull request based on the modified version of your branch
    * Once you have forked the project (Clicking on the Fork icon at the top right of the main page of the project) you want to contribute to you will need to then clone the project to your local machine from your GitHub account.
        ```
        git clone https://github.com/youraccount/forked-project.git
        ```
    * Now the project is on your local machine and you can modify and perform commits as you would with any project. It's a good idea to have very detailed commit messages in case the person that will review the pull request wants to have all the detail about your changes.
    * When all changes have been completed and the final commit is made, you need to initiate a pull request from the original projects pull request page by clicking on the green New pull request button.
    * Since we are not making a pull request from our own repository then we need to compare across forks. There should be a link that says `compare across forks` towards the top part of the new pull request page. Click that.
        * The first drop down is for the base-fork which is the project you originally forked, followed by a drop down for which branch. If you are contributing to the master branch, select that or select the branch you want to contribute to.
        * The head fork drop down is your own repo that you've been working on followed by a drop down for the branch you want to add to the pull request.
        * After all the drop downs are updated, you should see something from Git saying `Able to merge` in green. Which means everything is go to go.
    * Now click on Create pull request button.
        * This will open a form that contains the last commit message you had, which you can change or leave as is
        * You should leave very detailed information about what the pull request is going to do. Provide examples and be very clear about all the details. As Jeffery Way has said, speak to me as if I have no knowledge of the project and he is a beginner.
    * Once the pull request is submitted, then you wait for the original contributor to either ask for more details, changes to be made, acceptance or denial of the request.
* ### Notes:
    * Undo last commit
        * `git reset --soft HEAD~`
        * If Github is being used and the commit was pushed then you can undo it with
            * `git push origin master --force`
    * Sometimes when there is a conflict on `git pull`, it will put you into a `Vim` editor mode waiting for your to change a `commit message`. To exit and save the default message use:
        * `esc` to exit `insert mode`, if active
        * type `:` should put the cursor and the bottom left
        * type `x` followed by `return` to save
    * `Git Fork` will copy a repo to your `username`. i.e. If you fork Laravel, you will have access to a Laravel repo under your username.
* ### Git Work Flow from Laracasts video: <https://laracasts.com/series/how-do-i/episodes/20>
    * When working on someone else's project and you want to submit a PR for a new feature or bug fix. Using Laravel Framework as an example and we are going to submit a bug fix
        * Go to the Laravel Framework page on GitHub and click `Fork`. This will make a copy of the repo in your own account.
        * Clone the new repo, under your account to your local machine.
            * The default branch in this case is the last release. If there is current work being done then this is being applied to the master branch and that'll need to be checkout out before creating the bug fix, unless the bug fix pertains to a certain release.
            * In the local directory type `git checkout master`
        * Fix the needed files
            * If the effected files have a related test, then update that as well and `commit` and `push` your changes to your `GitHub Repo`
            * If you want to make sure your bug fix works and have another copy already checked out, you can use your modified version instead on the version of the package from packagist. <https://getcomposer.org/doc/05-repositories.md#vcs>
                * update the `composer.json` file and add or update the `repositories` key on the root of the doc.
                    ```
                    "repositories" : [
                    {
                    "type": "vcs",
                    "url": "https://github.com/yourusername/therepoyouforked"
                    }
                    ],
                    ```
                * Now run `composer update` and it will pull in your changes from the repo provided.
        * Go to GitHub and then to your account for the repo that your forked.
        * From the `Branch Menu`, make sure you are on the `master branch`.
            * You should see something that says that *This branch is 1 commit ahead of the original repo you forked*.
        * On the same line as the message above, you should be able to click a link for a `Pull Request`.
        * It should open a form showing the changed files along with the last commit message.
            * It the description field be very precise and describe exactly what the PR is doing and why it's being done. As JW says, treat him like a 4 year old and really break it down.