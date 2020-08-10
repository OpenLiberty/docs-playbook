# Publishing a new release of Open Liberty Docs

This is the flow that must be followed when releasing a new version of docs on openliberty.io. These steps also update the `vNext`, `staging` and `draft` branches so that the next version of docs can be worked on and ensures that fixes can still be made to the docs of the version being released.

1. Once all of the changes look correct on the [staging site](https://staging-openlibertyio.mybluemix.net/docs/), and all of the `staging` branch changes have been merged into the `vNext` branch, then create a branch off of the `vNext` branch with the next release version number.

2. Rebuild the openliberty.io site using the IBM Cloud pipeline.

3. Change the `version` in `antora.yml` in the `staging` and `draft` branches to the next release `vX.0.0.X` version incremented from the current release being published to openliberty.io. This is important so that the staging and draft builds can complete. Antora requires all of the versions of `antora.yml` of the branches being used in a build to have a unique version so they need to be updated to be unique from the version just released.

4. Create `vX.0.0.X-staging` and `vX.0.0.X-draft` branches off of the version branch just released which will be used for retroactive fixes to this version in the future.

5. Create a pull request from `staging` to `vNext` to update the `antora.yml` version in `vNext`.