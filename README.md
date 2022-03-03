# How are the docs on openliberty.io built and what is a docs playbook?

The docs on https://openliberty.io/ are built using a tool called [Antora](https://antora.org/). It is a tool that helps streamline writing, managing, and versioning documentation.

The [docs playbook](https://github.com/OpenLiberty/docs-playbook/blob/prod/antora-playbook.yml) is a key file used by Antora see https://docs.antora.org/antora/latest/playbook/. It tells the build what branches and versions of the [docs repo](https://github.com/OpenLiberty/docs) to use. Antora aggregates all of the content specified in the playbook and creates versions of pages based on the `version` attribute in the `antora.yml` of each doc branch. See https://github.com/OpenLiberty/docs/blob/vNext/antora.yml for an example. If Antora finds the same doc in multiple doc branches, it will create a version picker on the site that lets you easily switch between versions of the doc.

There are three doc playbooks used in the builds, located in the following branches in this repo: [prod](https://github.com/OpenLiberty/docs-playbook/blob/prod/antora-playbook.yml), [staging](https://github.com/OpenLiberty/docs-playbook/blob/staging/antora-playbook.yml), and [draft](https://github.com/OpenLiberty/docs-playbook/blob/draft/antora-playbook.yml).

https://openliberty.io/ is built using the [prod playbook](https://github.com/OpenLiberty/docs-playbook/blob/prod/antora-playbook.yml). This file explicitly only contains versions of the documentation for released Open Liberty versions. This is the file that needs to be [updated each release](#publishing-a-new-release-of-open-liberty-docs) to add the new release to be published to openliberty.io. This is intentionally manual and does not use wildcards for the branches to prevent any unintended branches/content from being published to the site.

The [staging site](http://staging-openlibertyio.mybluemix.net/) is built using the [staging playbook](https://github.com/OpenLiberty/docs-playbook/blob/staging/antora-playbook.yml). This playbook contains the `staging` branch which is used to stage in changes for the next documentation version to be released. It contains a wildcard `V*.0.0.*-staging` rule which pulls in all of the branches used to stage in changes to already released versions of documentation. This branch does not need any updating each release, and shouldn't need many, if any, updates in general.

The [draft site](https://draft-openlibertyio.mybluemix.net/) is built using the [draft playbook](https://github.com/OpenLiberty/docs-playbook/blob/draft/antora-playbook.yml). This playbook contains the `draft` branch which pulls in all of the content currently being drafted. It contains a wildcard `V*.0.0.*-draft` which pulls in all of the branches used to draft changes to already released versions of documentation. This branch does not need any updating each release, and shouldn't need many, if any, updates in general.

# Publishing a new release of Open Liberty Docs

This is the flow that must be followed when releasing a new version of docs on openliberty.io. These steps update the `vNext`, `staging` and `draft` branches of the [docs repo](https://github.com/OpenLiberty/docs) so that the next version of docs can be worked on and ensures that fixes can still be made to the docs of the version being released.

Before proceeding, make sure that there are no current builds in the `Build` step of the [prod pipeline](https://cloud.ibm.com/devops/pipelines/fcc7c3e9-9c40-4a58-8a7f-09c08413ab7d?env_id=ibm:yp:us-south). This is so merging pull requests later into prod does not impact any current deployments.

1. Once all of the changes look correct on the [staging site](https://staging-openlibertyio.mybluemix.net/docs/), and all of the `staging` branch changes have been merged into the `vNext` branch, checkout the `vNext` branch and run `git pull` to get the updates. Then, create a branch off of the `vNext` branch with the next release version number. E.g. if the most recent version on openliberty.io is `20.0.0.8`, then create a `v20.0.0.9`  branch off of `vNext` by checking out the `vNext` branch and running `git checkout -b v20.0.0.9` and `git push` the branch.

2. Create `vX.0.0.X-staging` and `vX.0.0.X-draft` branches off of the new version branch just created which will be used for retroactive fixes to this version in the future. E.g. if `v20.0.0.9` is the version being released, then from that branch run `git checkout -b v20.0.0.9-staging` and push that branch to the docs repo. Switch back to the `v20.0.0.9` branch and run `git checkout -b v20.0.0.9-draft` and push that branch to the docs repo.
   
3. Repeat steps 1 and 2 for the [docs-generated repo](https://github.com/OpenLiberty/docs-generated).

4. Create a new branch off of the `prod` branch of this repo to add the new release version to the `branches` section of the [`prod playbook`](https://github.com/OpenLiberty/docs-playbook/blob/prod/antora-playbook.yml). Ensure that the branch name added matches the branch name in Github (e.g. includes a 'v' at the start). Make a pull request into the `prod branch`. Once the pull request has been reviewed merge it in.

5. Change the `version` in `antora.yml` in the [`staging`](https://github.com/OpenLiberty/docs/blob/staging/antora.yml) and [`draft`](https://github.com/OpenLiberty/docs/blob/draft/antora.yml) branches of the docs repo to the next release `vX.0.0.X` version incremented from the current release being published to openliberty.io. Once those changes have been reviewed merge them in. This is important so that the staging and draft builds can complete. Antora requires all of the versions of `antora.yml` of the branches being used in a build to have a unique version so they need to be updated to be unique from the version just released. You can either change the version and commit to draft/staging or create a pull request to do so.

6. Once the antora.yml [`staging`](https://github.com/OpenLiberty/docs/blob/staging/antora.yml) and [`draft`](https://github.com/OpenLiberty/docs/blob/draft/antora.yml) versions have been updated in the docs repo, create a pull request from `staging` to `vNext` to update the `antora.yml` version in `vNext`.

7. Repeat steps 5 and 6 for the [docs-generated repo](https://github.com/OpenLiberty/docs-generated/blob/vNext/antora.yml)

8. With the latest update of antora version (antora 3.0.1) and by adding the following parameter latest_version_segment: latest in antora-playbook.yml the latest numerical version will be changed to symbolic version in the URL.

9. (Temporary for now). Create a branch off of the `prod branch` of the [openliberty.io repo](https://github.com/OpenLiberty/openliberty.io). Make the following updates all in the same branch and merge them into `draft` and `staging`:
  
   * Update the version of `{replace_version page.url 'X.0.0.X' }` in [src/main/content/antora_ui/src/partials/head-info.hbs](https://github.com/OpenLiberty/openliberty.io/blob/staging/src/main/content/antora_ui/src/partials/head-info.hbs) so that the canonical meta tag is set to the latest version of the documentation for all versions of the docs. 
   * Update the version of the url `<loc>https://openliberty.io/docs/X.0.0.X/overview.html</loc>` in the [src/main/content/sitemap.xml](https://github.com/OpenLiberty/openliberty.io/blob/staging/src/main/content/sitemap.xml).

10. Before proceeding, make sure all of the pull requests and changes from previous steps have been merged into their respective repos. Make a pull request from your branch into both the `draft` and `staging` branches. Once the pull requests have been reviewed merge them in. After each pull request has been merged in, the [draft site](https://draft-openlibertyio.mybluemix.net/) and [staging site](https://staging-openlibertyio.mybluemix.net/) should automatically rebuild and redeploy (takes ~60 minutes). If you have access to the IBM Cloud pipelines, you can track the progress through [IBM Cloud draft pipeline](https://cloud.ibm.com/devops/pipelines/3f4abe1b-0c51-4752-ac48-5ed9030f530f?env_id=ibm:yp:us-south) and [IBM Cloud staging pipeline](https://cloud.ibm.com/devops/pipelines/61002d76-1c13-4502-a109-3b794b3f2350?env_id=ibm:yp:us-south). Test that the docs link works and redirects to the newest doc version once the site is done building.

11. In the openliberty.io repo, [open a pull request from the `staging` branch into the `prod` branch](https://github.com/OpenLiberty/openliberty.io/compare/staging?expand=1). The [openliberty.io site](https://openliberty.io/) will automatically rebuild when the pull request is merged in. You can track the progress using the [IBM Cloud pipeline](https://cloud.ibm.com/devops/pipelines/063d397c-febc-4f73-8340-61da6bc775f5?env_id=ibm:yp:us-south).

