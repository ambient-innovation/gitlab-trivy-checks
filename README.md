This repository adds a job to your GitLab-CI to visualize the findings from Dependency(Security) and License Scanning as a CodeQuality Report widget in your merge requests.

[Dependency Scanning](https://github.com/ambient-innovation/gitlab-trivy-security-checks) is often considered part of Software Composition Analysis (SCA). SCA can contain aspects of inspecting the items your code uses. These items typically include application and system dependencies that are almost always imported from external sources, rather than sourced from items you wrote yourself.

[License Scanning](https://github.com/ambient-innovation/gitlab-trivy-license-checks) is often considered part of Compliance rules. Open source software licenses define how you can use, modify and distribute the open source software. Thus, when selecting an open source package to merge to your code it is imperative to understand the types of licenses and the user restrictions the package falls under, which helps you mitigate any compliance issues.

[Config Scanning](https://github.com/ambient-innovation/gitlab-trivy-config-checks) could be considered part of a wider definition of SCA that inlcudes Infrastructure as Code to be part of Software. The config checks try to find common misconfigurations that pose security risks to your infrastructure - such as running services as the root account or storing secrets in versioned config files.


This config template can be included in your .gitlab-ci.yml to get the above scanning jobs and the result visualisation for free.

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
  - deploy
```

The **test** stage has to come after the docker image has already been built and pushed to the registry or the scanner will not work.

Last but not least you need one job within that test stage going by the name `license_scanning`, one with the name `container_scanning` and one with the name `config_scanning`. A minimal config to check your backend container dependencies for vulnerabilities and license issues as well as the `helmcharts` directory for config issues looks like this:  
```yaml
license_scanning:
  variables:
    IMAGE: $IMAGE_TAG_BACKEND
container_scanning:
  variables:
    IMAGE: $IMAGE_TAG_BACKEND
config_scanning:
  variables:
    DIRECTORY: helmcharts
```

The example shown here will overwrite the `scanning` jobs from the template and tell them to

a) scan an image as specified in the `IMAGE_TAG_BACKEND` variable,\
b) perform a simple license scan\
c) only report errors with a level of HIGH,CRITICAL or UNKNOWN. 

**Note:** If you wish to run the `*_scanning` jobs in another stage than "`test`" (as they do by default) simply copy the above code to your .gitlab-ci.yml file and add the keyword `stage` with your custom stage name.

Example for minimal stage-overwrite setup:

```yaml
license_scanning:
  stage: my-custom-stage
```

# Errors in Pipeline
The `check trivy scan results` Job is built to be secure by default and will cause the job to fail in your pipeline if it finds any issues in either security or license scanning. The Job exits with a status code resembling the number of issues found.  
Sometimes it's unavoidable to have security/license issues in your project, in those cases, you can decide to allow the job to fail by overriding the `check trivy scan results` job in your gitlab-ci.yml and enabling the `allow_failure` option.  
Example:  
```yaml
check trivy scan results:
  allow_failure: true
```

# More config options
You can configure each of the scanner jobs with a ton of options not mentioned here.   
Please check the corresponding repositories and the trivy documentation for more config options.
* [Container Scanning](https://github.com/ambient-innovation/gitlab-trivy-security-checks)
* [License Scanning](https://github.com/ambient-innovation/gitlab-trivy-license-checks)
* [Config Scanning](https://github.com/ambient-innovation/gitlab-trivy-config-checks)
* [Trivy Documentation](https://aquasecurity.github.io/trivy/)
