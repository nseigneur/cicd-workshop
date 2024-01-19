# Quality Assurance and Tests

## 1 - Add code coverage to your workflow

When setting up CI for your project, it's common to provide additional information to users, such as code coverage stats for the project's tests.

Doing that is straightforward with GitHub Actions. You determine where and when a specific task should occur, and then search for an appropriate action in the [GitHub Marketplace](https://github.com/marketplace?category=&query=&type=actions&verification=).

### 1.1 - Find an action in the marketplace

1. Search for an Action in the GitHub Marketplace:  `vitest coverage report`
  ![Search Result for "Vitest Test Coverage" in the GitHub Marketplace](./images/marketplace-vitest-search-result.png)

2. Click on the **Vitest Coverage Report** action.

3. Read the provided documentation and incorporate the action into your workflow.

### 1.2 - Permissions in a workflow

This is a good moment to discuss **permissions** within a workflow. Any workflow interacting with GitHub resources requires permissions to do so. By managing permissions, GitHub users can ensure that only authorized users or processes can carry out specific actions, like calling an API with a private access key, executing certain automations, or deploying to production environments. This prevents unauthorized access to sensitive data, reduces the risk of unintentional or malicious changes, and helps to uphold the overall security and stability of the codebase. For example:

1. The `actions/checkout` action requires read permissions for your repository to execute the checkout on the runner machine.
2. The **Vitest Coverage Report** action needs to write a comment to a pull request, and thus needs permissions to do so.

By default, GitHub Actions workflows are executed with a restricted [set of default permissions](https://docs.github.com/en/actions/security-guides/automatic-token-authentication#permissions-for-the-github_token), which can be extended as needed with the `permissions` keyword. This can be applied:

- At the root of the workflow to set this permission for **all** jobs within the workflow.
- Within a [job](https://docs.github.com/en/actions/using-jobs) definition itself to specify permissions for that job only. This approach is recommended from a security perspective, as it provides the least required privileges to your workflows and jobs.

These permissions are applied to the `GITHUB_TOKEN`, which we will explore in more detail later.

For now, what you need to know is: as soon as you specify the `permissions` keyword, the default permissions no longer apply. This means you must explicitly configure all necessary permissions in the job or workflow. Let's do this in the next step.

### 1.3 - Update the worklow

1. In the `main` branch, edit the CI workflow `.github/workflows/node.js.yml`

2. Add the `permissions` keyword with the following permissions into the job section:

    ```yml
    build:
      runs-on: ubuntu-latest
      permissions:
        # Required to allow actions/checkout to clone the repository onto the runner
        contents: read
        # Required to allow the vitest coverage action to write a comment into the pull request
        pull-requests: write
      # ... rest of the node.js.yml
    ```

3. Add the following step in the `build` job section of your workflow, right after the `npm test` step:

    ```yml
        # ... rest of the node.js.yml
        - run: npm run test
        - uses: davelosert/vitest-coverage-report-action@v1
          with:
            vite-config-path: vite.config.ts
    ```

4. While you're at it, how about giving the job a better `name`?

    ```yml
      jobs:
        build:
          name: "Build and Test"
          runs-on: ubuntu-latest
      # ... rest of the node.js.yml
    ```

5. Commit the `node.js.yml` file.

### 1.4 - Remove the matrix build strategy (If you have implemented the extend goal)

As this is a frontend project, we don't need a matrix build strategy (which is more suited for backend projects that might be running on several Node.js versions). Removing the matrix build will also make the tests run only once.

<details>
<summary>Try to remove the matrix build yourself and make the `actions/setup-node` action only run on version 16.x. Expand this section to see the solution.</summary>

```yml
jobs:
  build:
    name: "Build and Test"
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: actions/checkout@v3
      - name: Use Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 20.x
          cache: 'npm'
      - run: npm ci
      - run: npm run build
      - run: npm run test
      - name: 'Report Coverage'
        uses:  davelosert/vitest-coverage-report-action@v2
```

</details>

### 1.5 - Create a new pull request

1. Go to the main page of the repository.

2. Click on [`./src/main.tsx`](../src/main.tsx), and edit the file (for instance, add a comment).

3. Scroll down and click **Create a new branch for this commit and start a pull request**.

4. Click **Propose changes**.

5. Click **Create pull request**.

6. Wait for the CI workflow to run, and you will see a new comment in your pull request with the code coverage.
![PR Comment with a coverage report from vitest](./images/vitest-coverage-report.png)

### 1.6. (Optional) - Enforce a certain coverage threshold with Branch Protection rules

As you can see, the test coverage of this project is quite low. Sometimes, we want to enforce a certain level of coverage on a project. This means that we would not allow merging a PR if it reduces the coverage below a certain threshold.

Let's try that out in this project:

1. On the branch you created earlier, navigate to the [`vite.config.ts`](../vite.config.ts) file (located at the root level of the repository). Within the `test.coverage` section, edit it to establish some thresholds like this:

    ```typescript
    coverage: {
      reporter: ["text", "json", "json-summary"],
      lines: 100,
      branches: 100,
      functions: 100,
      statements: 100
    },
    ```

2. With the coverage thresholds set, our workflow will now fail after the next commit on the `npm test` step. However, since we still want to report the coverage, we need to run the `vitest-coverage-report-action` even if the previous step fails. We can do this by adding an `if: always()` statement to the step:

      ```yml
      - name: 'Report Coverage'
        uses:  davelosert/vitest-coverage-report-action@v2
        if: always()
      ```

3. Commit the changes and wait for the workflow to run.

The `coverage` step should now fail. However, this does not yet prevent you from merging this PR. The merge button is still clickable:

![GitHub checks with a failed action-workflow, but the merge button is still active](./images/merge-possible-with-failed-checks.png)

To make this work, we need to set our target branch `main` as a protected branch and enforce that the `build` workflow be successful before a merge can be executed:

1. Within your repository, go to **Settings** and then to **Branches**.

2. Under **Branch protection rules**, click on **Add branch protection rule**.

3. For the **Branch name pattern**, type `main`.

4. Check the **Require status checks to pass before merging** box.

5. In the search box that will appear, look for `Build and Test` (or whatever name you chose for the job in step 3.3) and select that job. *(Note that you might also see the jobs of the previous matrix builds with specific Node versions. You can ignore these.)*
    ![Settings page with set up branch protection rule for main branch](./images/setting-up-branch-protection-rules.png)

6. Scroll down and click `Create`.

If you now return to the PR, you will see that the merge button is inactive and can't be clicked anymore.

![GitHub checks with a failed action-workflow and merge button is inactive](./images/merge-prevented-with-failed-checks.png)

As an administrator, you still have the option to force a merge. Regular users in your repo won't have this privilege.

> **Note**
> This will now prevent people from merging a branch to `main` not only if the coverage thresholds are not met, but also if the entire workflow fails for other reasons. For example, if the build isn't working anymore or if the tests are generally failing - which usually is a desired outcome.

From here, you have two options:

1. Write more tests (if you're into React 😉)
2. Remove the (admittedly stringent) thresholds or lower them to make the workflow pass

## Conclusion

In this lab, you have learned how to:

- 👍 Add a new action to your workflow.
- 👍 (optionally) Prevent merges on failing tests or coverage thresholds using Branch Protection rules.

---