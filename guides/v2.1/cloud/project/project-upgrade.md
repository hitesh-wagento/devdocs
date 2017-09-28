---
layout: default
group: cloud
subgroup: 130_upgrades
title: Upgrade Magento Commerce (Cloud)
menu_title: Upgrade Magento Commerce (Cloud)
menu_order: 10
menu_node:
version: 2.1
github_link: cloud/project/project-upgrade.md
redirect from:
  -  /guides/v2.0/cloud/howtos/upgrade-magento.html
  -  /guides/v2.1/cloud/howtos/upgrade-magento.html
  -  /guides/v2.1/cloud/howtos/upgrade-magento.html
---

This information details how to upgrade {{site.data.var.ece}} from any version to 2.1.X.

When you upgrade {{site.data.var.ece}}, you also upgrade with patches and available hotfixes as part of the `magento-cloud-metapackage`. Make sure you have `auth.json` in your project root folder if there isn’t one already.

Our upgrades are Composer driven. For more information on Composer, see [Composer in Cloud]({{ page.baseurl }}cloud/reference/cloud-composer.html).

<div class="bs-callout bs-callout-warning" markdown="1">
Always apply and test a patch your local system in an active branch. You can push and test in an Integration environment prior to deploying across all environments.
</div>

We recommend that you first back up the database of the system you are upgrading. Use the following steps to back up your integration, staging, and production systems.

## Back up the database {#backup-db}
Back up your integration system database and code:

1.  Enter the following command to make a local backup of the remote database:

        magento-cloud db:dump
2.  Enter the following command to back up code and media:

        php bin/magento setup:backup --code [--media]

    You can optionally omit `[--media]` if you have a large number of static files that are already in source control.

Back up your staging or production system database:

1.  [SSH to the server]({{ page.baseurl }}cloud/env/environments-ssh.html).
2.  Find the database login information:

        php -r 'print_r(json_decode(base64_decode($_ENV["MAGENTO_CLOUD_RELATIONSHIPS"]))->database);'

3.  Create a database dump:

        mysqldump -h <database host> --user=<database user name> --password=<password> --single-transaction <database name> | gzip - > /tmp/database.sql.gz

## Verify other changes {#verify-changes}
Verify other changes you're going to submit to source control before you start the upgrade:

1.  If you haven't done so already, change to your project root directory.
2.  Enter the following command:

        git status
3.  If there are changes you do *not* want to submit to source control, branch or stash them now.

## Complete the upgrade {#upgrade}

1.  Change to your Magento base directory and enter the following command:

        composer require magento/magento-cloud-metapackage <requiredversion> --no-update
        composer update

    For example, to upgrade to version 2.1.4:

        composer require magento/magento-cloud-metapackage 2.1.4 --no-update
        composer update

4.  Add, commit, and push your changes to initiate a deployment:

        git add -A
        git commit -m "Upgrade"
        git push origin <branch name>

    `git add -A` is required to add all changed files to source control because of the way Composer marshals base packages. Both `composer install` and `composer update` marshal files from the base package (that is, `magento/magento2-base` and `magento/magento2-ee-base`) into the package root.

    The files Composer marshals belong to the new version of Magento, to overwrite the outdated version of those same files. Currently, marshaling is disabled in Magento Commerce, so you must add the marshaled files to source control.

5.  Wait for deployment to complete.

6.  [Verify your upgrade](#upgrade-verify).

## Verify your upgrade {#upgrade-verify}
This section discusses how to verify your upgrade and to troubleshoot any issues you might find.

To verify the upgrade in your integration, staging, or production system:

1.  [SSH to the server]({{ page.baseurl }}cloud/env/environments-ssh.html).
2.  Enter the following command from your Magento root directory to verify the installed version:

        php bin/magento --version

## Troubleshoot your upgrade {#upgrade-verify-tshoot}
In some cases, an error similar to the following displays when you try to access your storefront or the Magento Admin in a browser:

    There has been an error processing your request
    Exception printing is disabled by default for security reasons.
      Error log record number: <error number>

### View error details on the server
To view the error in your integration system, [SSH to the server]({{ page.baseurl }}cloud/env/environments-ssh.html) and enter the following command:

    vi /app/var/report/<error number>

### Resolve the error
One possible error occurs when the deployment hook failed, and therefore the database has not yet been fully upgraded. If so, an error similar to the following is displayed:

    a:4:{i:0;s:433:"Please upgrade your database: Run "bin/magento setup:upgrade" from the Magento root directory.
    The following modules are outdated:
    Magento_Sales schema: current version - 2.0.2, required version - 2.0.3

To resolve the error:

1.  [SSH to the server]({{ page.baseurl }}cloud/env/environments-ssh.html).
2.  [Examine the logs]({{ page.baseurl }}cloud/trouble/environments-logs.html) to determine the source of the issue.
3.  After you fix the source of the issue, push the change to the server, which causes the upgrade to restart.

    For example, on a local branch, enter the following commands:

        git add -A && git commit -m "fixed deployment failure" && git push origin <branch name>

#### Related topic
* [Composer]({{page.baseurl}}cloud/reference/cloud-composer.html)
* [Install components]({{page.baseurl}}cloud/howtos/install-components.html)
* [Install optional sample data]({{page.baseurl}}cloud/howtos/sample-data.html)
* [Merge and delete an environment]({{page.baseurl}}cloud/howtos/environment-tutorial-env-merge.html)