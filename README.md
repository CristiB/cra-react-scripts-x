
# Forking a monorepo (Forking CRA/*react-script*).

Usually when you fork a package you create a replica of the whole repository, make some changes and use your own version instead of the one that is published to npm. In the case of [create-react-app](https://facebook.github.io/create-react-app/) (and many other tools these days), they are using a monorepo. Simply put, this means that several packages are kept in one repository, making it easier to develop packages that have dependencies on each other.

When setting up a project using *create-react-app*, you'll find out that the only dependency we actually use after initializing the app, is the package *react-scripts*. This means that we are not really interested in forking react-dev-utils, babel-preset-react-app or any of the other packages that are also in the monorepo.

Many of our projects are also using a monorepo structure, meaning that ideally we want to have react-scripts living alongside our project. So how in the world do we fork parts of a monorepo and insert them into our own monorepo, while also maintaining the possibility to update the package when there's a new version, without having to revert all of our changes and start over?

Luckily, there's a good, but pretty unknown, answer to that.

## [git subtree](https://github.com/git/git/blob/master/contrib/subtree/git-subtree.txt)

This amazing feature should not be confused with git submodule, which is a whole another story. `git subtree` lets you import parts of a git repository into another repository, while keeping the commit history, meaning you can update your local version when the remote has changed.

This might sound like magic, but it's not. Lets go through it step by step.

---
## Setting up the source package
---

### ***1. Start out by cloning the source repo:***

```sh
git clone https://github.com/facebook/create-react-app.git
```

### ***2. Check out the specific branch or tag you want to base your fork on:***

```sh
git checkout v3.0.1
```

### ***3. Split the tree to get a branch containing only the package you want, in our example react-scripts:***

```sh
git subtree split -P packages/react-scripts -b react-scripts-fork
```

This will create a specific branch containing only what's inside the path given (by the `-P` flag), in this case only the package we want - *react-scripts.*

---
## Add to your repository
---

### ***1. In your own repository, add the source package as a new remote:***

```sh
git remote add react-scripts-cra-remote ../create-react-app
```

### ***2. Add package to your repository, in the specified directory:***

```sh
git subtree add -P packages/react-scripts react-scripts-cra-remote react-scripts-fork --squash
```
or

```sh
git subtree add -P packages/react-scripts react-scripts-cra-remote react-scripts@3.0.1 --squash
```


Using `--squash` means that the whole commit history will be squashed into a single commit when we add the package. This is usually what you want to do when forking an external package like this. If you want, you can omit `--squash` but your commit log will then contain the full history for all the files in the repo you just forked out of. This might or might not be desirable.

Now you should have your own fork of *react-scripts* in your repository, free to do whatever modifications you want!

---

## Updating the package
---

An important feature of `git subtree` is that you're able to update your fork when the remote has changed, such as when a new version of the package has been released.

### ***1. In you source repo, check out the new version and create a branch containing only that package (see step 2 and 3 under [Setting up the source package](#Setting-up-the-source-package)).***

### ***2. In your own repo, merge in the changes:***

```sh
 git subtree pull-P packages/react-scripts react-scripts-fork-remote react-scripts@3.0.1 --squash
```

*After you've fixed potential merge conflicts, you should have an updated version of your forked package. Voilà!*

---
## Publish custom react-scripts to NPM

### 1. From your terminal, change path to *../path/to/repo/packages/react-scripts*

### 2. Login to npm:
```sh
npm login
```
### 3. Go ahead and publish:

```sh
npm publish --access public
```
---
[Read this for more info.](https://www.hyperlab.se/blog/forking-parts-of-a-monorepo)