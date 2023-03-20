# Contribution guidelines

We're glad that you would like to become a Percona community member and participate in keeping open source open.  

There are many ways how you can contribute:

* Submit bug reports or feature requests.
* Submit a code patch.
* Contribute to documentation.

# Submit bug reports or feature requests

If you find a bug in Percona Backup for MongoDB, you can submit a report to the [JIRA issue tracker](https://jira.percona.com/projects/PBM)
for Percona Backup for MongoDB.

Start by searching the open tickets for a similar report. If you find that
someone else has already reported your problem, then you can upvote that report
to increase its visibility.

If there is no existing report, submit a report following these steps:


1. Sign in to [JIRA issue tracker](https://jira.percona.com/projects/PBM). You will need to create an account if you
do not have one.


2. In the *Summary*, *Description*, *Steps To Reproduce*, *Affects Version* fields
describe the problem you have detected. For PBM the important diagnostic
information is: log files from the pbm-agents; a dump of the
PBM control collections.

As a general rule of thumb, try to create bug reports that are:


* *Reproducible*: Describe the steps to reproduce the problem.


* *Specific*: Include the version of Percona Backup for MongoDB, your environment, and so on.


* *Unique*: Check if there already exists a JIRA ticket to describe
the problem.


* *Scoped to a Single Bug*: Report only one bug in one JIRA ticket.

## Submit a code patch

If you'd like to submit a code patch, follow the [Contributing guide in PBM code repository](https://github.com/percona/percona-backup-mongodb/blob/main/CONTRIBUTING.md). 

## Contribute to documentation

### Edit documentation online via GitHub

1. Click the <img src="_resource/.icons/edit_page.png" width="20px" height="20px"/> **Edit this page** icon next to the page title. The source `.md` file of the page opens in GitHub editor in your browser. If you haven’t worked with the repository before, GitHub creates a [fork](https://docs.github.com/en/github/getting-started-with-github/fork-a-repo) of it for you.

2. Edit the page. You can check your changes on the **Preview** tab.

3. Commit your changes.

	 - In the *Commit changes* section, describe your changes.
	 - Select the **Create a new branch for this commit and start a pull request** option
	 - Click **Propose changes**.

4. GitHub creates a branch and a commit for your changes. It loads a new page on which you can open a pull request to Percona. The page shows the base branch - the one you offer your changes for, your commit message and a diff - a visual representation of your changes against the original page.  This allows you to make a last-minute review. When you are ready, click the **Create pull request** button.
5. Someone from our team reviews the pull request and if everything is correct, merges it into the documentation. Then it gets published on the site.

### Edit documentation locally

If you wish to have full control over the documentation process, follow the [Contributing to documentation guide](https://github.com/percona/pbm-docs/blob/main/CONTRIBUTING.md).
