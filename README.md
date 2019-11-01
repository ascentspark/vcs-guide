# Git Guidelines

###Table of content:
1. [Git configuration](#git-configuration)
2. [Master Branch](#master-branch)
3. [Branching Strategy](#branching-strategy)
4. [Commit messages](#commit-messages)
5. [Fixing conflicts](#fixing-conflicts)
6. [Rebase Strategy](#rebase-strategy)


## Git configuration

```
git config --global user.name <yourusername>
git config --global user.email <youremail>
```

   **Example**:
   `git config --global user.name xyz`


   &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`git config --global user.email xyz@example.com`

## Master branch

The `master` branch should be kept stable at all times.

Master reflects code currently running on production.

## Branching Strategy

Branches are used to develop features isolated from each other.
The master branch is the "default" branch when you create a repository.
Use other branches for development and merge them back to the master branch upon completion.

   1. **Creating a branch and switching to it**: `git checkout -b branch_name`
   2. **Switch back to master**: `git checkout master`
   3. **Deleting a local branch**: `git branch -d branch_name`
   4. **Pushing a branch**: A branch is not available to others unless you push the branch to your remote repository. `git push origin branch_name`

**Branch Naming Convention**:-
* Use your initials and grouping tokens (words) at the beginning of your branch names(see example below).
* Define and use short lead tokens to differentiate branches in a way that is meaningful to your workflow.
* Use slashes to separate parts of your branch names.
* Do not use bare numbers as leading parts. 
* Avoid long descriptive names for long-lived branches.

**Grouping Token Examples**:

|Prefix   | Use Case                                              |
|:-------:|-------------------------------------------------------|
| wip     | Works in progress; stuff I know won't be finished soon|
| feature | Feature I'm adding or expanding                       |
| bugfix  | Bug fix or experiment                                 |
| hotfix  | Immediate bug fix after merging the branch to master  |
| junk    | Throwaway branch created to experiment                |

**Example of few Good and Bad branch names**:

Suppose Your name is Jhon Doe, so take `jd` as your initial.

  **Good**:

    jd/feature/registration
    jd/bugfix/login_blocker
    jd/hotfix/text_change

  **Bad**:
  
  `jd/bugfix/crash-when-unformatted-disk-inserted`

## Commit messages

We use following format for our commit messages:

```
 - Capitalized, short (50 chars or less) summary

More detailed explanatory text, if necessary.

```

1. Summary is maximally 80 characters long, from capital letter, no dot at the end
2. We prepend a hyphen(-) at the beginning of summary
3. If you want a commit message to be reflected on changelog then start the message with a tilde sign (~), see below examples.
4. Next lines are description explaining the details
5. Write present-tense, imperative-style commit messages

   **GOOD**:
   `- Add currency service`

   &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;or

    `~ Add registration and login feature`

   **BAD**:
      `-- adds currency service`

## Fixing conflicts

First run `git status`, it will tell you which file was `both modified`. Open those files one by one in them, you'll see lines like this:
```
<<<<<<<<<<< HEAD
use std::cmp::{max, min};
===========
use std::cmp::{max, PartialEq};
>>>>>>>>>>> Your Commit
```

This means that in the commit on remote, the author wants to add `min` to the `use` line, but in your commit you want to add `PartialEq`. (Lines between the `<<<` and `===` are the version on `remote`; the lines between `===` and `>>>` are your local version.) A way to fix this is to include both, so you can delete all the lines from the `>>>` to `<<<`, and replace them with the correct code:

```
use std::cmp::{max, min, PartialEq};
```

After you fixed all the conflicts, you can run `git add <the files you edited>`, then `git rebase --continue`. You might need to repeat this action multiple times until every conflict is resolved. (In case you messed up, you can always run `git rebase --abort` to start over).

## Rebase Strategy

***a. Always make sure that all your working commits should be made as a single commit by squashing and then send merge request***

**Squashing**:

If your change consists of three commits `W`, `X` and `Y`, we want to squash them into a single commit `Z`.

```
P -> Q -> R
           \
            .--> W -> X -> Y
```

```
P -> Q -> R
           \
            .--> Z
```

To achieve this, we can use the `git rebase -i` command.

1. First we need to identify how many commits you want to merge, which is 3(commit W, X and Y) in our example.
2. Run `git rebase -i HEAD~3`, this will bring up your default text editor with a 
content like:


        pick 7de252c W
        pick 02e5bd1 X
        pick 034e534 Y

        # Rebase 170afb6..02e5bd1 onto 170afb6 (2 command(s))
        #
        # Commands:
        # p, pick = use commit
        # r, reword = use commit, but edit the commit message
        # e, edit = use commit, but stop for amending
        # s, squash = use commit, but meld into previous commit
        # f, fixup = like "squash", but discard this commit's log message
        # x, exec = run command (the rest of the line) using shell
        # d, drop = remove commit
        #
        # These lines can be re-ordered; they are executed from top to bottom.
        #
        # If you remove a line here THAT COMMIT WILL BE LOST.
        #
        # However, if you remove everything, the rebase will be aborted.
        #
        # Note that empty commits are commented out


3. Keep the first commit as `pick`, and change all the other `pick` to `squash` (or `s` for short):


        pick 7de252c W
        s 02e5bd1 X
        s 034e534 Y

        # Rebase 170afb6..02e5bd1 onto 170afb6 (2 command(s))
        ...


4. Now save and quit the text editor, the rebase will run until the end. You might meet conflicts like you do in rebasing. Fix then using the same method described in the previous section.
5. After the rebase is finished, the editor will pop-up again, now you can write the commit message for the new commit `Z`.
6. `git push -f` to push the squashed commit to GitLab (and update the branch).

If you made any mistake right after you run step 2, you can abort by deleting every line in the text editor then save and exit. If you mess up fixing the conflicts, you can also run `git rebase --abort` to reset everything and start over.

***b. Before sending a merge request the branch should always be rebased on top of master***

**How to perform the above said**:

`git pull --rebase origin master`
