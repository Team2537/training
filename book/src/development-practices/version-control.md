# Version Control
Our team uses GitHub. 

We have a Github Organization ([Team2537](https://github.com/Team2537)) where all of our robot code lives.

Each year, we create a new repository for the robot code. This repository is where all of the code for the robot lives. Here are some past years repositories:
- [2022 - Rapid React](https://github.com/Team2537/2022RapidReact)
- [2023 - Charged Up](https://github.com/Team2537/2023ChargedUp)
- [2024 - Crescendo](https://github.com/Team2537/2024Crescendo)

We use branches within these repositories to manage different versions of the robot code and to develop features in parallel.

## Special Branches
There are a few special branches that we use in our repositories:
- `main` - This is the main branch of the repository. This is where the code that is currently running on the robot lives.
- `dev` - This is the development branch. This is where we merge features and fix small bugs. When we are ready to deploy code to the robot, we merge the `dev` branch into the `main` branch. Code that is on the `dev` branch may not be fully tested, but should build and deploy successfully. 
- `[competition]` - Every competition that we participate in will have a branch. This branch will be created the night before we go to competition, and should include the contents of `main` (and maybe `dev`) at that time. Over the course of the competition, we may have to make emergency changes to the code. These changes will be made on the competition branch, and then possibly merged back into `dev` when we get back to the shop. The name of the competition branch should be the name of the competition, words separated by hyphens. For example, the branch for the CHS District Severn competition is `CHS-district-severn`.

## Feature Branches
Every new feature that we develop should be developed on a feature branch. This branch should be named after the feature that is being developed. For example, if we are developing a launcher subsystem, we might create a branch called `launcher-subsystem`. When the feature is complete, we will merge the feature branch into the `dev` branch through a **Pull Request.** You may not merge directly to `dev`. Each new feature branch should be created from the `dev` branch.

## Pull Requests
A pull request is a proposal to merge a set of changes from one branch into another. In a pull request, collaborators can review and discuss the proposed set of changes before they integrate the changes into the main codebase.

There are three types of Pull Requests that we use:
- **Feature Pull Request** - This is a pull request that proposes to merge a feature branch into the `dev` branch. This pull request should be reviewed by at least one other team member before it is merged.
- **Dev Pull Request** - This is a pull request that proposes to merge the `dev` branch into the `main` branch. This pull request should be reviewed by at least one other team member before it is merged. These pull requests should be merged by a Software Lead.

Please make sure your code follows the [code practices](development-practices/code.md) before creating a pull request. If you do not follow the code practices, your code may not be merged.

---
We also use other version control features:

## Commits
Each commit should be a small, atomic change. This means that each commit should only change one thing. This makes it easier to review changes and to understand what has been done. If you need to make multiple changes to implement a feature, you should make multiple commits.

### WIP Commits
If you are working on a feature and you are still writing code (maybe mid-line or mid-function) you shouldn't commit an incomplete change. Instead, you should commit a "Work In Progress" (WIP) commit. This commit should be marked as a WIP commit by starting the commit message with `WIP:`. This will let other team members know that the change is not complete. When you are finished with the feature, you can [squash](https://www.git-tower.com/learn/git/faq/git-squash) the WIP commits into a single commit. 

## Tags
We don't use tags very often, but we may use them to mark important points in the codebase. For example, we might tag the autonomous code that we first get working on the robot with a tag like `auto-working`.

## Issues
We use GitHub Issues to track absolutely everything. New feature? New issue. Bug? Issue. Need Documentation? Need Issue. Anyone can and should create issues on our robot's repository. For most Pull Requests, there will be an issue that the PR is addressing.

## Projects!
We use GitHub Projects to track the progress of our robot code. Each project has columns for different stages of development. For example, we might have columns for "To Do", "In Progress", "Review", and "Done". Each issue should be assigned to a column.

Everyone should be able to see the progress of the robot code by looking at the project board.

The Software Lead(s) will keep the project board up to date.