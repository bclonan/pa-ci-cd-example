
# Aurora FE CI/CD v1

Within is a proposed CI/CD Setup and several code standards to follow, so that a project remains both maintainable and scalable.

## Table of Contents

## Git Workflow

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

| Tool             | Description                       |
| :--------------- | :-------------------------------- |
| `Github Actions` | **Required**. Id of item to fetch |
| `CircleCI`       |                                   |
| `StoryBlock`     |                                   |

## Environments

| Environment | Description                                                                                                                | URL |
| ----------- | -------------------------------------------------------------------------------------------------------------------------- | --- |
| DEV         | The development environment is a workspace for developers to make changes without breaking anything in a live environment. | N/A |
| TEST        | The development environment is a workspace for developers to make changes without breaking anything in a live environment. | N/A |
| STAGE       | allow the application's main users to test new features before they are pushed into the production environment             | N/A |
| PROD        | The live environment made available to all users                                                                           | N/A |

### Dev

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

### Quick Links

[Working Dev Branch (TODO)](https://github.com)
[Development URL (TODO)](https://github.com)

### Automations

* [] TODO


### Environment Variables

To work in dev you will need the following environment variables

| Variable | Description                                                                                                                | KEY |
| ----------- | -------------------------------------------------------------------------------------------------------------------------- | --- |
| `Telium` |  | N/A |
| `Stripe Test Key`        | N/A | N/A |
| `Google API Key`        | N/A | N/A |


### Resources

* [] TODO

| Resource | Description | URL |
| ------| -----| ---- |
| IDE | Visual studio code | | |
| IDE Setup | Pre-configured vscode configuration download | |  
| Unit Testing | | |
| Repo | Github Repository | |
| Component Library | Molekule Brand Component Library | |

### Test

**Who Belongs in this branch?**

* QA Team

**Assumptions**

* [x] Unit Testing completed
* [x] Integration Testing Completed
* [x] Ready for QA Team
* [x] Integration testing Completed

### Pre-Prod (UAT)

```http
  GET /api/items/${id}
```

| Parameter | Type     | Description                       |
| :-------- | :------- | :-------------------------------- |
| `id`      | `string` | **Required**. Id of item to fetch |

### Prod

```http
  GET /api/items/${id}
```

| Parameter | Type     | Description                       |
| :-------- | :------- | :-------------------------------- |
| `id`      | `string` | **Required**. Id of item to fetch |

## Authors

* [@katherinepeterson](https://www.github.com/katherinepeterson)

## Badges

Add badges from somewhere like: [shields.io](https://shields.io/)

[![MIT License](https://img.shields.io/apm/l/atomic-design-ui.svg?)](https://github.com/tterb/atomic-design-ui/blob/master/LICENSEs)
[![GPLv3 License](https://img.shields.io/badge/License-GPL%20v3-yellow.svg)](https://opensource.org/licenses/)
[![AGPL License](https://img.shields.io/badge/license-AGPL-blue.svg)](http://www.gnu.org/licenses/agpl-3.0)

## Contributing

Contributions are always welcome!

See `contributing.md` for ways to get started.

Please adhere to this project's `code of conduct`.

## Demo

Insert gif or link to demo

## Deployment

To deploy this project run

```bash
  npm run deploy
```

## Documentation

[Documentation](https://linktodocumentation)


## Roadmap

## Resources

* [Nuxt](https://nuxtjs.org/)
* [Vue](https://cli.vuejs.org/)
* [Github Actions](https://github.com/features/actions)
* [Vue Testing Framework](https://vue-test-utils.vuejs.org/)
* [Feature Branch Workflow](https://www.atlassian.com/git/tutorials/comparing-workflows/feature-branch-workflow)
