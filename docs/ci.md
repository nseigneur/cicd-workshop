## 1 - Implement Continuous Integration (CI)

### 1.1 - Use a starter workflow

To build a workflow that employs Actions for your Continuous Integration process, start by adding a **starter workflow** to your repository:

1. From your repository's main view, find and navigate to the **Actions** tab.
2. Select **New workflow**.
3. Search for `push`.
4. Click **Configure** under the `push` starter workflow.

To finish setting up your initial CI workflow, commit the `push.yml` file to the `main` branch.

<details>
<summary>Your `.github/workflows/push.yml` should contain the following:</summary>

```yml
name: Build and Push

on:
  push:
    branches: [ main ]
  workflow_dispatch:

jobs:
  build:
    name: Build and Test
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
    - uses: actions/checkout@v3
    - name: Use Node.js 20.x
      uses: actions/setup-node@v3
      with:
        node-version: 20.x
        cache: npm
    - run: npm ci
    - run: npm run build --if-present
    - run: npm test
    - name: Report Coverage
      uses: davelosert/vitest-coverage-report-action@v2
      if: always()

  package-and-publish:
    needs:
      - build

    name: 🐳 Package & Publish
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write

    steps:
      - uses: actions/checkout@v3
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v2

      - name: Sign in to GitHub Container Registry
        uses: docker/login-action@v2
        with:
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
          registry: ghcr.io
      - name: Generate Docker Metadatadoc
        id: meta
        uses: docker/metadata-action@v4
        with:
          images: ghcr.io/${{ github.repository }}
          tags: |
            type=ref,event=tag
            type=ref,event=pr
            type=sha,event=branch,prefix=,suffix=,format=short
      - name: Build and Push Docker Image
        uses: docker/build-push-action@v2
        with:
          push: true
          platforms: linux/amd64, linux/arm64
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
```

</details>

### 1.3 - Understanding references to actions

As you can see, we're employing a workflow, `actions/setup-node`, which is used to install a specific Node.js version on the runner.

Let's dissect the reference to that action to understand its structure:

- `actions/` references the owner of the action, which is translated into a user or organization on GitHub.
- `setup-node` refers to the name of the action, which corresponds to a repository on GitHub.
- `@v3` represents the version of the action, which corresponds to a Git tag or a general reference (such as a branch or even a commit SHA) on the repository.

This reference structure makes it straightforward to navigate to the source code of any action by merely appending the `owner` and `name` to the `github.com` URL, like so: `https://github.com/{owner}/{name}`. For the above example, this would be <https://github.com/actions/setup-node>.

### 1.4 - Understanding Docker push

As in the previous action, we're first using the checkout action, afterward we're using the `docker/setup-buildx-action` to set up the Docker Buildx builder. This is required to build multi-architecture images which we intend to build to support Mac and Windows users. 
We must login to the Github Docker Registry by leveraging built-in variables and secrets. We prepare the Docker Image MetaData by using the `docker/metadata-action` and finally build and push the Docker Image using the `docker/build-push-action`.

We can see the multi-plaform build configuration `platforms: linux/amd64, linux/arm64`.


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

