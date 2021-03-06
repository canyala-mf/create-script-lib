# create-script-lib@1.0.1<!-- omit in toc -->

## Table of contents<!-- omit in toc -->

- [1. Abstract](#1-abstract)
  - [1.1. Commentary](#11-commentary)
  - [1.2. Releases](#12-releases)
- [2. Included Files](#2-included-files)
- [3. Pre-requisites](#3-pre-requisites)
  - [3.1. **N**ode **V**ersion **M**anager (NVM)](#31-node-version-manager-nvm)
  - [3.2. Prepare a local bin folder](#32-prepare-a-local-bin-folder)
  - [3.3. Github user-repo-access-token](#33-github-user-repo-access-token)
  - [3.4. Environment variables](#34-environment-variables)
  - [3.5. **J**SON **Q**uery (JQ)](#35-json-query-jq)
- [4. Install the source code](#4-install-the-source-code)
  - [4.1. Create a local repo](#41-create-a-local-repo)
  - [4.2. Make the tool available](#42-make-the-tool-available)
- [5. Using the scaffolding tool](#5-using-the-scaffolding-tool)
- [6. Developing the package](#6-developing-the-package)
- [7. Transfer ownership to an organisation](#7-transfer-ownership-to-an-organisation)
- [8. Publishing the package](#8-publishing-the-package)

## 1. Abstract

`create-script-lib` is a shell scaffolding utility for JS/[TS](https://www.typescriptlang.org) lib [npm](https://www.npmjs.com/)-packages with an internal [AS](https://www.assemblyscript.org/) core, an exposed [TS](https://www.typescriptlang.org) api and [TS](https://www.typescriptlang.org) unit tests.  

TypeScript (TS) adds build time quality checks and AssemblyScript (AS) adds performance to the parts of a library that need it.

There are repositories for both [TypeScript](https://github.com/microsoft/TypeScript) and [AssemblyScript](https://github.com/AssemblyScript/assemblyscript) on github.

### 1.1. Commentary

*The reasons for wrapping AS code with TS code is that it creates an encapsulation layer for code that use the library. Logic can be moved between the TS layer and the AS layer and vice versa if warranted, without affecting client code.*

*Data structures like arrays, strings and complex objects require special handling since assemblyscript ([webassembly](https://webassembly.org/) under the hood) and typescript (javascript under the hood) manage memory differently.*

*These interop differences can be mitigated in the TS layer and thus be hidden from client code. The ability to implement unit tests in typescript also improves the developer experience as well as the quality of a library.*

### 1.2. Releases

Version | Description | Known issues
---------|----------|-----------
 1.0.1 | *Feature complete and usable* | Stepping into wasm not yet working.
 0.0.1 | *Initial development version, do not use* | N/A

## 2. Included Files

File | Description  
---------|---------
 `create-script-lib.sh` | *The shell script for the scaffolding tool*
 `LICENSE` | *The MIT license for this tool*  
 `local-publish.sh` | *A utility script for local publish of the tool*
 `README.md` | *This `README.md` file*  
 `TEMPLATE.md` | *A `README.md` template for the scaffolded packages*

## 3. Pre-requisites

In order for `create-script-lib` to run, we need to prepare a few things before we can install the tool locally from source.  

### 3.1. **N**ode **V**ersion **M**anager (NVM)

NVM is a popular tool that allows you to have multiple versions of node installed.
You will be able to select a specific version to use as well as specifying a default version.

You can read about `nvm` [here](https://github.com/nvm-sh/nvm/blob/master/README.md) if you want to learn more.  

- Test if `nvm` is installed

```lang-bash
account@computer:$ nvm --version
```

If you do get something like this:

```lang-bash
Command 'nvm' not found, did you mean:

  command 'nvm' from deb kore (3.3.1-1)

Try: sudo apt install <deb name>
```

Do **NOT** try the suggested `sudo apt install <deb name>` above; instead, do the following:  

- Install nvm

```lang-bash
sudo apt install curl 
curl https://raw.githubusercontent.com/creationix/nvm/master/install.sh | bash 
source ~/.profile
nvm --version
```

- Install the latest versions of node

```lang-bash
nvm install node
nvm install --lts
```

- You can `ls` list which versions of node are installed locally and which are available remotely like this:

```lang-bash
nvm ls
nvm ls-remote
```

- Use the stable LTS version like this:

```lang-bash
nvm use --lts
```

- If you want to try out the latest version do this:

```lang-bash
nvm use node
```

- Using a specific version and setting it as default version might look like this:

```lang-bash
nvm use 14.15.1
nvm alias default 14.15.1
```

*At the time of writing, node@14.15.1 was the latest LTS available and it has been verified to work with the scaffolding tool. Using the **14.15.1** version and setting it as `default` is recommended.*

### 3.2. Prepare a local bin folder

The `local-publish.sh` and the `create-script-lib` scripts requires a `~/bin` folder to publish to. If you do not have such a folder you need to create one.

- How to create a bin folder:

```lang-bash
cd ~
mkdir bin
```

### 3.3. Github user-repo-access-token

Besides creating a local git repo for the package, the tool also creates the remote repo using a git api call that requires an access token in order to validate. A step by step instruction follows:

- Use a browser and go to your personal git account page.
- You need to login in order to continue, do that if needed.
- Click your happy face or your cool symbol in the top right corner.
- An account menu pops up. Click `Settings`
- To the left you find a button `Developer settings`. Click at it.
- The `Developer settings` page opens. Click `Personal access tokens`
- To the right we find a button `Generate new token`, click it.
- When asked for your password, provide it.
- In the `Note` box enter a name/description e.g `repo-access-token`
- Click the `repo` checkbox
- Scroll down and click the `Generate token` button.
- Copy the generated token value in the green box to the clipboard.
- Read the information in the blue box.

### 3.4. Environment variables

A couple of variables are needed by the `create-script-lib` and the `local-publish.sh` scripts.

- Open the `~/.bashrc` file using your favorite editor.
- Paste the access token value in the clipboard at the end so you do not lose it. You will move it to it´s proper location further down.
- Add the following to the end of the `~/.bashrc` file.

```lang-bash
export PATH="$PATH:~/bin"
export GH_API_TOKEN="github-user-repo-access-token"
export GH_USER="your-github-user-name"
export NPM_ORG="your-organisation"
```

- Substitute `"github-user-repo-accesstoken"`, `"your-github-user-name"` and `"your-organisation"` with alternatives suitable for you. Move the previously pasted `github-user-repo-access-token` value to it´s proper position.

- Apply the changes immidiately

```lang-bash
account@computer:$ source ~/.bashrc
```

### 3.5. **J**SON **Q**uery (JQ)

`jq` is a tool that allows you to edit json files from the command line. `create-script-lib.sh' uses it to edit config files. You can read about jq [here](http://manpages.ubuntu.com/manpages/focal/man1/jq.1.html).

Test for `jq` like this:

```lang-bash
jq --version
```

Install `jq` if needed like this:

```lang-bash
sudo apt install jq
```

## 4. Install the source code

### 4.1. Create a local repo

- Browse to <https://github.com/canyala/create-script-lib> or to a forked version of the repo.  

- Click the green `Code` button.

- Click the copy url button.

- Use a terminal to go to your parent repo folder.

- Clone the repo using git

```lang-bash
$git clone <paste_url_from_clipboard_here>
```

### 4.2. Make the tool available

- Make the local-publish.sh script executable.

```lang-bash
$cd create-script-lib 
$chmod +x local-publish.sh
```

- Publish the scaffolding tool script

```lang-bash
$./local-publish.sh
```

*The `local-publish.sh` script copies the `create-script-lib.sh` into the `~/bin` folder we created earlier and renames it to `create-script-lib` in the process while also making it executable.*

## 5. Using the scaffolding tool

The tool can be used in two ways:

- In a parent folder

```lang-bash
$create-script-lib a-nice-name-for-the-library
```

The tool creates a sub folder using `a-nice-name-for-the-library`. If such a folder already exist the tool fails gracefully and no library is created.

- In an empty folder

```lang-bash
$mkdir a-nicely-named-lib && cd a-nicely-named-lib
$create-script-lib
```

The tool use the folder name when writing configuration info etc and it also checks if the folder is empty. If not empty, the tool fails gracefully without changing or touching anything.  

## 6. Developing the package

The only actual implementation that the `create-script-lib` provides is an `add(a, b)` function together with tests for that function.

The rest of the implementation is up to you.

## 7. Transfer ownership to an organisation

By using the tool, you get a local repo and you also get a remote repo under your personal git account. If you need the repo to instead be owned by an organisation, you can change it by navigating to the package `settings` and further on to `transfer ownership` on github.

## 8. Publishing the package

Packages can be published to either or both of github and/or npm. The [npm](https://www.npmjs.com/) organisation is owned by github, which in turn is owned by microsoft, so it really doesn't matter where you publish to.

You could also publish to a private file share on your internal network. `npm install /path/to/package` works. This can be useful for early test scenarios.
