---
tags:
  - umbraco
  - azure
  - azure-pipelines
  - azure-app-service
---

# Azure DevOps Web App Deployment and Umbraco

When using the AzureRmWebAppDeployment@4 task in yaml pipelines, the default is to run from a zip file, which sets `WEBSITE_RUN_FROM_PACKAGE=1`. This is fine for most cases, but [Umbraco sites don't work on a read only filesystem](https://docs.umbraco.com/umbraco-cms/fundamentals/setup/server-setup/azure-web-apps#issues-with-read-only-filesystems).

## The secret setting

The pipelines [Azure Web App Deployment Task](https://learn.microsoft.com/en-us/azure/devops/pipelines/tasks/reference/azure-rm-web-app-deployment-v4?view=azure-pipelines) input value `enableCustomDeployment: true` will stop pipelines from auto-detecting the best deployment method and use 'webDeploy'.

This will unzip the files into App Service storage, and leave `WEBSITE_RUN_FROM_PACKAGE` unset, giving Umbraco write access to the filesystem, and allowing it to run.
