# Contributing guide

Welcome to Percona Backup for MongoDB documentation!

We're glad that you would like to become a Percona community member and participate in keeping open source open.  

Percona Backup for MongoDB (PBM) is a distributed, low-impact solution for achieving consistent backups of MongoDB sharded clusters and replica sets.

This repository contains the source file for PBM documentation and this document explains how you can contribute to it. 

If you'd like to submit a code patch, follow the [Contributing guide in PBM code repository](https://github.com/percona/percona-backup-mongodb/blob/main/CONTRIBUTING.md). 

## Contributing to documentation

Percona Backup for MongoDB documentation is written in [reStructured text markup language](https://docutils.sourceforge.io/rst.html) and is created using [Sphinx Python Documentation Generator](https://www.sphinx-doc.org/en/master/). The doc files are in the ``source`` directory.

To contribute to documentation, learn about the following:
- [reStructured text](https://www.sphinx-doc.org/en/master/usage/restructuredtext/basics.html) markup language. 
- [Sphinx](https://www.sphinx-doc.org/en/master/usage/quickstart.html) documentation generator. We use it to convert source ``.rst`` files to html and PDF documents.
- [git](https://git-scm.com/)
- [Docker](https://docs.docker.com/get-docker/). It allows you to run Sphinx in a virtual environment instead of installing it and its dependencies on your machine.

### Edit documentation online via GitHub

1. Click the **Edit this page** link on the sidebar. The source ``.rst`` file of the page opens in GitHub editor in your browser. If you haven’t worked with the repository before, GitHub creates a [fork](https://docs.github.com/en/github/getting-started-with-github/fork-a-repo) of it for you.

2. Edit the page. You can check your changes on the **Preview** tab.

   **NOTE**: GitHub’s native markup language is [Markdown](https://daringfireball.net/projects/markdown/) which differs from the reStructured text. Therefore, though GitHub renders titles, headings and lists properly, some rst-specific elements like variables, directives, internal links will not be rendered.

3. Commit your changes.

	 - In the *Commit changes* section, describe your changes.
	 - Select the **Create a new branch for this commit and start a pull request** option
	 - Click **Propose changes**.

4. GitHub creates a branch and a commit for your changes. It loads a new page on which you can open a pull request to Percona. The page shows the base branch - the one you offer your changes for, your commit message and a diff - a visual representation of your changes against the original page.  This allows you to make a last-minute review. When you are ready, click the **Create pull request** button.
5. Someone from our team reviews the pull request and if everything is correct, merges it into the documentation. Then it gets published on the site.

### Edit documentation locally

This option is for users who prefer to work from their computer and / or have the full control over the documentation process.

The steps are the following:

1. Fork this repository
2. Clone the repository on your machine:

```sh
git clone git@github.com:<your_name>/percona-backup-mongodb.git
```

3. Change the directory to ``percona-backup-mongodb`` and add the remote upstream repository:

```sh
git remote add upstream git@github.com:percona/percona-backup-mongodb.git
```

4. Pull the latest changes from upstream

```sh
git fetch upstream
git merge upstream/main
```

5. Create a separate branch for your changes

```sh
git checkout -b <my_branch>
```

6. Make changes
7. Commit your changes. The [commit message guidelines](https://gist.github.com/robertpainsi/b632364184e70900af4ab688decf6f53) will help you with writing great commit messages

8. Open a pull request to Percona

### Building the documentation

To verify how your changes look, generate the static site with the documentation. This process is called *building*. You can do it in these ways:
- [Use Docker](#use-docker)
- [Install Sphinx and build locally](#install-sphinx-and-build-locally)

#### Use Docker

1. [Get Docker](https://docs.docker.com/get-docker/)
2. We use [this Docker image](https://hub.docker.com/r/ddidier/sphinx-doc) to build documentation. Run the following command:

```sh
docker run --rm -i -v `pwd`:/doc -e USER_ID=$UID ddidier/sphinx-doc:0.9.0 make clean thtml
```
   If Docker can't find the image locally, it first downloads the image, and then runs it to build the documentation.

3. Go to the ``build/html`` directory and open the ``index.html`` file to see the documentation.

#### Install Sphinx and build locally

1. Install [pip](https://pip.pypa.io/en/stable/installing/)
2. Install [Sphinx](https://www.sphinx-doc.org/en/master/usage/installation.html).
3. While in the root directory of the doc project, run the following command to build the documentation:

```sh
make clean thtml
```

4. Go to the ``build/html`` directory and open the ``index.html`` file to see the documentation.
5. To create a PDF version of the documentation, run the following command:

```sh
make clean latexpdf
```

The PDF document is in the ``build/latex`` folder.

If you wish to have a live preview of your changes, do the following:

- Install the [sphinx-autobuild](https://pypi.org/project/sphinx-autobuild/) extension
- Run the following command:

```sh
sphinx-autobuild  -b html -d build/doctrees -c source/conf-material --open-browser source build/html
```

## After your pull request is merged

Once your pull request is merged, you are an official Percona Community Contributor. Welcome to the community!
