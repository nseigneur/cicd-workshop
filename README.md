# CI/CD WorkShop 
> Spot issues quickly, save time and be thrifty

## Table of Contents 📑
- [CI/CD WorkShop](#cicd-workshop)
  - [Table of Contents 📑](#table-of-contents-)
  - [Introduction 📝](#introduction-)
  - [Dependencies 📦](#dependencies-)
  - [Getting Started 🚀](#getting-started-)
  - [Workshop Goals 🎯](#workshop-goals-)
  - [Running the Application](#running-the-application)
  - [Other useful commands 🛠](#other-useful-commands-)
  - [References 📚](#references-)

## Introduction 📝

This workshop is designed to help you understand the importance of CI/CD and how to implement it in your project. We will be using [GitHub Actions]() to automate our CI/CD pipeline. 
We will also be using [AWS S3]() to host our application.Below you will find a list of dependencies and tools you will need to complete this workshop.

We have provided a baseline CI for building and deploying the application, we invite you to choose one primary workshop goal and complete it. If you have time, you can complete the other goals as well.

Following this workshop, you will create a set of GitHub Action workflows in order to **test, release, secure** and **deploy** the **Rocketdex** 🚀 - an awesome React-Application that lets you view and collect all your favorite rockets!

You will learn how to

- run your **GitHub Actions**
- use [**the Marketplace**](https://github.com/marketplace)
- create a complete and secure **CI pipeline** from scratch

## Dependencies 📦
- [Node.js](https://nodejs.org/en/)
- [npm](https://www.npmjs.com/)
- [git](https://git-scm.com/)
- [GitHub Account](https://github.com)
- [AWS S3 Credentials](https://vault.indigolabs.ca/)

## Getting Started 🚀

Anything you need for this workshop is contained within this template-repository - so the first thing you need to do is get a copy of it:

1. On the top right, click the green `Use this template` button
    ![Image of the green button saying Use this Repo](./docs/images/setsup_use-this-template.png)
2. Give your new repository any name you like (for example `<your-name>-gh-actions-workshop`)
3. Decide for an owner of the new repository (when in doubt, use your private GitHub account)
4. Make it a `Public` repository so you don't use up any of your private action minutes
5. Click `Create repository from template`

![Image of Repository Creation Screen with settings as explained above](docs/images/setup_create-repo.png)

Now, you can start working on the [basics of your CI/CD pipeline](./docs/basics.md) once completed, you can choose your [workshop goal](#workshop-goals-) below.

## Workshop Goals 🎯

These are the self-serve documents you can follow to complete this workshop.

1. **[Code Quality & Commit management](./docs/code_quality.md)**
2. **[QA & Test Coverage](./docs/qa_tests.md)**
3. **[Security](./docs/security.md)**

## Running the Application 

The **Rocketdex** application is a [React](https://reactjs.org/)-Application build with [vite](https://vitejs.dev/) and tested with [vitest](https://vitest.dev/).

You can simply clone it and install all dependencies using:

```sh
npm install
```

After that, you can start it in development mode to have a look at it:

```sh
npm run dev
```

## Other useful commands 🛠

| Description                                               | Command         |
| --------------------------------------------------------- | --------------- |
| Build and Bundle the App using [vite](https://vitejs.dev) | `npm run build` |
| Run Unit Tests with [vitest](https://vitest.dev/)         | `npm run test`  |
| Lint the Code with [ESLint](https://eslint.org/)          | `npm run lint`  |

## References 📚
- [Reference Workshop](https://github.com/actions-workshop/actions-workshopp)
- [ESLint Conventions](https://typescript-eslint.io/rules/naming-convention/)
