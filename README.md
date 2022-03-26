# gha-test
## Problem
Running a sufficient number of hypothesis test iterations (denoted by `brownie` as `max_examples`) to ensure code correctness takes too long when developing on working branches.

The reasonable fix would be to have two separate `brownie-config.yaml` files, one for production (`main`) and runs the default number of iterations (50), and one for working branches that runs a very small number of iterations (3 is chosen).
Because `brownie` does not allow the user to specify the configuration file, but instead requires the configuration file to be in the root directory and to be named `brownie-config.yaml`, we unfortunately cannot simply create two separate configuration files. 
`brownie` does however allow you to pass in environment variables into the configuration file.
Therefore, using environment variables to set the number of iterations will be used as a work around.

## Goal 
Reduce the time it takes to run the GitHub Actions tests on WIP branches.
In doing this, introduces a new developer flow that is enforced by the GitHub Actions.

### Developer Flow
Previously, PRs were opened and merged directly to the `main` branch.
The new developer flow uses a `dev` branch instead.
Now, PRs will be opened and merged directly to `dev`, NOT `main`.
Once the PR to `dev` is finalized and all GitHub Actions have passed, THEN a new PR is opened on the `main` branch from the `dev` branch.
The `main` branch will ONLY accept PRs (and therefore merges) from the `dev` branch (this is enforced by logic included in the GitHub Action file).

How to do git comments in merge commits:
1. When merging from a working branch to `dev`, leave all git comments in the commit.
1. When merging from `dev` to `main`, only include one single line of the git comment that refers to the `dev` branch.

### GitHub Actions Updates
The following changes are included in the GitHub Actions.
The first change listed below enforces the new developer flow.
The second and third changes listed below are directly related to reducing the time a developer has to wait for tests to run on their working branches.

1. Force GitHub Action fail if a developer is pushing any branch that is NOT `dev` to `main`.
2. Make a `main` branch that GitHub Actions will run the brownie tests with the number of hypothesis iterations (`max_examples`) equal to 50 (the default).
3. Make a `dev` branch that GitHub Actions will run the brownie tests with the number of hypothesis iterations  (`max_examples`) equal to 3.

### Running Locally
1. In `brownie-config.yaml` we define `max_examples` and the `MAX_EXAMPLES` environment variable.
1. In the local `.env` file, the following MUST be added:
   ```
   export MAX_EXAMPLES=<number of desired hypothesis iterations>
   ```

### Notes
Ensure that branch protection is turned on for both `main` and `dev`.
Go to Settings -> Branches -> Add Rule, then for both `main` and `dev` select:
- Select: `Require a pull request before merging`
- Select: `Require status checks to pass before merging`
  - Select: `Require branches to be up to date before merging`
  - In the search bar, type the name of the job of the GitHub Action that should stop a merge if it fails (`test` for our Action) and add it so it is present under `Status checks that are required`.

