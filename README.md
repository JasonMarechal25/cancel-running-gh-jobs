# cancel-running-gh-jobs
One liner to cancel running action on previous commits

## Why

When pushing new commits on github, jobs are started. If you push a new set of commits a new set of jobs is queued. There exists a github option to automatically cancel "old" jobs but it may not be activated on a repository or even unwanted.

## What

This script will cancel jobs running for the current branch, except for jobs corresponding to the top most commit.

## How

We're using gh command line and jquery

1) Get current branch

`$(git branch --show-current)`

2) Get active workflows on current branch

`gh run list --branch $(git branch --show-current) --json databaseId,status,name,createdAt`

3) Filter Active Workflows. No need to cancel finished jobs

`[ .[] | select(.status == "in_progress" or .status == "queued") ]`

4) Group by Workflow Type to be able to cancel all except the newest one of each type.

`| group_by(.name)`

5) Exclude newest job

`| map(sort_by(.createdAt) | reverse | .[1:][])`

6) Get id

`| flatten | .[].databaseId`

7) Cancel

`| xargs -I {} gh run cancel {}`

## Limitation

Need to be run at root repository level (next to .git)

## Improvement

* Get upstream name instead of assuming upstream==local
* Add arguments to control some options, like giving a branch name to avoid having to checkout.
