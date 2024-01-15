## 1 - Implement Continuous Integration (CI)

### 1.1 - Set the AWS Secret Keys

1. TODO: Add a link to the AWS Secret Keys

### 1.2 - Use a starter workflow

To build a workflow that employs Actions for your Continuous Integration process, start by adding a **starter workflow** to your repository:

1. From your repository's main view, find and navigate to the **Actions** tab.
2. Select **New workflow**.
3. Search for `Node.js`.
4. Click **Configure** under the `Node.js` starter workflow.
5. In the `node-version` field within the YAML configuration, remove `14.x` (since our app isn't compatible with this version).

To finish setting up your initial CI workflow, commit the `node.js.yml` file to the `main` branch.

<details>
<summary>Your `.github/workflows/node.js.yml` should contain the following:</summary>

```yml
name: Node.js CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build:
    runs-on: ubuntu-latest

    strategy:
      matrix:
        node-version: [16.x, 18.x]
        # See supported Node.js release schedule at https://nodejs.org/en/about/releases/

    steps:
      - uses: actions/checkout@v3
      - name: Use Node.js ${{ matrix.node-version }}
        uses: actions/setup-node@v3
        with:
          node-version: ${{ matrix.node-version }}
          cache: 'npm'
      - run: npm ci
      - run: npm run build --if-present
      - run: npm test
```

</details>

### 1.3 - Understanding references to actions

As you can see, we're now employing a second action in our workflow, `actions/setup-node`, which is used to install a specific Node.js version on the runner.

Let's dissect the reference to that action to understand its structure:

- `actions/` references the owner of the action, which is translated into a user or organization on GitHub.
- `setup-node` refers to the name of the action, which corresponds to a repository on GitHub.
- `@v3` represents the version of the action, which corresponds to a Git tag or a general reference (such as a branch or even a commit SHA) on the repository.

This reference structure makes it straightforward to navigate to the source code of any action by merely appending the `owner` and `name` to the `github.com` URL, like so: `https://github.com/{owner}/{name}`. For the above example, this would be <https://github.com/actions/setup-node>.

### 1.4 - Understanding matrix builds

Observe that our workflow employs a [matrix build strategy](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs) with two Node.js versions: 16 and 18. A matrix build enables you to execute a job in parallel using various input parameters. In our case, we're running the same job twice, but with distinct Node.js versions.

### Checking workflow runs

Your newly implemented CI workflow now runs with every push. Given that you just pushed a new commit containing the workflow you've created, you should already have a workflow run in progress.

![Actions overview showing the Node.js workflow running](./images/running-nodejs-workflow.png)

Keep in mind that we will need to run tests as part of our CI workflow. You can find most of this application's tests in the [`src/pages/Home.test.tsx`](../src/pages/Home.test.tsx) file, which partly looks like this:

```typescript
// ... imports

describe("<Home />", (): void => {
  afterEach((): void => {
    cleanup();
  });

  it("renders the octocats returned from the API.", async (): Promise<void> => {
    const inMemoryAPI = createInMemoryOctocatApi();
    inMemoryAPI.addOctocats([
      createTestOctocat({ id: "#1", name: "Octocat 1" }),
      createTestOctocat({ id: "#2", name: "Octocat 2" }),
    ]);

    renderWithProviders({ component: <Home />, inMemoryApi: inMemoryAPI });

    expect(await screen.findByText("Octocat 1")).toBeDefined();
    expect(screen.getByText("Octocat 2")).toBeDefined();
  });

  // ... more tests

});

```

The result of your last push to the main branch should resemble the following:

![Actions overview showing a successful workflow run](./images/success-nodejs-workflow.png)

