# Test application basics

## 1 - Inspect the repository

This repository contains a [React](https://reactjs.org/)-based application built with [Vite](https://vitejs.dev/). We aim to automate its testing and building in this lab.

Feel free to explore the files if you're curious about the app's operation (though it's not strictly necessary for understanding the remainder of the workshop).

- [`src/main.tsx`](../src/main.ts) : This is the main entry point of the application.
- [`src/pages/Home.tsx`](../src/pages/Home.tsx) : This route contains most of what you will see upon launching the application.
- [`src/pages/Home.test.ts`](../src/pages/Home.test.tsx) : Here, you will find [`vitest`](https://vitest.dev/) tests that we will run with GitHub Actions.
- [`Dockerfile`](../Dockerfile) : This Docker file packages the application into a container that will be used later in this workshop.

If you want to test the application, you can start a Codespace and run it with the command `npm run dev` 
(you can also run the tests with `npm test`). To run the application on your local machine, you will need to install Node.js first. For running locally, you need to clone the repository with `git clone https://<repository_name>`

To test the container, run `docker build . -t local:latest` to build the image and `docker run -p 8080:8080 local:latest` to run it. These commands require a local installation of [Docker](https://www.docker.com/).

