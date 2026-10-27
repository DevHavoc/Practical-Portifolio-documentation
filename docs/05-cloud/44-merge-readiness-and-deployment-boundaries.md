# Checking What a Merge and Deployment Will Include

## Problem

Before deploying, we needed to know whether the branches would merge cleanly and whether all the intended changes would be included.

## What I checked

I refreshed the remote branches, compared their histories and simulated the merge. It had no conflicts, and its files matched the reviewed staging version.

I also checked local changes separately. The background worker was still local, so a deployment built from the remote branch would not include it. A local publish package can contain different files from a build made from that branch.

## Database and access checks

The access module needed more than a clean code merge. Existing admins were missing an access grant, so I added a migration and tested the database upgrade on a temporary PostgreSQL instance.

The team used a publishing flow that the old pipeline file did not describe. The actual source branch, application settings, runtime and database migration history therefore needed checking in that publishing flow.

## Result

The corrected admin-access branch also passed the merge simulation and kept only the intended migration. The review did not deploy the application.

A clean merge confirms that Git can combine the files. Build checks, database tests and deployment checks answer different questions. None of them alone guarantees that production will work with every permission, setting and record.

The commands below show the branch comparison and merge simulation used in the review. They are review commands, rather than application code.

## Code example

```bash
git status --short
git rev-list --left-right --count origin/SPRINT_3...origin/SPRINT_3-HML
git merge-tree --write-tree origin/SPRINT_3 origin/SPRINT_3-HML
git rev-parse 'origin/SPRINT_3-HML^{tree}'
```
