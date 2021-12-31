# openscad-actions

There are two actions:

  - **build**: Builds [OpenSCAD](https://openscad.org) models and images from a git repository using make.
  - **update-media-artifacts**: Checks-in the artifacts (STLs, images, etc..) to an
    orphan "media" branch of a git repository.

The **build** action is straight forward with the only real "trick" being an option the use
of Xvfb. An X server is necessary for OpenSCAD to capture images. So, roughly, the **build**
action is:

  - *Checkout source* &rarr;
  - *Install OpenSCAD and optionally Xvfb* &rarr;
  - *Build with make (targets) optionally under Xvfb (xvfb-run)*

The **update-media-artifacts** is built on the idea of using an orphan branch of a repository
to store images and other built artifacts (described [here](https://medium.com/@minamimunakata/how-to-store-images-for-use-in-readme-md-on-github-9fb54256e951)).
In addition to the separate branch, the action allows the built artifacts to be
moved into a separate subdirectory. So the **update-media-artifacts** action is:

  - *Switch to media branch, leaving built artifacts (i.e. checkout with no clean)* &rarr;
  - *Remove everything except built artifacts matching specified extensions* &rarr;
  - *Remove everything any addition artifacts (optional)* &rarr;
  - *Move all artifacts to a separate subdirectory (optional)* &rarr;
  - *Checkin the artifacts*

## Examples

Combine the two actions to update media icons, images, and stl. This uses the target
option of the **build** action to make special targets and then uses the **update-media-artifacts**
action to check-in those files:

```yaml
  name: 'Update Media'

  on:
    workflow_dispatch: 

  jobs:
    build:
      runs-on: ubuntu-latest
      - name: Build Media
        uses: swthemathwiz/openscad-actions/build@v1
        with:
          target: 'all icons images'
          use_xvfb: true
      - name: Update Media
        uses: swthemathwiz/openscad-actions/update-media-artifacts@v1
        with:
          message: 'Update media'
          files: 'png stl'
          media_branch: 'media'
          media_subdirectory: 'media'
```

Combine a version tag/push trigger with the **build** action to create a github
release (using action [softprops/action-gh-release](https://github.com/softprops/action-gh-release))
that includes the STL files: 

```yaml
  name: 'Release with STLs'

  on:
    push:
      tags:
        - "v*"

  jobs:
    build:
      runs-on: ubuntu-latest
      steps:
      - name: Build
        uses: swthemathwiz/openscad-action/build@v1
      - name: Release
        uses: softprops/action-gh-release@v1
        with:
          body: New Release
          files: '*.stl'
          generate_release_notes: true
```

## Parameters

The parameters of the **build** action are:

| Name                      | Type    | Default        | Description                                                    |
| ------------------------- | ------- | -------------- | -------------------------------------------------------------- |
| targets                   | string  | 'all'          | Build target(s) used for the 'make' command                    |
| check\_targets            | boolean | false          | Check if the _targets_ string is empty and fail if so          |
| use\_xvfb                 | boolean | false          | Run the build under Xvfb                                       |

The parameters of the **update-media-artifacts** action are:

| Name                        | Type    | Default                       | Description                                                 |
| --------------------------- | ------- | ----------------------------- | ----------------------------------------------------------- |
| message                     | string  | 'Update media'                | Message used when committing artifacts                      |
| files                       | string  | 'png stl off amf 3mf dxf svg' | Extensions of files to be committed                         |
| files\_except               | string  | ''                            | Files (or glob) to be removed after build and before commit |
| media\_branch               | string  | 'media'                       | Orphan branch name (as described in the article)            |
| media\_subdirectory         | string  | 'media'                       | Subdirectory to which all files are moved                   |
| media\_create\_subdirectory | boolean | true                          | Create the media subdirectory if it doesn't exist           |
| dry\_run                    | boolean | false                         | Do everything but use --dry-run on the final push           |
