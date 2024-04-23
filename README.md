# action: Release Candidate
This repo contains an action to create a release tag in frontend repositories


----
Dynamic props that action supports:
```
inputs:
  repo-link:  # id of input
    description: 'what is the link for the repo: ie. https://api.github.com/repos/tajawal/almosafer-business/'
    required: true
 
  node-version:  # id of input
    description: 'which node version to use?'
    required: false
    default: '16.14'

```


----

Usage:

```
on:
  pull_request:
    types: [closed]
    branches: [develop]

jobs:
  # job 1
  check:
    runs-on: CT-selfhosted
    steps:
      - name: Exit gracefully if not a release
        id: releaseExitStep
        run: |
          echo "::set-output name=should_release::true"
          if [[ ${{ contains(github.event.pull_request.labels.*.name, 'norelease') }} = "true" ]]; then
            echo "::set-output name=should_release::false"
          fi
    outputs:
      status: ${{steps.releaseExitStep.outputs.should_release}}

  # job 2
  release:
    name: Release Canditate
    runs-on: CT-selfhosted
    needs: [check]
    if: github.event.pull_request.merged == true && needs.check.outputs.status == 'true'
    steps:
      - uses: actions/checkout@v2
      - id: foo
        uses: tajawal/action-rc@1.0.0
        with:
          repo-link: 'https://api.github.com/repos/tajawal/almosafer-business/'
          node-version: '12.x'
```


----

To update the tag, run:
```
git tag -a -m "Description of this release" 1.0.2
git push --follow-tags
```


----

Prerequisites to run the action:
- `{{SEERA_FRONTEND_TOKEN}}`  should be set up in the repo/org
- have following scripts in your package.json:
```
   "release:major": "release-it major",
    "release:patch": "release-it patch",
    "release:minor": "release-it minor",
    "release:rc": "release-it $RELEASE_TYPE --preRelease=RC"
```
- have release-it.json file in the root of your app:
 ```
  {
  "preReleaseId": null,
  "pkgFiles": ["package.json"],
  "use": "pkg.version",
  "scripts": {
    "beforeStart": null,
    "beforeBump": null,
    "afterBump": null,
    "beforeStage": null,
    "changelog": "git log --pretty=format:\"* %s (%h)\" [REV_RANGE]",
    "afterRelease": null
  },
  "git": {
    "requireCleanWorkingDir": true,
    "requireUpstream": false,
    "addUntrackedFiles": false,
    "commit": true,
    "commitMessage": "Release ${version}",
    "commitArgs": "",
    "tag": true,
    "tagName": "v${version}",
    "tagAnnotation": "Release ${version}",
    "tagArgs": "",
    "push": true,
    "pushArgs": "",
    "pushRepo": "origin"
  },
  "npm": {
    "publish": false
  }
}
```
