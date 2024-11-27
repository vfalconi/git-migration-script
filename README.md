# Gitea/Forgejo to GitLab migration script

```bash
# when you're ready.
$ bash migrate
```

## Assumptions

This migration script assumes a number of things:

1. You don't have a more elegant solution already.
2. You are wanting to migrate all of the repos owned by a list of users from Gitea/Forgejo (`config:SOURCE_GIT`) and into GitLab (`config:DESTINATION_GIT`). It does not matter if you are self-hosting or not, as long as you have both API and basic repo access.
3. You don't care about migrating all branches, and preserving `main` only is fine.
4. Your GitLab groups'/users' names (`config:NAMESPACES`) match your Gitea organizations'/users' names (`config:OWNERS`) identically. See below for more.
5. You've stored your GitLab user/pass, if only temporarily, so you don't have to enter it on every `push`.
6. You have `jq`, `git`, and `curl` installed.
7. You have admin access in GitLab.
8. You have a GitLab personal access token with `api` scope (`config:GITLAB_PAT`).

## What happens

This migration script does the follow:

1. Retrieves a list of repositories from Gitea/Forgejo via API. This ends up being a list of `owner/repo` strings saved into `repos.txt`.
2. `repos.txt` is read into a loop that iterates on each repo:
   1. Create `repo` in destination GitLab via API.
   2. Clone `repo` from Gitea.
   3. Change `origin` in `repo` to GitLab.
   4. `push main` to GitLab.

## Configuration

You need to add a `config` bash script with the following contents:

```bash
#!/bin/bash

SOURCE_GIT=https://gitea
# Gitea/Forgejo repo owners
OWNERS=("[group name]" ...)

# GITLAB_PAT is NOT the same as your user PAT
GITLAB_PAT="[your GitLab API PAT]"
DESTINATION_GIT=https://gitlab

# GitLab groups and their namespace IDs
declare -A NAMESPACES=( ["group"]="namespace_id" ... )
```

## LOVE YOURSELF, OR MATCHING ORGANIZATIONS TO GROUPS

Ignoring nuance, Gitea/Forgejo organizations are the same as GitLab groups.

In a perfect world, you're moving orgs into groups and the names are the same and it's no problem.

In an imperfect world, decide on your GitLab groups' names then change the Gitea/Forgejo organizations to whatever they will be in GitLab.

Otherwise you're going to need to do weird if-then or something and it just isn't worth it.

You're leaving Gitea/Forgejo behind, who cares what it looks like when the migration is over.
