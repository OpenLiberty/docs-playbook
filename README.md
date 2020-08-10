# Publishing a new release of Open Liberty Docs

This is the flow that must be followed when releasing a new version of docs on openliberty.io. These steps also update the `vNext`, `staging` and `draft` branches so that the next version of docs can be worked on and ensures that fixes can still be made to the docs of the version being released.

1. Once all of the changes look correct on the [staging site](https://staging-openlibertyio.mybluemix.net/docs/), and all of the `staging` branch changes have been merged into the `vNext` branch, then create a branch off of the `vNext` branch with the next release version number. E.g. if the most recent version on openliberty.io is `20.0.0.8`, then create a `v20.0.0.9`  branch off of `vNext` by checking out the `vNext` branch and running `git checkout -b v20.0.0.9`.

2. Create `vX.0.0.X-staging` and `vX.0.0.X-draft` branches off of the version branch just released which will be used for retroactive fixes to this version in the future. E.g. if `v20.0.0.9` is the version being released, then from that branch run `git checkout -b v20.0.0.9-staging` and push that branch to the docs repo. Switch back to the `v20.0.0.9` branch and run `git checkout -b v20.0.0.9-draft` and push that branch to the docs repo.

3. Add this new release version to the `branches` section of `antora-playbook.yml`(https://github.com/OpenLiberty/docs-playbook/blob/prod/antora-playbook.yml) in the `prod` branch of this repo.

4. Create a pull request to change the `version` in `antora.yml` in the `staging` and `draft` branches to the next release `vX.0.0.X` version incremented from the current release being published to openliberty.io. This is important so that the staging and draft builds can complete. Antora requires all of the versions of `antora.yml` of the branches being used in a build to have a unique version so they need to be updated to be unique from the version just released.

5. Once the `staging` and `draft` `antora.yml` versions have been updated, create a pull request from `staging` to `vNext` to update the `antora.yml` version in `vNext`.

6. Rebuild the openliberty.io site using the IBM Cloud pipeline(https://cloud.ibm.com/devops/pipelines/fcc7c3e9-9c40-4a58-8a7f-09c08413ab7d?env_id=ibm:yp:us-south).