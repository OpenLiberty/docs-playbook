# How are the docs on openliberty.io built and what is a docs playbook?

The docs on https://openliberty.io/ are built using a tool called [Antora](https://antora.org/). It is a tool that helps streamline writing, managing, and versioning documentation.

The [docs playbook](https://github.com/OpenLiberty/docs-playbook/blob/prod/antora-playbook.yml) is a key file used by Antora see https://docs.antora.org/antora/latest/playbook/. It tells the build what branches and versions of the [docs repo](https://github.com/OpenLiberty/docs) to use. Antora aggregates all of the content specified in the playbook and creates versions of pages based on the `version` attribute in the `antora.yml` of each doc branch. See https://github.com/OpenLiberty/docs/blob/vNext/antora.yml for an example. If Antora finds the same doc in multiple doc branches, it will create a version picker on the site that lets you easily switch between versions of the doc.

There are three doc playbooks used in the builds, located in the following branches in this repo: [prod](https://github.com/OpenLiberty/docs-playbook/blob/prod/antora-playbook.yml), [staging](https://github.com/OpenLiberty/docs-playbook/blob/staging/antora-playbook.yml), and [draft](https://github.com/OpenLiberty/docs-playbook/blob/draft/antora-playbook.yml).

https://openliberty.io/ is built using the [prod playbook](https://github.com/OpenLiberty/docs-playbook/blob/prod/antora-playbook.yml). This file explicitly only contains versions of the documentation for released Open Liberty versions. This is the file that needs to be [updated each release](#publishing-a-new-release-of-open-liberty-docs) to add the new release to be published to openliberty.io. This is intentionally manual and does not use wildcards for the branches to prevent any unintended branches/content from being published to the site.

The [staging site](https://staging-openlibertyio.mqj6zf7jocq.us-south.codeengine.appdomain.cloud/docs/latest/overview.html) is built using the [staging playbook](https://github.com/OpenLiberty/docs-playbook/blob/staging/antora-playbook.yml). This playbook contains the `staging` branch which is used to stage in changes for the next documentation version to be released. It contains a wildcard `V*.0.0.*-staging` rule which pulls in all of the branches used to stage in changes to already released versions of documentation. This branch does not need any updating each release, and shouldn't need many, if any, updates in general.

The [draft site](https://draft-openlibertyio.mqj6zf7jocq.us-south.codeengine.appdomain.cloud/docs/latest/overview.html) is built using the [draft playbook](https://github.com/OpenLiberty/docs-playbook/blob/draft/antora-playbook.yml). This playbook contains the `draft` branch which pulls in all of the content currently being drafted. It contains a wildcard `V*.0.0.*-draft` which pulls in all of the branches used to draft changes to already released versions of documentation. This branch does not need any updating each release, and shouldn't need many, if any, updates in general.

# Publishing a new release of Open Liberty Docs

This is the flow that must be followed when releasing a new version of docs on openliberty.io. These steps update the `vNext`, `staging` and `draft` branches of the [docs repo](https://github.com/OpenLiberty/docs) so that the next version of docs can be worked on and ensures that fixes can still be made to the docs of the version being released. This procedure assumes you have already cloned the [docs](https://github.com/OpenLiberty/docs), [docs-generated](https://github.com/OpenLiberty/docs-generated), and docs-playbook repos.

The following sections provide instructions to publish an Open Liberty release from the command line. For similar instructions that use the GitHub desktop client, see [Publishing a new release of Open Liberty Docs by using GitHub desktop](#publishing-a-new-release-of-Open-Liberty-Docs-by-using-GitHub-desktop)

## Updating the generated docs and API docs

The Open Liberty generated docs, which comprise the majority of the **Features** and **Server configuration** sections of the doc, are built from the [docs-generated repo](https://github.com/OpenLiberty/docs-generated). For each Open Liberty release, Chuck Bridgham from the kernel team posts the updated generated docs to the `draft` branch of the docs-generated repo. Contact Chuck several days before the release to ensure the generated docs are updated on draft in time to publish. 

To verify the generated doc on draft, go to the [docs-generated repo draft branch](https://github.com/OpenLiberty/docs-generated/tree/draft) in a browser and find Chuck's most recent PR in the Git history. Look for a change in the PR, such as updates to a config element or a new feature page, and check the [docs draft site](https://docs-draft-openlibertyio.mqj6zf7jocq.us-south.codeengine.appdomain.cloud/docs/latest/overview.html) to confirm that the changes are showing.

### Updating the Open Liberty API and SPI documentation

 The Open Liberty API and SPI documentation is updated with each 4-week release of the runtime. The navigation for the API and SPI sections of the docs is updated along with the generated docs. However, you must manually add the Javadoc API and SPI files to the staging branch of the docs-javadoc repository so that when the navigation builds on the staging site, the files are available for review and verification. For more information, see [Open Liberty Javadoc](https://github.com/OpenLiberty/docs-javadoc/).
 
### Preparing the generated doc for publication

Once you have verified the generated doc on the draft site, open a PR from `draft` to `staging` in the docs-generated repo and get it reviewed and merged. Once youy verify the generated doc on the `staging` site, open a PR from `staging` to `vNext` in the docs-generated repo. Get the PR reviewed and merge it in.

## Publishing the docs and preparing for the next release

1. Once all of the changes look correct on the [staging site](https://staging-openlibertyio.mqj6zf7jocq.us-south.codeengine.appdomain.cloud/docs/latest/overview.html), and all of the `staging` branch changes have been merged into the `vNext` branch in the docs and docs-generated repos, check out the `staging` branch in the docs repo and run `git pull` to get the updates. Then, create a branch off of the `vNext` branch with the next release version number. E.g. if the most recent version on openliberty.io is `20.0.0.8`, then create a `v20.0.0.9`  branch off of `vNext` by checking out the `vNext` branch and running `git checkout -b v20.0.0.9` and `git push` the branch.

2. Create `vX.0.0.X-staging` and `vX.0.0.X-draft` branches off of the new version branch just created which will be used for retroactive fixes to this version in the future. E.g. if `v20.0.0.9` is the version being released, then from that branch run `git checkout -b v20.0.0.9-staging` and push that branch to the docs repo. Switch back to the `v20.0.0.9` branch and run `git checkout -b v20.0.0.9-draft` and push that branch to the docs repo.
   
3. Repeat steps 1 and 2 for the [docs-generated repo](https://github.com/OpenLiberty/docs-generated).

4. Update the docs-playbook `antora-playbook.yml` file to add the new release version to production and remove the oldest release version from production, draft, and staging.

   Openliberty.io docs support 2 years worth of previous releases, which can be selected from the version picker at the top of the docs TOC. With each release, you must add the new release version and remove the oldest release version from the `antora-playbook.yml` file to update the version picker.
   
   a. Create a new branch off of the `prod` branch of this repo (docs-playbook) to add the new release version to the `branches` section of the [`antora-playbook.yml`](https://github.com/OpenLiberty/docs-playbook/blob/prod/antora-playbook.yml). 
   
   b. Remove the oldest branch from the beginning of the list. This version should be 2 years + 1 release behind the current release, ie, if you are publishing 22.0.0.8, you should remove `v20.0.0.7` from the list of branches.
   
   c. Make a pull request into the `prod` branch. Once the pull request has been reviewed merge it in.

   d. Repeat steps 4a-4c for the [staging](https://github.com/OpenLiberty/docs-playbook/blob/staging/antora-playbook.yml) branch of this repo.
   
   d. The branches list is specified differently in the [draft](https://github.com/OpenLiberty/docs-playbook/blob/draft/antora-playbook.yml) branch of the docs playbook, which use a wildcard schema to define the branches (for example, `branches: [v*.0.0.*-draft, draft]`). To remove the oldest release, you must exclude the corresponding version branch by appending an exclamation point (`!`) to the branch name, surrounding it in single quotation marks, and adding it to the list of branches. For example, to remove 20.0.0.7 from draft, you must specify `branches: [v*.0.0.*-draft, '!v20.0.0.7-draft', draft]`. For each release, add the value that you want to remove to the list. Do not replace any existing values. For example, to remove v.20.0.0.8, add it to the list instead of replacing 20.0.0.7: `branches: [v*.0.0.*-draft, '!v20.0.0.7-draft', '!v20.0.0.8-draft', draft]`.

5. In the docs repo, change the `version` in `antora.yml` in the [`staging`](https://github.com/OpenLiberty/docs/blob/staging/antora.yml) and [`draft`](https://github.com/OpenLiberty/docs/blob/draft/antora.yml) branches to the next release `vX.0.0.X` version incremented from the current release being published to openliberty.io. Once those changes have been reviewed merge them in. This is important so that the staging and draft builds can complete. Antora requires all of the versions of `antora.yml` of the branches being used in a build to have a unique version so they need to be updated to be unique from the version just released. You can either change the version and commit to draft/staging or create a pull request to do so.

6. Once the antora.yml [`staging`](https://github.com/OpenLiberty/docs/blob/staging/antora.yml) and [`draft`](https://github.com/OpenLiberty/docs/blob/draft/antora.yml) versions have been updated in the docs repo, create a pull request from `staging` to `vNext` to update the `antora.yml` version in `vNext`.

7. Repeat steps 5 and 6 for the [docs-generated repo](https://github.com/OpenLiberty/docs-generated/blob/vNext/antora.yml)

8. Antora natively supports automatically redirecting to the latest version by replacing the latest numerical version number in the URL with `latest`. However, you must update the doc-redirects properties file so that if a user manually types the latest numerical version into the URL they are redirected to the `latest` symbolic version.

   In the docs-playbook repo, update [the redirects file on `prod`](https://github.com/OpenLiberty/docs-playbook/blob/prod/doc-redirects.properties) so that `/docs/xx.0.0.x/*=/docs/latest/`, where `xx.0.0.x` is the version of the doc that is being released. Then, update the same file on [`draft`](https://github.com/OpenLiberty/docs-playbook/blob/draft/doc-redirects.properties) and [`staging`](https://github.com/OpenLiberty/docs-playbook/blob/staging/doc-redirects.properties) to point to the _next_ upcoming release (the newest released version + 1). For example, if the version that is being released is `22.0.0.5`, you must define `/docs/22.0.0.5/*=/docs/latest/` in the prod version of the docs-redirects properties file and `/docs/22.0.0.6/*=/docs/latest/` in the draft and staging versions of the file.

10. Before you proceed, make sure all of the pull requests and changes from previous steps have been merged into their respective repos (docs, docs-generated, and docs-playbook).
11. Confirm the updated content and release versions on the draft and staging sites.
   a. The PRs that you merged in step 7 & 8 triggered builds of the [`docs-draft site`](https://docs-draft-openlibertyio.mqj6zf7jocq.us-south.codeengine.appdomain.cloud/docs/latest/overview.html), [`full draft site`](https://draft-openlibertyio.mqj6zf7jocq.us-south.codeengine.appdomain.cloud/docs/latest/overview.html), [`docs-staging site`](https://docs-staging-openlibertyio.mqj6zf7jocq.us-south.codeengine.appdomain.cloud/docs/latest/overview.html), and [`full staging site`](https://staging-openlibertyio.mqj6zf7jocq.us-south.codeengine.appdomain.cloud/docs/latest/overview.html). Since the changes made in the previous steps were only to the doc specific repos, you can use the doc specific sites to validate your changes which take ~25min to redeploy (whereas the full sites take ~60min). If you have access, you can track the build progress in the [#was-ol-draft-site-alerts](https://ibm-cloud.slack.com/archives/C01G7L68KAP) and [#was-ol-staging-site-alerts](https://ibm-cloud.slack.com/archives/C01GX9P8YP2) slack channels.
   b. Once the builds complete, you can verify them on the draft and staging sites, which should now default to the _next_ upcoming version as `latest`. Look for the next upcoming version in the version-picker display at the beginning of the table of contents. When the prod site publishes, it will default to the version that is being released.

12. Finally, in the [#was-open-liberty-site](https://ibm-cloud.slack.com/archives/C4U7TQUSY) Slack channel, request a rebuild of the production openliberty.io site to publish the release. Once a trigger is added to kick off a prod site build for changes to the `prod` branch of this repo, this step will be removed.

# Publishing a new release of Open Liberty Docs by using GitHub Desktop

These instructions are identical to those in the previous section except for Git processes are described from the perpsective of using the GitHub Desktop client instead of the command line. This procedure assumes you have already have GitHub Desktop and have already cloned the [docs](https://github.com/OpenLiberty/docs), [docs-generated](https://github.com/OpenLiberty/docs-generated), and docs-playbook repos.

## Updating the generated docs and API Docs
The Open Liberty generated docs, which comprise the majority of the **Features** and **Server configuration** sections of the doc, are built from the [docs-generated repo](https://github.com/OpenLiberty/docs-generated). For each Open Liberty release, Chuck Bridgham from the kernel team posts the updated generated docs to the draft branch of the docs-generated repo. Contact Chuck several days before the release to ensure the generated docs are updated on draft in time to publish. 

To verify the generated doc on draft, go to the [docs-generated repo draft branch](https://github.com/OpenLiberty/docs-generated/tree/draft) in a browser and find Chuck's most recent PR in the Git history. Look for a change in the PR, such as updates to a config element or a new feature page, and check the [docs draft site](https://docs-draft-openlibertyio.mqj6zf7jocq.us-south.codeengine.appdomain.cloud/docs/latest/overview.html) to confirm that the changes are showing.

### Updating the Open Liberty API and SPI documentation

 The Open Liberty API and SPI documentation is updated with each 4-week release of the runtime. The navigation for the API and SPI sections of the docs is updated along with the generated docs, as described in the previous section. However, you must manually add the Javadoc API and SPI files to the staging branch of the docs-javadoc repository so that when the navigation builds on the staging site, the files are available for review and verification. 
 
 1. Before you send the generated docs to staging, go to the Open Liberty docs-javadoc repo and complete steps 1-10 of [Preparing Open Liberty API and SPI Javadoc files for publication.](https://github.com/OpenLiberty/docs-javadoc/). These steps make the updated Javadocs available to the staging site build.
 2. Complete steps 1-6 in the [Preparing the generated doc for publication](#Preparing-the-generated-doc-for-publication) section of this page.
 3. Return to the Open Liberty Javadoc Repo and complete steps 11-13 of [Preparing Open Liberty API and SPI Javadoc files for publication.](https://github.com/OpenLiberty/docs-javadoc/). In these steps, you verify the Javadoc on the newly updated staging site and commit the files to the prod branch so they are available for the production release.
 
### Preparing the generated doc for publication
Once you have verified the generated doc on the draft site and added the API and SPI doc files to the staging branch of the docs-javadoc repository, open a PR from `draft` to `staging` in the docs generated repo:

1. Open Github Desktop and select the docs-generated repo.
2. Select `draft` in the **Current Branch** dropdown menu and click **Fetch origin**.
3. Click the **Create a Pull Request** button.
4. In the resulting PR, click the **Edit** button and change the Base to `staging`.
5. Get the resulting PR reviewed and merged. Merging the PR kicks off a build of the staging site.
6. Verify the generated doc, API doc, and SPI doc on the [staging](https://docs-staging-openlibertyio.mqj6zf7jocq.us-south.codeengine.appdomain.cloud/docs/latest/overview.html) site.

Next, Commit the changes to the `vNext` branch. 
7. Open Github Desktop and select the docs-generated repo.
8. Select `staging` in the **Current Branch** dropdown menu and click **Fetch origin** to get any new changes.
9. Click the **Create a Pull Request** button. Give the PR a descriptiive title.
10. Get the resulting PR reviewed and merged.


## Publishing the docs and preparing for the next release

1. Once all of the changes look correct on the [staging site](https://staging-openlibertyio.mqj6zf7jocq.us-south.codeengine.appdomain.cloud/docs/latest/overview.html), and all of the `staging` branch changes have been merged into the `vNext` branch in the docs and docs-generated repos, create the branches you need for the new release in the docs repo.

   a. To confirm that all doc changes have merged from staging to vNext, go to the docs repo in GithUb Desktop and select the `staging` in the **Current Branch** menu. Click the **Create a Pull Request** button. If all changes are merged, you will see "There's nothing to compare". If you see any changes, verify and merge them in.
   
   b. In Github Desktop, make sure `vNext` is showing in the **Current Branch** menu. click **Fetch origin** to get any new changes.

   c. Click the **New Branch** button and create a branch with the next release version number. E.g. if the most recent version on **openliberty.io** is `20.0.0.8`, then create a `v20.0.0.9`  branch. 

2. Create `vX.0.0.X-staging` and `vX.0.0.X-draft` branches off of the new version branch you just created, which will be used for retroactive fixes to this version in the future.

   a. E.g. if `v20.0.0.9` is the version being released, then select `v20.0.0.9` from the **Current Branch** menu in GitHub desktop. Click the **New Branch** button to create a new branch called `v20.0.0.9-staging`.

   b. Switch back to `v20.0.0.9` branch and repeat to create `v20.0.0.9-draft`. **Don't forget to publish the branch all the branches that you create by clicking "Publish this branch" after you create each branch**.
  
3. Repeat steps 1 and 2 for the [docs-generated repo](https://github.com/OpenLiberty/docs-generated).

4. Update the docs-playbook `antora-playbook.yml` file to add the new release version to production and remove the oldest release version from production, draft, and staging.

   Openliberty.io docs support 2 years worth of previous releases, which can be selected from the version picker at the top of the docs TOC. You must add the new release version and remove the oldest release version fromn the `antora-playbook.yml` file to update the version picker.

   a. Create a new branch off of the `prod` branch of this repo (docs-playbook) to add the new release version to the `branches` section of the `antora-     playbook.yml`. You can make this change in your browser. Go to the [antora-playbook.yml](https://github.com/OpenLiberty/docs-playbook/blob/prod/antora-playbook.yml).  

   b. Click the pencil icon to edit the file. 

   c. Add the new release version to the `branches` sections of the doc. Remove the oldest release from the beginning of the list. 

   d. After you edit the file, create a pull request into the `prod` branch by clicking the **Propose changes** button. Get the pull request reviewed and merge it in.
   
   e. Remove the oldest release version from the `antora-playbook.yml` in the draft and staging branches. The branches list is specified differently in the draft and staging branches of the docs playbook, which use a wildcard schema to define the branches (for example, `branches: [v*.0.0.*-draft, draft]`). To remove the oldest release, you must exclude the corresponding version branch by appending an exclamation point (`!`) to the branch name, surrounding it in single quotation marks, and adding it to the list of branches. For example, to remove 20.0.0.7 from draft, you must specify `branches: [v*.0.0.*-draft, '!v20.0.0.7-draft', draft]`. For each release, add the value that you want to remove to the list. Do not replace any existing values. For example, to remove v.20.0.0.8, add it to the list instead of replacing 20.0.0.7: `branches: [v*.0.0.*-draft, '!v20.0.0.7-draft', '!v20.0.0.8-draft', draft]`. Follow the same process you used in steps 4b-4d to make the changes in both the draft](https://github.com/OpenLiberty/docs-playbook/blob/draft/antora-playbook.yml) and [staging](https://github.com/OpenLiberty/docs-playbook/blob/staging/antora-playbook.yml) version of the file.
   
5. Update the `antora.yml` file in the `draft` and `staging` branches of the docs repo.

   This is important so that the staging and draft builds can complete. Antora requires all of the versions of `antora.yml` of the branches being used in a build to have a unique version so they need to be updated to be unique from the version just released. You can either change the version and commit to draft/staging or create a pull request to do so.
   
   a. In GitHub Desktop, create a new vNext-based branch and give it a descritpive name (eg `22.0.0.9 yaml updates`).
   
   b. In your local GitHub Open Liberty docs folder, open the `antora.yml`file and change the `version` value to the next release `vX.0.0.X` version, incremented from the current release being published to openliberty.io. For example, if `22.0.0.6` is being released, set the version to `22.0.0.7`.
   
   c. Create a PR to add this change the [`staging`](https://github.com/OpenLiberty/docs/blob/staging/antora.yml) branch. Get the PR reviewed and merge it in.

   d. Create a PR to add the change to the  [`draft`](https://github.com/OpenLiberty/docs/blob/draft/antora.yml) branch. Get the PR reviewed and merge it in.

6. Once the antora.yml [`staging`](https://github.com/OpenLiberty/docs/blob/staging/antora.yml) and [`draft`](https://github.com/OpenLiberty/docs/blob/draft/antora.yml) versions have been updated in the docs repo, create a pull request from `staging` to `vNext` to update the `antora.yml` version in `vNext`.

   a. Open Github Desktop and select the docs repo

   b. Select `staging` in the **Current Branch** dropdown menu. Click **Fetch origin** to get any new changes.

   c. Click the **Create a Pull Request** button.

   d. Get the resulting PR reviewed and merged.

7. Repeat steps 5 and 6 for the [docs-generated repo](https://github.com/OpenLiberty/docs-generated/blob/vNext/antora.yml)

8. Antora natively supports automatically redirecting to the latest version by replacing the latest numerical version number in the URL with 'latest'. However, you must update the doc-redirects properties file so that if a user manually types the latest numerical version into the URL they are redirected to the `latest` symbolic version. You can create these PRs in your browser by following the same procedure that is described in steps 4c-4e.

   a. In the docs-playbook repo, update [the redirects file on `prod`](https://github.com/OpenLiberty/docs-playbook/blob/prod/doc-redirects.properties) so that `/docs/xx.0.0.x/*=/docs/latest/`, where `xx.0.0.x` is the version of the doc that is being released. 
    
   b. Then, update the same file on [`draft`](https://github.com/OpenLiberty/docs-playbook/blob/draft/doc-redirects.properties) and [`staging`](https://github.com/OpenLiberty/docs-playbook/blob/staging/doc-redirects.properties) to point to the _next_ upcoming release (the newest released version + 1). For example, if the version that is being released is `22.0.0.5`, you must define `/docs/22.0.0.5/*=/docs/latest/` in the prod version of the docs-redirects properties file and `/docs/22.0.0.6/*=/docs/latest/` in the draft and staging versions of the file. 

9. Before you proceed, make sure all of the pull requests and changes from previous steps have been merged into their respective repos (docs, docs-generated, and docs-playbook).
10. Confirm the updated content and release versions on the draft and staging sites.
   a. The PRs that you merged in step 7 & 8 triggered builds of the [`docs-draft site`](https://docs-draft-openlibertyio.mqj6zf7jocq.us-south.codeengine.appdomain.cloud/docs/latest/overview.html), [`full draft site`](https://draft-openlibertyio.mqj6zf7jocq.us-south.codeengine.appdomain.cloud/docs/latest/overview.html), [`docs-staging site`](https://docs-staging-openlibertyio.mqj6zf7jocq.us-south.codeengine.appdomain.cloud/docs/latest/overview.html), and [`full staging site`](https://staging-openlibertyio.mqj6zf7jocq.us-south.codeengine.appdomain.cloud/docs/latest/overview.html). Since the changes made in the previous steps were only to the doc specific repos, you can use the doc specific sites to validate your changes which take ~25min to redeploy (whereas the full sites take ~60min). If you have access, you can track the build progress in the [#was-ol-draft-site-alerts](https://ibm-cloud.slack.com/archives/C01G7L68KAP) and [#was-ol-staging-site-alerts](https://ibm-cloud.slack.com/archives/C01GX9P8YP2) slack channels.
   b. Once the builds complete, you can verify them on the draft and staging sites, which should now default to the _next_ upcoming version as `latest`. Look for the next upcoming version in the version-picker display at the beginning of the table of contents. When the prod site publishes, it will default to the version that is being released.

11. Finally, in the [#was-open-liberty-site](https://ibm-cloud.slack.com/archives/C4U7TQUSY) Slack channel, request a rebuild of the production openliberty.io site to publish the release. Once a trigger is added to kick off a prod site build for changes to the `prod` branch of this repo, this step will be removed.
