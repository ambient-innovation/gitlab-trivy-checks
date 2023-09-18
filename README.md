This repository adds a job to your GitLab-CI to visualize the findings from Dependency(Security) and License Scanning as a CodeQuality Report widget in your merge requests.

Dependency Scanning is often considered part of Software Composition Analysis (SCA). SCA can contain aspects of inspecting the items your code uses. These items typically include application and system dependencies that are almost always imported from external sources, rather than sourced from items you wrote yourself.

License Scanning is often considered part of Software Composition Analysis (SCA). SCA can contain aspects of inspecting the items your code uses. Open source software licenses define how you can use, modify and distribute the open source software. Thus, when selecting an open source package to merge to your code it is imperative to understand the types of licenses and the user restrictions the package falls under, which helps you mitigate any compliance issues.

This config template can be included in your .gitlab-ci.yml to get both scanning jobs and the result visualisation for free.

# Setup Instructions
At the very top of your .gitlab-ci.yml either add or expand the include: section so it looks similar to this:  
```yaml
include:
  - remote: https://raw.githubusercontent.com/ambient-innovation/gitlab-trivy-checks/main/gitlab-trivy-checks.yaml
  # There might be more includes here, usually starting with template like the following:
  # - template: 'Workflows/Branch-Pipelines.gitlab-ci.yml'
```

You will also need to have at least one stage called test in your top-level stages config for the default configuration:  
```yaml
stages:
  - prebuild
  - build
  - test
  - posttest
  - deploy
```

The **test** stage has to come after the docker image has already been built and pushed to the registry or the scanner will not work.

Last but not least you need one job within that test stage going by the name `license_scanning` and one with the name `container_scanning`. A minimal config looks like this:  
```yaml
license_scanning:
  variables:
    IMAGE: $IMAGE_TAG_BACKEND
container_scanning:
  variables:
    IMAGE: $IMAGE_TAG_BACKEND    
```

The example shown here will overwrite the `scanning` jobs from the template and tell them to

a) scan an image as specified in the `IMAGE_TAG_BACKEND` variable,\
b) perform a simple license scan\
c) only report errors with a level of HIGH,CRITICAL or UNKNOWN. 

**Note:** If you wish to run the `license_scanning` job in another stage than "`test`" (as it does by default) simply copy the above code to your .gitlab-ci.yml file and add the keyword `stage` with your custom stage name.

Example for minimal stage-overwrite setup:

```yaml
license_scanning:
  stage: my-custom-stage
```
