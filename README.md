
# 1. Aurora FE CI/CD v1

Within is a proposed CI/CD Setup and several code standards to follow, so that a project remains both maintainable and scalable.

## 1.1. Table of Contents

[1. Aurora FE CI/CD v1](#1-aurora-fe-cicd-v1)

- [1. Aurora FE CI/CD v1](#1-aurora-fe-cicd-v1)
  - [1.1. Table of Contents](#11-table-of-contents)
  - [1.2. Terminology](#12-terminology)
  - [1.3. Git](#13-git)
    - [1.3.1. Git Rules](#131-git-rules)
    - [1.3.2. Git workflow](#132-git-workflow)
    - [1.3.3. Writing good commit messages](#133-writing-good-commit-messages)
  - [1.4. Tech Stack](#14-tech-stack)
  - [1.5. Environments](#15-environments)
  - [1.6. Github Actions [automation]](#16-github-actions-automation)
    - [1.6.1. GA Terms](#161-ga-terms)
    - [1.6.2. Creating and Maintaining Secrets](#162-creating-and-maintaining-secrets)
      - [1.6.2.1. Managing Secrets](#1621-managing-secrets)
    - [1.6.3. Workflows](#163-workflows)
      - [1.6.3.1. Project Aurora Github Action table](#1631-project-aurora-github-action-table)
      - [1.6.3.2. branch-overwatch.yml](#1632-branch-overwatchyml)
      - [1.6.3.3. dev-pull.yml](#1633-dev-pullyml)
      - [1.6.3.4. dev-push.yml](#1634-dev-pushyml)
      - [1.6.3.5. stage-push.yml](#1635-stage-pushyml)
      - [1.6.3.6. prod-push.yml](#1636-prod-pushyml)
    - [1.6.4. Getting Started](#164-getting-started)
      - [1.6.4.1. Developer Checklist](#1641-developer-checklist)
      - [1.6.4.2. Sys Admin Checklist](#1642-sys-admin-checklist)
      - [1.6.4.3. QA Checklist](#1643-qa-checklist)
    - [1.6.5. Development Branch](#165-development-branch)
      - [1.6.5.1. Overview](#1651-overview)
      - [1.6.5.2. New Feature Dev steps](#1652-new-feature-dev-steps)
      - [1.6.5.3. As a Developer](#1653-as-a-developer)
    - [1.6.6. Test](#166-test)
    - [1.6.7. Pre-Prod (UAT)](#167-pre-prod-uat)
    - [1.6.8. Prod](#168-prod)
  - [1.7. Handling Hedge Cases](#17-handling-hedge-cases)
    - [1.7.1. Stage Defects found](#171-stage-defects-found)
    - [1.7.2. Test Defects found](#172-test-defects-found)
    - [1.7.3. Stage Defects found](#173-stage-defects-found)
    - [1.7.4. Production defects found](#174-production-defects-found)
    - [1.7.5. Test Environment Fales](#175-test-environment-fales)
    - [1.7.6. FE Settings and Configurations](#176-fe-settings-and-configurations)
      - [1.7.6.1. Scripts](#1761-scripts)
      - [1.7.6.2. Directory and Naming Conventions](#1762-directory-and-naming-conventions)
      - [1.7.6.3. Code Style Lint](#1763-code-style-lint)
      - [1.7.6.4. Code Quality Tools](#1764-code-quality-tools)
  - [1.8. Resources and Links](#18-resources-and-links)
    - [1.8.1. Repositories and Release Branches](#181-repositories-and-release-branches)
    - [1.8.2. Developer Resources](#182-developer-resources)
    - [1.8.3. Role Hierarchy and Permissions](#183-role-hierarchy-and-permissions)

## 1.2. Terminology

| Term                            | Meaning                                                                                                                                                                                                                 |
| ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Continious Integration (**CI**) | Continuous Integration (CI) automates the process of testing and building your code before merging it. In practice, developers should commit or integrate their changes to the main shared repo once-per-day (or more). |
| Continuous Deployment (**CD**)  | Continuous Deployment (CD) automatically releases new production code to users. It is the step that happens after new code has been integrated.                                                                         |

## 1.3. Git

### 1.3.1. Git Rules

There are a set of rules to keep in mind:

- Perform work in a feature branch.

    _Why:_
    >Because this way all work is done in isolation on a dedicated branch rather than the main branch. It allows you to submit multiple pull requests without confusion. You can iterate without polluting the master branch with potentially unstable, unfinished code. [read more...](https://www.atlassian.com/git/tutorials/comparing-workflows#feature-branch-workflow)
- Branch out from `develop`

    _Why:_
    >This way, you can make sure that code in master will almost always build without problems, and can be mostly used directly for releases (this might be overkill for some projects).

- Never push into `develop` or `master` branch. Make a Pull Request.

    _Why:_
    > It notifies team members that they have completed a feature. It also enables easy peer-review of the code and dedicates forum for discussing the proposed feature.

- Update your local `develop` branch and do an interactive rebase before pushing your feature and making a Pull Request.

    _Why:_
    > Rebasing will merge in the requested branch (`master` or `develop`) and apply the commits that you have made locally to the top of the history without creating a merge commit (assuming there were no conflicts). Resulting in a nice and clean history. [read more ...](https://www.atlassian.com/git/tutorials/merging-vs-rebasing)

- Resolve potential conflicts while rebasing and before making a Pull Request.
- Delete local and remote feature branches after merging.

    _Why:_
    > It will clutter up your list of branches with dead branches. It ensures you only ever merge the branch back into (`master` or `develop`) once. Feature branches should only exist while the work is still in progress.

- Before making a Pull Request, make sure your feature branch builds successfully and passes all tests (including code style checks).

    _Why:_
    > You are about to add your code to a stable branch. If your feature-branch tests fail, there is a high chance that your destination branch build will fail too. Additionally, you need to apply code style check before making a Pull Request. It aids readability and reduces the chance of formatting fixes being mingled in with actual changes.

- Use [this](./.gitignore) `.gitignore` file.

    _Why:_
    > It already has a list of system files that should not be sent with your code into a remote repository. In addition, it excludes setting folders and files for most used editors, as well as most common dependency folders.

- Protect your `develop` and `master` branch.
  
    _Why:_
    > It protects your production-ready branches from receiving unexpected and irreversible changes. read more... [Github](https://help.github.com/articles/about-protected-branches/), [Bitbucket](https://confluence.atlassian.com/bitbucketserver/using-branch-permissions-776639807.html) and [GitLab](https://docs.gitlab.com/ee/user/project/protected_branches.html)

<a name="git-workflow"></a>

### 1.3.2. Git workflow

Because of most of the reasons above, we use [Feature-branch-workflow](https://www.atlassian.com/git/tutorials/comparing-workflows#feature-branch-workflow) with [Interactive Rebasing](https://www.atlassian.com/git/tutorials/merging-vs-rebasing#the-golden-rule-of-rebasing) and some elements of [Gitflow](https://www.atlassian.com/git/tutorials/comparing-workflows#gitflow-workflow) (naming and having a develop branch). The main steps are as follows:

- For a new project, initialize a git repository in the project directory. __For subsequent features/changes this step should be ignored__.

   ```sh
   cd <project directory>
   git init
   ```

- Checkout a new feature/bug-fix branch.

    ```sh
    git checkout -b <branchname>
    ```

- Make Changes.

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

- Sync with remote to get changes you’ve missed.

    ```sh
    git checkout develop
    git pull
    ```

    _Why:_
    > This will give you a chance to deal with conflicts on your machine while rebasing (later) rather than creating a Pull Request that contains conflicts.

- Update your feature branch with latest changes from develop by interactive rebase.

    ```sh
    git checkout <branchname>
    git rebase -i --autosquash develop
    ```

    _Why:_
    > You can use --autosquash to squash all your commits to a single commit. Nobody wants many commits for a single feature in develop branch. [read more...](https://robots.thoughtbot.com/autosquashing-git-commits)

- If you don’t have conflicts, skip this step. If you have conflicts, [resolve them](https://help.github.com/articles/resolving-a-merge-conflict-using-the-command-line/)  and continue rebase.

    ```sh
    git add <file1> <file2> ...
    git rebase --continue
    ```

- Push your branch. Rebase will change history, so you'll have to use `-f` to force changes into the remote branch. If someone else is working on your branch, use the less destructive `--force-with-lease`.

    ```sh
    git push -f
    ```

    _Why:_
    > When you do a rebase, you are changing the history on your feature branch. As a result, Git will reject normal `git push`. Instead, you'll need to use the -f or --force flag. [read more...](https://developer.atlassian.com/blog/2015/04/force-with-lease/)

- Make a Pull Request.
- Pull request will be accepted, merged and close by a reviewer.
- Remove your local feature branch if you're done.

  ```sh
  git branch -d <branchname>
  ```

  to remove all branches which are no longer on remote

  ```sh
  git fetch -p && for branch in `git branch -vv --no-color | grep ': gone]' | awk '{print $1}'`; do git branch -D $branch; done
  ```

### 1.3.3. Writing good commit messages

Having a good guideline for creating commits and sticking to it makes working with Git and collaborating with others a lot easier. Here are some rules of thumb ([source](https://chris.beams.io/posts/git-commit/#seven-rules)):

- Separate the subject from the body with a newline between the two.

    _Why:_
    > Git is smart enough to distinguish the first line of your commit message as your summary. In fact, if you try git shortlog, instead of git log, you will see a long list of commit messages, consisting of the id of the commit, and the summary only.

- Limit the subject line to 50 characters and Wrap the body at 72 characters.

    _why_
    > Commits should be as fine-grained and focused as possible, it is not the place to be verbose. [read more...](https://medium.com/@preslavrachev/what-s-with-the-50-72-rule-8a906f61f09c)

- Capitalize the subject line.
- Do not end the subject line with a period.
- Use [imperative mood](https://en.wikipedia.org/wiki/Imperative_mood) in the subject line.

    _Why:_
    > Rather than writing messages that say what a committer has done. It's better to consider these messages as the instructions for what is going to be done after the commit is applied on the repository. [read more...](https://news.ycombinator.com/item?id=2079612)

- Use the body to explain **what** and **why** as opposed to **how**.

## 1.4. Tech Stack

**Client:** Nuxt, Vue, TailwindCSS

**Tools:** Github Actions, Docker, circleci

| Tool             | Description                                              |
| :--------------- | :------------------------------------------------------- |
| `Github Actions` | **Required**. Id of item to fetch                        |
| `StoryBlock`     | Used to show catelog and display global brand components |

## 1.5. Environments

| Environment | Description                                                                                                                | URL |
| ----------- | -------------------------------------------------------------------------------------------------------------------------- | --- |
| DEV         | The development environment is a workspace for developers to make changes without breaking anything in a live environment. | N/A |
| TEST        | The development environment is a workspace for developers to make changes without breaking anything in a live environment. | N/A |
| STAGE       | allow the application's main users to test new features before they are pushed into the production environment             | N/A |
| PROD        | The live environment made available to all users                                                                           | N/A |

## 1.6. Github Actions [automation]

### 1.6.1. GA Terms

| Term          | Purpose                                                                                                                                                                                                                                                                                                                                                                         |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Runners**   | These are hosted virtual operating systems that could run commands to carry out a build process.                                                                                                                                                                                                                                                                                |
| **Workflows** | These are laid out instructions that give the Github Action application on a runner. Workflow instructions are defined in a YAML file and live inside of the .github/workflows folder in a git repository. The name of a workflow file does not have a correlation with the intention of the file, as Github parses and runs every file inside of the .github/workflows folder. |
| **Events**    | For a Workflow file to be processed and executed, an event is required.                                                                                                                                                                                                                                                                                                         |
| **Jobs**      | Async or Sync series of steps carried out within any given workflow.                                                                                                                                                                                                                                                                                                            |
| **Steps**     | These are the single elements that make up a job. A step groups the actions that are carried out on a runner.                                                                                                                                                                                                                                                                   |
| **Actions**   | An action is an element of a step and is basically a command.                                                                                                                                                                                                                                                                                                                   |

### 1.6.2. Creating and Maintaining Secrets

Throughout our CI/CD System there are a number of secrets which must be created and maintained so that they can be used in our github actions.

**Important : Keep secrets, managed by implimenting privilages on them**
[Implimenting Lease Privilage for secrets in github actions](https://github.blog/2021-04-13-implementing-least-privilege-for-secrets-in-github-actions/)

#### 1.6.2.1. Managing Secrets

1. Go to the github repositiory you wish to work on.
2. Go to the `Settings` Tab.
3. Select `Secrets`
4. Name your secret
   1. **I.E** 'MY_SECRET_KEY'
5. Add the secret value below the name.
6. Press save.

**NOTE : By default, secrets are not made available to workflows and jobs, to have access to secrets, you have to request them at the steps that they are required.**
[Managing Secrets](https://docs.github.com/en/actions/learn-github-actions/managing-complex-workflows#storing-secrets)

### 1.6.3. Workflows

#### 1.6.3.1. Project Aurora Github Action table

| Workflow                       | Purpose                                                                                                                                                                                                                                                                                                    | Branch     | Initiate With | Fail                                                                                   | Pass                                                                 |
| ------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------- | ------------- | -------------------------------------------------------------------------------------- | -------------------------------------------------------------------- |
| branch-overwatch.yml(optional) | A new branch is created within the repository                                                                                                                                                                                                                                                              | ${x}       | created       | N/A                                                                                    | Notify stakeholders that work has begun through a slack message etc. |
| dev-pull.yml                   | Tests pull requests into the TEST branch against developers feature pr                                                                                                                                                                                                                                     | TEST       | pull_request  | denys the pr                                                                           | marks the pr as succesfull                                           |
| dev-push.yml                   | When the maintainer of the TEST branch approves pull requests into TEST this action will kick off a build and deploy it to the TEST server                                                                                                                                                                 | push       | push          | doesnt deploy the built code to dev server, and requires failed conditions to be fixed | Deploys built code to dev server                                     |
| stage-push.yml                 | After the TEST server has been verified by the QA Team the repo maintainer can initiate a push request into STAGE (or Prod if no UAT testing required) this workflow will perform tests, and also run utility scripts such as compressing images, finally it will produce a new build and deploy it to AWS | STAGE      | push          | Create issues, dissalow from code deployment                                           | Deploys code to STAGE server                                         |
| prod-push.yml                  | After the TEST server has been verified by the QA Team the repo maintainer can initiate a push request into STAGE (or Prod if no UAT testing required) this workflow will perform tests, and also run utility scripts such as compressing images, finally it will produce a new build and deploy it to AWS | PRODUCTION | push          | Create issues, dissalow from code deployment                                           | Deploys code to STAGE server                                         |

#### 1.6.3.2. branch-overwatch.yml

**Features**

**Flow**

**code**

```
todo
```

#### 1.6.3.3. dev-pull.yml

**Features**

**Flow**

**code**

```
todo
```

#### 1.6.3.4. dev-push.yml

**Features**

**Flow**

**code**

```
todo
```

#### 1.6.3.5. stage-push.yml

**Features**

**Flow**

**code**

```
todo
```

#### 1.6.3.6. prod-push.yml

**Features**

**Flow**

**code**

```
todo
```

### 1.6.4. Getting Started

Before entering the development cycle, or contributing to the project in any way ensure that you are fully aware of both your role, and that you have the tools for the job.

#### 1.6.4.1. Developer Checklist

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

#### 1.6.4.2. Sys Admin Checklist

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

#### 1.6.4.3. QA Checklist

[x] Ensure you have Accessibility testing tools (Axe)
[x] Ensure you understand the *feature* scope
[x] Ensure you have a basic understanding of git and git workflows

### 1.6.5. Development Branch

**Who Belongs in this branch?**

- Development Team

**Assumptions**

- [x] Everything Goes
- [x] A developer has a fully, scoped 'feature' or task to complete.
- [x] **Code Ownership** : Individual developers own their code in this branch, they are required to write all tests before marking their task is complete.
- [x] **Commenting** : Don't use comments as an excuse for a bad code. Keep your code clean. Additionally, don't use clean code as an excuse not to comment.

**Path to Test**

To make it to stage a piece of code must meet the following guidlines.

- [x] Unit Tests Written
- [x] Basic Integration Testion Complete
- [x] Code Style and formatting compatible with project guidlines
- [x] Remove All Console Logging
- [x] Ensure No Secrets have been included

#### 1.6.5.1. Overview

1. Developers are assigned tasks
2. Developers Create a feature branch for their task
3. Developers Complete their task, and write tests for their new code.
4. Developers Open A pull request into the main development working branch
5. Github Action  

#### 1.6.5.2. New Feature Dev steps

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

#### 1.6.5.3. As a Developer

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

### 1.6.6. Test

**Who Belongs in this branch?**

- QA Team

**Assumptions**

- [x] Unit Testing completed
- [x] Integration Testing Completed
- [x] Ready for QA Team
- [x] Integration testing Completed

### 1.6.7. Pre-Prod (UAT)

### 1.6.8. Prod

## 1.7. Handling Hedge Cases

### 1.7.1. Stage Defects found

**If a defect is found in UAT/Stage, it is recommended to perform all fixes in the TEST environment, and then to re-deploy the fixed version to PROD.**

### 1.7.2. Test Defects found

**If Defects are found in TEST, do not send the build to STAGE or PROD, document the issues and send them back to DEV**

### 1.7.3. Stage Defects found

***IF* a defect is found in UAT/Stage, it is recommended to perform all fixes in the TEST environment, and then to re-deploy the fixed version to PROD.**

### 1.7.4. Production defects found

***IF* a bug is found in production, perform all fixes in the TEST/QA environment, and then to re-deploy the fixed version to STAGE. This ensures that any continuing development being performed in the DEV environment is not disrupted.**

### 1.7.5. Test Environment Fales

**If a defect is found in production, it is recommended to perform all fixes in the TEST environment, and then to re-deploy the fixed version to PROD. This practice ensures that any continuing development being performed in the DEV environment is not disrupted.**

### 1.7.6. FE Settings and Configurations

#### 1.7.6.1. Scripts

Listed within are all of the available scripts and their purposes

#### 1.7.6.2. Directory and Naming Conventions

#### 1.7.6.3. Code Style Lint

#### 1.7.6.4. Code Quality Tools

## 1.8. Resources and Links

- [Nuxt](https://nuxtjs.org/)
- [Vue](https://cli.vuejs.org/)
- [Github Actions](https://github.com/features/actions)
- [Vue Testing Framework](https://vue-test-utils.vuejs.org/)
- [Feature Branch Workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow)

### 1.8.1. Repositories and Release Branches

[Test Branch (TODO)](https://github.com)
[URL (TODO)](https://github.com)

[Stage Branch (TODO)](https://github.com)
[URL (TODO)](https://github.com)

[Production Branch (TODO)](https://github.com)
[URL (TODO)](https://github.com)

### 1.8.2. Developer Resources

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

### 1.8.3. Role Hierarchy and Permissions

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
