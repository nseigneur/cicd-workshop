# Continuous Delivery

In the previous lab, you used GitHub Actions to create a continuous integration (CI) workflow. The next step in a classical continuous delivery process is to **package and release** your application.

In this lab, you will extend the workflow you created to package the application as a container image and publish it to the GitHub Container Registry.

We keep things simple, by pushing a Docker Container to GitHub Registry - we could deploy "anywhere" from there.

## 1 - Using the visualization graph

Every workflow run generates a real-time graph that illustrates the run progress. You can use this graph to monitor and debug workflows. The graph displays each job in the workflow. An icon to the left of the job name indicates the status of the job. Lines between jobs represent dependencies.

## 2 - Dependent jobs

By default, the jobs in your workflow run in parallel at the same time. If you have a job that must run only after another job has completed, you can use the `needs` keyword to create this dependency. If one of the jobs fails, all dependent jobs are skipped; however, if you want the jobs to continue, you can define this using the `if` conditional statement. In the following example, the `build` and `publish-container` jobs run in series, with `publish-container` dependent on the successful completion of `build`:

```yml
jobs:
  build:
    needs: setup
    runs-on: ubuntu-latest
    steps:
      # Build Node application
      - ...
        ...
  publish-container:
    needs: build
    runs-on: ubuntu-latest
    steps:
      # Build and publish Docker image
      - ...
        ...
```

## 3 - Package your application as a container image

For delivering your application, you will need to complete the following steps:

1. Create a new job that depends on the `build` job.
2. Add steps to build and publish a container image.

When building workflows, you should always check the GitHub Marketplace to see if certain actions can perform some of the workflow steps for you.

#### GitHub Marketplace

1. Visit the GitHub Marketplace: <https://github.com/marketplace>
2. Search for "Docker".
3. Scroll down to the **Actions** section.

You will find many actions related to Docker. For this lab, you will use the following actions:

- [Docker Login](https://github.com/marketplace/actions/docker-login): to connect to the GitHub Container Registry (<https://ghcr.io>).
- [Build and push Docker images](https://github.com/marketplace/actions/build-and-push-docker-images).

### 3.1 - Edit the workflow

1. Edit the file `.github/workflows/node.js.yml`, and add the `package-and-publish` job so the file looks like this:

```yaml
name: Packaging

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

1. Notice that Build and Push Docker image supports multi-platform `platforms: linux/amd64, linux/arm64` building silmutaneously for `amd64` and `arm64` architectures.

2. Commit the changes to `.github/workflows/node.js.yml`.

3. Upon pushing, the workflow will automatically start and carry out the full CI process.

4. Review the workflow runs and inspect the "Build and Publish Container Image" logs. The packaging and publishing task may take several minutes to complete.

## 4 - Locate your image in the GitHub Container Registry

1. Navigate to your project's main page.
2. Click on the **Packages** link on the right menu.
3. Select your container.

![](../images/img-037.png)

## Conclusion

In this lab, you have learned how to:

- 👏 Build and publish a container image using GitHub Actions.
- 👏 Navigate to GitHub Packages.

---


