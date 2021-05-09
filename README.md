
# Aurora FE CI/CD v1

Within is a proposed CI/CD Setup and several code standards to follow, so that a project remains both maintainable and scalable.

## Table of Contents

## Terminology

| Term                            | Meaning                                                                                                                                                                                                                 |
| ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Continious Integration (**CI**) | Continuous Integration (CI) automates the process of testing and building your code before merging it. In practice, developers should commit or integrate their changes to the main shared repo once-per-day (or more). |
| Continuous Deployment (**CD**)  | Continuous Deployment (CD) automatically releases new production code to users. It is the step that happens after new code has been integrated.                                                                         |

## Git

### Git Rules

There are a set of rules to keep in mind:

* Perform work in a feature branch.

    _Why:_
    >Because this way all work is done in isolation on a dedicated branch rather than the main branch. It allows you to submit multiple pull requests without confusion. You can iterate without polluting the master branch with potentially unstable, unfinished code. [read more...](https://www.atlassian.com/git/tutorials/comparing-workflows#feature-branch-workflow)
* Branch out from `develop`

    _Why:_
    >This way, you can make sure that code in master will almost always build without problems, and can be mostly used directly for releases (this might be overkill for some projects).

* Never push into `develop` or `master` branch. Make a Pull Request.

    _Why:_
    > It notifies team members that they have completed a feature. It also enables easy peer-review of the code and dedicates forum for discussing the proposed feature.

* Update your local `develop` branch and do an interactive rebase before pushing your feature and making a Pull Request.

    _Why:_
    > Rebasing will merge in the requested branch (`master` or `develop`) and apply the commits that you have made locally to the top of the history without creating a merge commit (assuming there were no conflicts). Resulting in a nice and clean history. [read more ...](https://www.atlassian.com/git/tutorials/merging-vs-rebasing)

* Resolve potential conflicts while rebasing and before making a Pull Request.
* Delete local and remote feature branches after merging.

    _Why:_
    > It will clutter up your list of branches with dead branches. It ensures you only ever merge the branch back into (`master` or `develop`) once. Feature branches should only exist while the work is still in progress.

* Before making a Pull Request, make sure your feature branch builds successfully and passes all tests (including code style checks).

    _Why:_
    > You are about to add your code to a stable branch. If your feature-branch tests fail, there is a high chance that your destination branch build will fail too. Additionally, you need to apply code style check before making a Pull Request. It aids readability and reduces the chance of formatting fixes being mingled in with actual changes.

* Use [this](./.gitignore) `.gitignore` file.

    _Why:_
    > It already has a list of system files that should not be sent with your code into a remote repository. In addition, it excludes setting folders and files for most used editors, as well as most common dependency folders.

* Protect your `develop` and `master` branch.
  
    _Why:_
    > It protects your production-ready branches from receiving unexpected and irreversible changes. read more... [Github](https://help.github.com/articles/about-protected-branches/), [Bitbucket](https://confluence.atlassian.com/bitbucketserver/using-branch-permissions-776639807.html) and [GitLab](https://docs.gitlab.com/ee/user/project/protected_branches.html)

<a name="git-workflow"></a>

### Git workflow

Because of most of the reasons above, we use [Feature-branch-workflow](https://www.atlassian.com/git/tutorials/comparing-workflows#feature-branch-workflow) with [Interactive Rebasing](https://www.atlassian.com/git/tutorials/merging-vs-rebasing#the-golden-rule-of-rebasing) and some elements of [Gitflow](https://www.atlassian.com/git/tutorials/comparing-workflows#gitflow-workflow) (naming and having a develop branch). The main steps are as follows:

* For a new project, initialize a git repository in the project directory. __For subsequent features/changes this step should be ignored__.

   ```sh
   cd <project directory>
   git init
   ```

* Checkout a new feature/bug-fix branch.

    ```sh
    git checkout -b <branchname>
    ```

* Make Changes.

    ```sh
    git add <file1> <file2> ...
    git commit
    ```

    _Why:_
    > `git add <file1> <file2> ...` - you should add only files that make up a small and coherent change.

    > `git commit` will start an editor which lets you separate the subject from the body.

    > Read more about it in *section 1.3*.

    _Tip:_
    > You could use `git add -p` instead, which will give you chance to review all of the introduced changes one by one, and decide whether to include them in the commit or not.

* Sync with remote to get changes you’ve missed.

    ```sh
    git checkout develop
    git pull
    ```

    _Why:_
    > This will give you a chance to deal with conflicts on your machine while rebasing (later) rather than creating a Pull Request that contains conflicts.

* Update your feature branch with latest changes from develop by interactive rebase.

    ```sh
    git checkout <branchname>
    git rebase -i --autosquash develop
    ```

    _Why:_
    > You can use --autosquash to squash all your commits to a single commit. Nobody wants many commits for a single feature in develop branch. [read more...](https://robots.thoughtbot.com/autosquashing-git-commits)

* If you don’t have conflicts, skip this step. If you have conflicts, [resolve them](https://help.github.com/articles/resolving-a-merge-conflict-using-the-command-line/)  and continue rebase.

    ```sh
    git add <file1> <file2> ...
    git rebase --continue
    ```

* Push your branch. Rebase will change history, so you'll have to use `-f` to force changes into the remote branch. If someone else is working on your branch, use the less destructive `--force-with-lease`.

    ```sh
    git push -f
    ```

    _Why:_
    > When you do a rebase, you are changing the history on your feature branch. As a result, Git will reject normal `git push`. Instead, you'll need to use the -f or --force flag. [read more...](https://developer.atlassian.com/blog/2015/04/force-with-lease/)

* Make a Pull Request.
* Pull request will be accepted, merged and close by a reviewer.
* Remove your local feature branch if you're done.

  ```sh
  git branch -d <branchname>
  ```

  to remove all branches which are no longer on remote

  ```sh
  git fetch -p && for branch in `git branch -vv --no-color | grep ': gone]' | awk '{print $1}'`; do git branch -D $branch; done
  ```

### Writing good commit messages

Having a good guideline for creating commits and sticking to it makes working with Git and collaborating with others a lot easier. Here are some rules of thumb ([source](https://chris.beams.io/posts/git-commit/#seven-rules)):

* Separate the subject from the body with a newline between the two.

    _Why:_
    > Git is smart enough to distinguish the first line of your commit message as your summary. In fact, if you try git shortlog, instead of git log, you will see a long list of commit messages, consisting of the id of the commit, and the summary only.

* Limit the subject line to 50 characters and Wrap the body at 72 characters.

    _why_
    > Commits should be as fine-grained and focused as possible, it is not the place to be verbose. [read more...](https://medium.com/@preslavrachev/what-s-with-the-50-72-rule-8a906f61f09c)

* Capitalize the subject line.
* Do not end the subject line with a period.
* Use [imperative mood](https://en.wikipedia.org/wiki/Imperative_mood) in the subject line.

    _Why:_
    > Rather than writing messages that say what a committer has done. It's better to consider these messages as the instructions for what is going to be done after the commit is applied on the repository. [read more...](https://news.ycombinator.com/item?id=2079612)

* Use the body to explain **what** and **why** as opposed to **how**.

## Tech Stack

**Client:** Nuxt, Vue, TailwindCSS

**Tools:** Github Actions, Docker, circleci

| Tool             | Description                                              |
| :--------------- | :------------------------------------------------------- |
| `Github Actions` | **Required**. Id of item to fetch                        |
| `StoryBlock`     | Used to show catelog and display global brand components |

## Environments

| Environment | Description                                                                                                                | URL |
| ----------- | -------------------------------------------------------------------------------------------------------------------------- | --- |
| DEV         | The development environment is a workspace for developers to make changes without breaking anything in a live environment. | N/A |
| TEST        | The development environment is a workspace for developers to make changes without breaking anything in a live environment. | N/A |
| STAGE       | allow the application's main users to test new features before they are pushed into the production environment             | N/A |
| PROD        | The live environment made available to all users                                                                           | N/A |

## Github Actions [automation]

### GA Terms

| Term          | Purpose                                                                                                                                                                                                                                                                                                                                                                         |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Runners**   | These are hosted virtual operating systems that could run commands to carry out a build process.                                                                                                                                                                                                                                                                                |
| **Workflows** | These are laid out instructions that give the Github Action application on a runner. Workflow instructions are defined in a YAML file and live inside of the .github/workflows folder in a git repository. The name of a workflow file does not have a correlation with the intention of the file, as Github parses and runs every file inside of the .github/workflows folder. |
| **Events**    | For a Workflow file to be processed and executed, an event is required.                                                                                                                                                                                                                                                                                                         |
| **Jobs**      | Async or Sync series of steps carried out within any given workflow.                                                                                                                                                                                                                                                                                                            |
| **Steps**     | These are the single elements that make up a job. A step groups the actions that are carried out on a runner.                                                                                                                                                                                                                                                                   |
| **Actions**   | An action is an element of a step and is basically a command.                                                                                                                                                                                                                                                                                                                   |

### Creating and Maintaining Secrets

Throughout our CI/CD System there are a number of secrets which must be created and maintained so that they can be used in our github actions.

**Important : Keep secrets, managed by implimenting privilages on them**
[Implimenting Lease Privilage for secrets in github actions](https://github.blog/2021-04-13-implementing-least-privilege-for-secrets-in-github-actions/)

#### Managing Secrets

1. Go to the github repositiory you wish to work on.
2. Go to the `Settings` Tab.
3. Select `Secrets`
4. Name your secret
   1. **I.E** 'MY_SECRET_KEY'
5. Add the secret value below the name.
6. Press save.

**NOTE : By default, secrets are not made available to workflows and jobs, to have access to secrets, you have to request them at the steps that they are required.**
[Managing Secrets](https://docs.github.com/en/actions/learn-github-actions/managing-complex-workflows#storing-secrets)



### Workflows

#### Project Aurora Github Action table


| Workflow                       | Purpose                                                                                                                                                                                                                                                                                                    | Branch     | Initiate With | Fail                                                                                   | Pass                                                                 |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- | ------------- | -------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| branch-overwatch.yml(optional) | A new branch is created within the repository                                                                                                                                                                                                                                                              | ${x}       | created       | N/A                                                                                    | Notify stakeholders that work has begun through a slack message etc. |
| dev-pull.yml                   | Tests pull requests into the TEST branch against developers feature pr                                                                                                                                                                                                                                     | TEST       | pull_request  | denys the pr                                                                           | marks the pr as succesfull                                           |
| dev-push.yml                   | When the maintainer of the TEST branch approves pull requests into TEST this action will kick off a build and deploy it to the TEST server                                                                                                                                                                 | push       | push          | doesnt deploy the built code to dev server, and requires failed conditions to be fixed | Deploys built code to dev server                                     |
| stage-push.yml                 | After the TEST server has been verified by the QA Team the repo maintainer can initiate a push request into STAGE (or Prod if no UAT testing required) this workflow will perform tests, and also run utility scripts such as compressing images, finally it will produce a new build and deploy it to AWS | STAGE      | push          | Create issues, dissalow from code deployment                                           | Deploys code to STAGE server                                         |
| prod-push.yml                  | After the TEST server has been verified by the QA Team the repo maintainer can initiate a push request into STAGE (or Prod if no UAT testing required) this workflow will perform tests, and also run utility scripts such as compressing images, finally it will produce a new build and deploy it to AWS | PRODUCTION | push          | Create issues, dissalow from code deployment                                           | Deploys code to STAGE server                                         |

#### branch-overwatch.yml

**Features**

**Flow**


**code**

```
todo
```

#### dev-pull.yml

**Features**

**Flow**


**code**

```
todo
```

#### dev-push.yml

**Features**

**Flow**


**code**

```
todo
```

#### stage-push.yml

**Features**

**Flow**


**code**

```
todo
```

#### prod-push.yml

**Features**

**Flow**

**code**

```
todo
```

### Getting Started

Before entering the development cycle, or contributing to the project in any way ensure that you are fully aware of both your role, and that you have the tools for the job.

#### Developer Checklist

[x] IDE Setup and configured to match MK Organization
[x] Access to Workflow tools
[x] Environment variables
[x] Clear understanding of Git
[x] Install and set up nvm (__node version manager__)

*Dev Environment Variables*
To work in dev you will need the following environment variables

|  Key  | Value | Description | Secret |
| :---: | :---: | :---------: | :----: |
| todo  | todo  |    todo     | __Y__  |

#### Sys Admin Checklist

[x] IDE Setup and configured to match MK Organization
[x] Access to Workflow tools
[x] Environment variables
[x] Clear understanding of Git

*Dev Environment Variables*
To work in dev you will need the following environment variables

|  Key  | Value | Description | Secret |
| :---: | :---: | :---------: | :----: |
| todo  | todo  |    todo     | __Y__  |

*Test Env Variables*
To work in dev you will need the following environment variables

|  Key  | Value | Description | Secret |
| :---: | :---: | :---------: | :----: |
| todo  | todo  |    todo     | __Y__  |

#### QA Checklist

[x] Ensure you have Accessibility testing tools (Axe)
[x] Ensure you understand the *feature* scope
[x] Ensure you have a basic understanding of git and git workflows

### Development Branch

**Who Belongs in this branch?**

* Development Team

**Assumptions**

* [x] Everything Goes
* [x] A developer has a fully, scoped 'feature' or task to complete.
* [x] **Code Ownership** : Individual developers own their code in this branch, they are required to write all tests before marking their task is complete.
* [x] **Commenting** : Don't use comments as an excuse for a bad code. Keep your code clean. Additionally, don't use clean code as an excuse not to comment.

**Path to Test**

To make it to stage a piece of code must meet the following guidlines.

* [x] Unit Tests Written
* [x] Basic Integration Testion Complete
* [x] Code Style and formatting compatible with project guidlines
* [x] Remove All Console Logging
* [x] Ensure No Secrets have been included

#### Overview

1. Developers are assigned tasks
2. Developers Create a feature branch for their task
3. Developers Complete their task, and write tests for their new code.
4. Developers Open A pull request into the main development working branch
5. Github Action  

#### New Feature Dev steps

**First** All 'Features' Are developed off of the master branch (latest code).

```
# Switches repo to master branch
git checkout master

# Pulls latest
git fetch origin

# Resets repos local copy of master to match latest
git reset --hard origin/master
```

**Second** Create a new branch for your feature

```
#Checks out a branch called feature/${your-feature-name}

git checkout -b feature/example-new-feature

```

**While Developing** Update, add, commit, and push changes

```
git status
git add <filename>
git commit
```

**Important** Push your new feature branch to remote to allow other developers to see the work in progress as well as backup your code
```
git push -u origin feature/${your-feature-name}
```

**Finally** When you are finished with your feature, create a pull request on the Main Development branch. If no conflicts arise, the maintainer will merge the code.



#### As a Developer

As a developer it is important before you create any pull requests, against the main development branch you at the bare mininum run the following commands locally.

**Run *unit* tests**

```bash
yarn test:unit
```

**Run *e2e* tests**

```bash
yarn test:e2e
```

**Run *all* tests**

```bash
yarn test
```

**Make sure everything runs and functions correctly**

```bash
yarn dev
```

**Make sure you have no linting errors**

```bash
yarn lint
```


### Test

**Who Belongs in this branch?**

* QA Team

**Assumptions**

* [x] Unit Testing completed
* [x] Integration Testing Completed
* [x] Ready for QA Team
* [x] Integration testing Completed

### Pre-Prod (UAT)

### Prod

## Handling Hedge Cases

### Stage Defects found

**If a defect is found in UAT/Stage, it is recommended to perform all fixes in the TEST environment, and then to re-deploy the fixed version to PROD.**

### Test Defects found

**If Defects are found in TEST, do not send the build to STAGE or PROD, document the issues and send them back to DEV**

### Stage Defects found

***IF* a defect is found in UAT/Stage, it is recommended to perform all fixes in the TEST environment, and then to re-deploy the fixed version to PROD.**

### Production defects found

***IF* a bug is found in production, perform all fixes in the TEST/QA environment, and then to re-deploy the fixed version to STAGE. This ensures that any continuing development being performed in the DEV environment is not disrupted.**

### Test Environment Fales

**If a defect is found in production, it is recommended to perform all fixes in the TEST environment, and then to re-deploy the fixed version to PROD. This practice ensures that any continuing development being performed in the DEV environment is not disrupted.**

### FE Settings and Configurations

#### Scripts
Listed within are all of the available scripts and their purposes

#### Directory and Naming Conventions

#### Code Style Lint

#### Code Quality Tools

## Resources and Links

* [Nuxt](https://nuxtjs.org/)
* [Vue](https://cli.vuejs.org/)
* [Github Actions](https://github.com/features/actions)
* [Vue Testing Framework](https://vue-test-utils.vuejs.org/)
* [Feature Branch Workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow)

### Repositories and Release Branches

[Test Branch (TODO)](https://github.com)
[URL (TODO)](https://github.com)

[Stage Branch (TODO)](https://github.com)
[URL (TODO)](https://github.com)

[Production Branch (TODO)](https://github.com)
[URL (TODO)](https://github.com)

### Developer Resources

| Resource                   | Description                                  | URL                                     |
| -------------------------- | -------------------------------------------- | --------------------------------------- |
| IDE                        | Visual studio code                           |                                         |  |
| IDE Setup                  | Pre-configured vscode configuration download |                                         |
| Unit Testing               |                                              |                                         |
| Repo                       | Github Repository                            |                                         |
| Component Library          | Molekule Brand Component Library             |                                         |
| Node Version Manager (nvm) | Node version manager                         | [github](https://github.com/nvm-sh/nvm) |
| Yarn                       |                                              | [github](https://github.com)            |
| Jira| Primary Source of truth for planning and tasks  | [jira](https://jira.com) |


### Role Hierarchy and Permissions

- Maintainer
  - Name :
  - Contact :
  - Permissions :
- QA Lead :
  - Name :
  - Contact :
  - Permissions :
- PM :
  - Name :
  - Contact :
  - Permissions :



