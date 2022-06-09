# How are the docs on openliberty.io built and what is a docs playbook?

The docs on https://openliberty.io/ are built using a tool called [Antora](https://antora.org/). It is a tool that helps streamline writing, managing, and versioning documentation.

The [docs playbook](https://github.com/OpenLiberty/docs-playbook/blob/prod/antora-playbook.yml) is a key file used by Antora see https://docs.antora.org/antora/latest/playbook/. It tells the build what branches and versions of the [docs repo](https://github.com/OpenLiberty/docs) to use. Antora aggregates all of the content specified in the playbook and creates versions of pages based on the `version` attribute in the `antora.yml` of each doc branch. See https://github.com/OpenLiberty/docs/blob/vNext/antora.yml for an example. If Antora finds the same doc in multiple doc branches, it will create a version picker on the site that lets you easily switch between versions of the doc.

There are three doc playbooks used in the builds, located in the following branches in this repo: [prod](https://github.com/OpenLiberty/docs-playbook/blob/prod/antora-playbook.yml), [staging](https://github.com/OpenLiberty/docs-playbook/blob/staging/antora-playbook.yml), and [draft](https://github.com/OpenLiberty/docs-playbook/blob/draft/antora-playbook.yml).

https://openliberty.io/ is built using the [prod playbook](https://github.com/OpenLiberty/docs-playbook/blob/prod/antora-playbook.yml). This file explicitly only contains versions of the documentation for released Open Liberty versions. This is the file that needs to be [updated each release](#publishing-a-new-release-of-open-liberty-docs) to add the new release to be published to openliberty.io. This is intentionally manual and does not use wildcards for the branches to prevent any unintended branches/content from being published to the site.

The [staging site](http://staging-openlibertyio.mybluemix.net/) is built using the [staging playbook](https://github.com/OpenLiberty/docs-playbook/blob/staging/antora-playbook.yml). This playbook contains the `staging` branch which is used to stage in changes for the next documentation version to be released. It contains a wildcard `V*.0.0.*-staging` rule which pulls in all of the branches used to stage in changes to already released versions of documentation. This branch does not need any updating each release, and shouldn't need many, if any, updates in general.

The [draft site](https://draft-openlibertyio.mybluemix.net/) is built using the [draft playbook](https://github.com/OpenLiberty/docs-playbook/blob/draft/antora-playbook.yml). This playbook contains the `draft` branch which pulls in all of the content currently being drafted. It contains a wildcard `V*.0.0.*-draft` which pulls in all of the branches used to draft changes to already released versions of documentation. This branch does not need any updating each release, and shouldn't need many, if any, updates in general.

# Publishing a new release of Open Liberty Docs

This is the flow that must be followed when releasing a new version of docs on openliberty.io. These steps update the `vNext`, `staging` and `draft` branches of the [docs repo](https://github.com/OpenLiberty/docs) so that the next version of docs can be worked on and ensures that fixes can still be made to the docs of the version being released. This procedure assumes you have already cloned the [docs](https://github.com/OpenLiberty/docs), [docs-generated](https://github.com/OpenLiberty/docs-generated), and docs-playbook repos.

The following sections provide instructions to publish an Open Liberty release from the command line. For similar instructions that use the GitHub desktop client, see [Publishing a new release of Open Liberty Docs by using GitHub desktop](https://github.com/OpenLiberty/docs-playbook/edit/draft/README.md#Publishing-a-new-release-of-Open-Liberty-Docs-by-using-GitHub-desktop)

## Updating the generated docs

The Open Liberty generated docs, which comprise the majority of the **Features** and **Server configuration** sections of the doc, are built from the [docs-generated repo](https://github.com/OpenLiberty/docs-generated). For each Open Liberty release, Chuck Bridgham from the kernel team posts the updated generated docs to the `draft` branch of the docs-generated repo. Contact Chuck several days before the release to ensure the generated docs are updated on draft in time to publish. 

Once you have verified the generated doc on the draft site, open a PR from `draft` to `staging` in the docs-generated repo and get it reviewed and merged. Once youy verify the generated doc on the `staging` site, open a PR from `staging` to `vNext` in the docs-generated repo. Get the PR reviewed and merge it in.

## Publishing the docs and preparing for the next release

1. Once all of the changes look correct on the [staging site](https://staging-openlibertyio.mybluemix.net/docs/), and all of the `staging` branch changes have been merged into the `vNext` branch in the docs and docs-generated repos, check out the `vNext` branch in the docs repo and run `git pull` to get the updates. Then, create a branch off of the `vNext` branch with the next release version number. E.g. if the most recent version on openliberty.io is `20.0.0.8`, then create a `v20.0.0.9`  branch off of `vNext` by checking out the `vNext` branch and running `git checkout -b v20.0.0.9` and `git push` the branch.

2. Create `vX.0.0.X-staging` and `vX.0.0.X-draft` branches off of the new version branch just created which will be used for retroactive fixes to this version in the future. E.g. if `v20.0.0.9` is the version being released, then from that branch run `git checkout -b v20.0.0.9-staging` and push that branch to the docs repo. Switch back to the `v20.0.0.9` branch and run `git checkout -b v20.0.0.9-draft` and push that branch to the docs repo.
   
3. Repeat steps 1 and 2 for the [docs-generated repo](https://github.com/OpenLiberty/docs-generated).

4. Create a new branch off of the `prod` branch of this repo (docs-playbook) to add the new release version to the `branches` section of the [`antora-playbook.yml`](https://github.com/OpenLiberty/docs-playbook/blob/prod/antora-playbook.yml). Make a pull request into the `prod branch`. Once the pull request has been reviewed merge it in.

5. In the docs repo, change the `version` in `antora.yml` in the [`staging`](https://github.com/OpenLiberty/docs/blob/staging/antora.yml) and [`draft`](https://github.com/OpenLiberty/docs/blob/draft/antora.yml) branches to the next release `vX.0.0.X` version incremented from the current release being published to openliberty.io. Once those changes have been reviewed merge them in. This is important so that the staging and draft builds can complete. Antora requires all of the versions of `antora.yml` of the branches being used in a build to have a unique version so they need to be updated to be unique from the version just released. You can either change the version and commit to draft/staging or create a pull request to do so.

6. Once the antora.yml [`staging`](https://github.com/OpenLiberty/docs/blob/staging/antora.yml) and [`draft`](https://github.com/OpenLiberty/docs/blob/draft/antora.yml) versions have been updated in the docs repo, create a pull request from `staging` to `vNext` to update the `antora.yml` version in `vNext`.

7. Repeat steps 5 and 6 for the [docs-generated repo](https://github.com/OpenLiberty/docs-generated/blob/vNext/antora.yml)

8. Antora natively supports automatically redirecting to the latest version by replacing the latest numerical version number in the URL with `latest`. However, you must update the doc-redirects properties file so that if a user manually types the latest numerical version into the URL they are redirected to the `latest` symbolic version.

   In the docs-playbook repo, update [the redirects file on `prod`](https://github.com/OpenLiberty/docs-playbook/blob/prod/doc-redirects.properties) so that `/docs/xx.0.0.x/*=/docs/latest/`, where `xx.0.0.x` is the version of the doc that is being released. Then, update the same file on [`draft`](https://github.com/OpenLiberty/docs-playbook/blob/draft/doc-redirects.properties) and [`staging`](https://github.com/OpenLiberty/docs-playbook/blob/staging/doc-redirects.properties) to point to the _next_ upcoming release (the newest released version + 1). For example, if the version that is being released is `22.0.0.5`, you must define `/docs/22.0.0.5/*=/docs/latest/` in the prod version of the docs-redirects properties file and `/docs/22.0.0.6/*=/docs/latest/` in the draft and staging versions of the file.

9. Before proceeding, make sure all of the pull requests and changes from previous steps have been merged into their respective repos (docs, docs-generated, and docs-playbook). The PRs that you merged in step 7 & 8 triggered builds of the [`docs-draft site`](https://docs-draft-openlibertyio.mybluemix.net/docs/), [`full draft site`](https://draft-openlibertyio.mybluemix.net/docs/), [`docs-staging site`](https://docs-staging-openlibertyio.mybluemix.net/docs/), and [`full staging site`](https://staging-openlibertyio.mybluemix.net/docs/). Since the changes made in the previous steps were only to the doc specific repos, you can use the doc specific sites to validate your changes which take ~25min to redeploy (whereas the full sites take ~60min). If you have access, you can track the build progress in the [#was-ol-draft-site-alerts](https://ibm-cloud.slack.com/archives/C01GDRYT4UC) and [#was-ol-staging-site-alerts](https://ibm-cloud.slack.com/archives/C01G7TAFR8T) slack channels. Once the builds complete, you can verify them on the draft and staging sites, which should now default to the _next_ upcoming version as `latest`. Look for the next upcoming version in the version-picker display at the beginning of the table of contents. When the prod site publishes, it will default to the version that is being released.

10. Finally, in the [#was-open-liberty-site](https://ibm-cloud.slack.com/archives/C4U7TQUSY/p1652190177756459) Slack channel, request a rebuild of the production openliberty.io site to publish the release.  Once a trigger is added to kick off a prod site build for changes to the `prod` branch of this repo, this step will be removed.

# Publishing a new release of Open Liberty Docs by using GitHub Desktop

These instructions are identical to those in the previous section except for Git processes are described from the perpsective of using the GitHub Desktop client instead of the command line. This procedure assumes you have already have GitHub Desktop and have already cloned the [docs](https://github.com/OpenLiberty/docs), [docs-generated](https://github.com/OpenLiberty/docs-generated), and docs-playbook repos.

## Updating the generated docs
The Open Liberty generated docs, which comprise the majority of the **Features** and **Server configuration** sections of the doc, are built from the [docs-generated repo](https://github.com/OpenLiberty/docs-generated). For each Open Liberty release, Chuck Bridgham from the kernel team posts the updated generated docs to the draft branch of the docs-generated repo. Contact Chuck several days before the release to ensure the generated docs are updated on draft in time to publish. 

Once you have verified the generated doc on the draft site, open a PR from `draft` to `staging` in the docs generated repo:

1. Open Github Desktop and select the docs-generated repo
2. Select `draft` in the **Current Branch** dropdown menu
3. Click the **Create a Pull Request** button.
4. In the resulting PR, click the **Edit** button and change the Base to `staging`
5. Get the resulting PR reviewed and merged.

 Once you verify the generated doc on the `staging` site, on a PR from `staging` to `vNext` in the docs-generated repo:

1. Open Github Desktop and select the docs-generated repo
2. Select `staging` in the **Current Branch** dropdown menu
3. Click the **Create a Pull Request** button.
4. Get the resulting PR reviewed and merged.

## Publishing the docs and preparing for the next release

1. Once all of the changes look correct on the [staging site](https://staging-openlibertyio.mybluemix.net/docs/), and all of the `staging` branch changes have been merged into the `vNext` branch in the docs and docs-generated repos, create the branches you need for the new release in the docs repo.

   a. Select `vNext` in the **Current Branch** menu in Github Desktop

   b. Click the **New Branch** button and create a branch with the next release version number. E.g. if the most recent version on openliberty.io is `20.0.0.8`, then create a `v20.0.0.9`  branch. 

2. Follow the same procedure in steps 1a-1b to create `vX.0.0.X-staging` and `vX.0.0.X-draft` branches off of the new version branch you just created, which will be used for retroactive fixes to this version in the future. E.g. if `v20.0.0.9` is the version being released, then select `v20.0.0.9` from the **Current Branch** menu in GitHub desktop and create a new branch called `v20.0.0.9-staging`.  Switch back to `v20.0.0.9` branch and repeat to create `v20.0.0.9-draft`. **Don't forget to publish the branch all the branches that you create by clicking "Publish this branch" after you create each branch**.
  
3. Repeat steps 1 and 2 for the [docs-generated repo](https://github.com/OpenLiberty/docs-generated).

4. Create a new branch off of the `prod` branch of this repo (docs-playbook) to add the new release version to the `branches` section of the [`antora- playbook.yml`](https://github.com/OpenLiberty/docs-playbook/blob/prod/antora-playbook.yml). You can make this change in your browser.

   a. Go to the [antora-playbook.yml](https://github.com/OpenLiberty/docs-playbook/blob/prod/antora-playbook.yml). 

   b. Click the pencil icon to edit the file. 

   c. Add the new release version to the `branches` section

   d. After you edit the file, create a pull request into the `prod` branch by clicking the **Propose changes** button. Once the pull request has been reviewed, merge it in.

5. In the docs repo, create a new vNext-based branch and change the `version` value in the `antora.yml`file to the next release `vX.0.0.X` version, incremented from the current release being published to openliberty.io. Create PRs to add this change to both the [`staging`](https://github.com/OpenLiberty/docs/blob/staging/antora.yml) and [`draft`](https://github.com/OpenLiberty/docs/blob/draft/antora.yml) branches.  For example, if `22.0.0.6` is being released, set the version to `22.0.0.7` in both `draft` and `staging`.

Once those changes have been reviewed merge them in. This is important so that the staging and draft builds can complete. Antora requires all of the versions of `antora.yml` of the branches being used in a build to have a unique version so they need to be updated to be unique from the version just released. You can either change the version and commit to draft/staging or create a pull request to do so.

6. Once the antora.yml [`staging`](https://github.com/OpenLiberty/docs/blob/staging/antora.yml) and [`draft`](https://github.com/OpenLiberty/docs/blob/draft/antora.yml) versions have been updated in the docs repo, create a pull request from `staging` to `vNext` to update the `antora.yml` version in `vNext`.

   a. Open Github Desktop and select the docs repo

   b. Select `staging` in the **Current Branch** dropdown menu

   c. Click the **Create a Pull Request** button.

   d. Get the resulting PR reviewed and merged.

7. Repeat steps 5 and 6 for the [docs-generated repo](https://github.com/OpenLiberty/docs-generated/blob/vNext/antora.yml)

8. Antora natively supports automatically redirecting to the latest version by replacing the latest numerical version number in the URL with 'latest'. However, you must update the doc-redirects properties file so that if a user manually types the latest numerical version into the URL they are redirected to the `latest` symbolic version. You can create these PRs by following the same procedure that is described in steps 4b-4d.

   a. In the docs-playbook repo, update [the redirects file on `prod`](https://github.com/OpenLiberty/docs-playbook/blob/prod/doc-redirects.properties) so that `/docs/xx.0.0.x/*=/docs/latest/`, where `xx.0.0.x` is the version of the doc that is being released. 
    
   b. Then, update the same file on [`draft`](https://github.com/OpenLiberty/docs-playbook/blob/draft/doc-redirects.properties) and [`staging`](https://github.com/OpenLiberty/docs-playbook/blob/staging/doc-redirects.properties) to point to the _next_ upcoming release (the newest released version + 1). For example, if the version that is being released is `22.0.0.5`, you must define `/docs/22.0.0.5/*=/docs/latest/` in the prod version of the docs-redirects properties file and `/docs/22.0.0.6/*=/docs/latest/` in the draft and staging versions of the file. 

9. Before proceeding, make sure all of the pull requests and changes from previous steps have been merged into their respective repos (docs, docs-generated, and docs-playbook). The PRs that you merged in step 7 & 8 triggered builds of the [`docs-draft site`](https://docs-draft-openlibertyio.mybluemix.net/docs/), [`full draft site`](https://draft-openlibertyio.mybluemix.net/docs/), [`docs-staging site`](https://docs-staging-openlibertyio.mybluemix.net/docs/), and [`full staging site`](https://staging-openlibertyio.mybluemix.net/docs/). Since the changes made in the previous steps were only to the doc specific repos, you can use the doc specific sites to validate your changes which take ~25min to redeploy (whereas the full sites take ~60min). If you have access, you can track the build progress in the [#was-ol-draft-site-alerts](https://ibm-cloud.slack.com/archives/C01GDRYT4UC) and [#was-ol-staging-site-alerts](https://ibm-cloud.slack.com/archives/C01G7TAFR8T) slack channels. Once the builds complete, you can verify them on the draft and staging sites, which should now default to the _next_ upcoming version as `latest`. Look for the next upcoming version in the version-picker display at the beginning of the table of contents. When the prod site publishes, it will default to the version that is being released.

10. Finally, in the [#was-open-liberty-site](https://ibm-cloud.slack.com/archives/C4U7TQUSY/p1652190177756459) Slack channel, request a rebuild of the production openliberty.io site to publish the release.  Once a trigger is added to kick off a prod site build for changes to the `prod` branch of this repo, this step will be removed.
