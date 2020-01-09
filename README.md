# Design Systems in Framer X

Working with design systems in Framer is what sets the tool apart from others. However, getting your components into the tool, and maintaining them for others to use sometimes requires a bit of understanding.

This repository aims to illustrate how all of these pieces work togther to provide an automated workflow, using a real design system called [Carbon](https://www.carbondesignsystem.com/).

### Table of contents

[Overview](#overview)

[Workflow](#workflow)

[Getting started](#getting-started)

## Overview

There are a few tools that have been developed to help maintain a design system in Framer, and understanding how all of them work indiviudally can be helpful. While you can use any or all of these tools in isolation, the rest of this document should help you understand how they can work together.

### [Component Importer](https://www.npmjs.com/package/@framerjs/component-importer)

The Component Importer is an integral piece of keeping a Design System in sync with Framer.

It makes an analysis of a component library and outputs a list of code components that wrap the underlying components from the library and generates a list of property controls which are determined based on the properties of the components (defined using Flow/TypeScript/PropTypes).

On every subsequent import, the component importer will merge any new props with existing ones. If properties are deemed not to be relevant within Framer (i.e. accessibility properties/classNames), they can be removed and ignored in importer.config.json generated by the component importer.

### [Color Importer](https://github.com/tisho/framer-shared-colors-cli)

The Shared Color Importer is another _optional_ tool that will import all color tokens from a Design System into [Framer Shared Colors](https://www.framer.com/support/using-framer-x/shared-colors/). These colors can also be shipped through Framer Packages. See the docs for more information on using the colors importer on it's own.

### [Icon Generator](https://github.com/iKettles/framer-svg-component-generator)

The Icon generator is a piece that allows you to import a folder of `svg` icons into Framer. It will generate a simple Icon Component that lets you easily toggle between icons on the Framer canvas.

### [Yarn Package Differ](https://www.npmjs.com/package/yarn-pkg-version-diff)

The Yarn Package Differ is the seccond tool that is used within this workflow.

Set to run on a scheduled basis, it runs a check against a design system, and returns a boolean to determine whether there have been changes or not within the design system.

### [Framer CLI](https://www.npmjs.com/package/framer-cli)

The Framer CLI is the last tool in this workflow.

It is responsible for publishing and delivering updates to the wider team through Framer Packages. Through this, users on your team can directly install design system components into a Framer project, and recieve updates as they occur.

### [Automation](https://circleci.com/)

All of these tools have been packaged togehter with some additional logic to create a fully automated process. See the Worflow section to learn how different situations play out.

Automation is handled through [CircleCI Orbs](https://circleci.com/orbs/). There are configuration options available, that make this setup truly flexible for using this in different setups.

## Workflow

### Import Components

The first step that occurs in the workflow is the importing of components. This is automatically run when the project is connected with circleCI, but can additionally run on it's own through the Component Importer

**Case**: Initial Import

The Component Importer will run, create the initial imported components under `/code` and the `importer.config.json` file.

### Determine Dependency Updates

To update the Framer projects autonomously when component libraries have been updated, we use a scheduled job and a small CLI to determine if any of the “targeted” dependencies have been updated. As certain Framer projects may rely on multiple component libraries you are able to provide a comma-separated list of NPM package names which are then taken into account when calculating a diff between the current `yarn.lock` file and the new one after running `yarn`.

**Case**: Dependencies have been updated

If the process determines that some dependencies have been updated, we move onto importing components. After the new updates have been added to the system through the component importer, components are committed & pushed to a branch called `framer-bridge/component-importer`.

**Case**: No dependencies have been updated

If no dependencies have been updated the rest of the CI steps are skipped.

### Create Pull Request

**Case**: Component importer finds and merges in new changes from the design system

After component updates havve been identified, and the component importer has added these updates, a PR is made. This PR contains commits that include updated props, new comonents, or anything else that had been identified and added.

## Getting Started

To get started, you'll need to make sure you have a few things first:

**User**:

- Github account
- CircleCI account

**Project**:

- A folder backed [`.framerfx`](https://www.framer.com/support/using-framer-x/folder-backed-projects/) project.
- A `.circleci/config.yml` [file](https://github.com/iKettles/carbon-design-system.framerfx/blob/master/.circleci/config.yml).
- `git` initialized in the project.
- `dependencies` from the Design System installed in the project.

Inspect the structure of this project to get a good idea of what your setup should look like. You can also view the [Pull Request](https://github.com/iKettles/carbon-design-system.framerfx/pull/5) to see what the component importer generates for the components.

### Step by step:

1. Open a new Framer X project, and save it as a [folder backed project](https://www.framer.com/support/using-framer-x/folder-backed-projects/).
1. Open your newly created `.framerfx` file in a code editor.
1. Open a terminal in this directory, and [add the dependencies](https://www.framer.com/learn/lesson/install-packages-with-yarn/) to your design system. (i.e. `yarn add carbon-components carbon-components-react @carbon/icons-react`).
1. intialize `git` and add your project. (i.e. `git init` / `git add .` / `git commit -m "initial commit"`).
1. Add a folder in this directory called `.circleci`, and a file in this new folder called `config.yml`.
1. Copy and paste this [`config.yml`](https://github.com/iKettles/carbon-design-system.framerfx/blob/master/.circleci/config.yml) into your project.
1. Add parameters to customize the CI to your setup. See the table at the bottom to see examples of accepted parameters. **All parameters can also optionally be set as environment variables.** See [here](#using-environment-variables-as-parameters) for examples of using environment variables for parameters.
1. Create a new repository on Github, and add the remote origin to your project (i.e. `git remote add origin git@github.com:iKettles/carbon-design-system.framerfx.git`). We reccomend keeping the name of the Github repository the same as the file name. (i.e. `carbon-design-system.framerfx`). See [this video](https://www.framer.com/learn/lesson/use-git-with-framer/) for an example.
1. Open this file in Framer X, and[ publish this file](https://www.framer.com/support/using-framer-x/publishing-packages/) to Framer Packages. (There won't be any components at this point, but we'll come back to this.) Make sure you give this package a name, description, and artwork.
1. Push your project from your local machine to Github. (i.e. `git push -u origin master`).
1. Connect your Github repository to your CircleCI account, and select "Start Building". (The initial build will fail).
1. Under CircleCI project settings, add in the necessary environment variables. See the [table](#circleci-environment-variables) below to see which varibales are required, and details steps on where to find each one.
1. Re-run the CI workflow

After successful completion of these steps, a PR should be made to the repository that contains all of the imported components. After merging this PR to amster, the components will automatically be added to Framer Packages, completing the cycle.

### CircleCI Environment Variables

| Variable                      | Description                                                                                                                                                                                                                                                                      | Example                                  |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------- |
| `FRAMER_TOKEN`                | A token that verifies the the account the publish and build command will run under. You can generate a `FRAMER_TOKEN` by using the `authenticate` command from the [`framer-cli`](https://www.npmjs.com/package/framer-cli)                                                      | `f063k553-137c-4185-bca2-23510a784dbld8` |
| `GITHUB_TOKEN`                | Read / Write access so CircleCI can create Pull Requests when new changes are found in the Design System repository. This can be generated on Github under `User > Settings > Developer Settings > Personal access tokens > Generate New Token` Make sure to select repo access. | `d8x9f3d9k2x8m1s2t6`                     |
| `CI_GIT_USER_KEY_FINGERPRINT` | A fingerprint to be identified with when using CircleCI. This can be generated through CircleCI under `Project > Settings > Permissions > Checkout SSH Keys`.                                                                                                                    | `97:b9:11:7b:06:3f:3d:ec:10:ba`          |

### CircleCI Parameters

| Parameter                   | Description                                                                                                                                     | Example                                                         | Type     |
| --------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------- | -------- |
| `component-library-package` | The name of the package to be imported. This should be the package where the components live.                                                   | "carbon-components-react"                                       | `string` |
| `dependant-libraries`       | A comma-separated list of values that will be checked to see if changes occurred.                                                               | "carbon-components-react,carbon-components,@carbon/icons-react" | `string` |
| `framer-project-path`       | The path to the Framer Project. <br><br>Set this to `./` if within the Framer Project itself.                                                   | "./"                                                            | `string` |
| `color-token-path`          | The path to your color token json file. See [here](https://github.com/tisho/framer-shared-colors-cli) for formatting information.               | "./carbon-components-react/utils/colors.json"                   | `string` |
| `icon-path`                 | The path to a folder of SVG icons. See [here](https://github.com/iKettles/framer-svg-component-generator) for formatting and usage information. | "./carbon-components-react/utils/icons"                         | `string` |

### Using Environment Variables as Parameters

If you'd like to stick to only using Environment variables, you'll need to set this in your `config.yml` file, and make sure you add them to your CircleCI Settings.

<!-- prettier-ignore -->
```yml
...

- framer/component-import:
    component-library-package: $COMPONENT_LIBRARY_PACKAGE
    dependant-libraries: $DEPENDANT_LIBRARIES 
    framer-project-path: $FRAMER_PROJECT_PATH
    color-token-path: $COLOR_TOKEN_PATH
    icon-path: $ICON_PATH

...

- framer/build:
    framer-project-path: $FRAMER_PROJECT_PATH

...

- framer/publish:
    framer-project-path: $FRAMER_PROJECT_PATH

...
```
