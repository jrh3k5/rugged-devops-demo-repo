# DN6 Simple Web with Auth and Insights

This is a simple template with Auth and Insights for DotNet 6 MVC.  I've also added app configuration and code to integrate Azure App Config and Azure Key Vault.  If you try to use those, make sure to read all the notes in the Program.cs file.  It would also help if you've seen the video from the Opsgility Azure A-Z conference.

## Use at your own risk

We are not responsible for anything you do with this code.

## Set Up the solution

You will need to add your Application insights at Azure and you will need to wire up the Instrumentation connection string.

Additionally, you will need SQL Developer locally, then you will need to do some configuration and create and wire up the SQLAzure at Azure.

## Notes

Additional Thoughts

### YAML

Use the sample YAML at your discretion.

You will want to update your user secret to map correctly for your publish profile and you will want to update the name of your azure web app

### Developer Secrets

As you develop the solution, do not store your connection strings in the appsettings.json file.

Instead, put them in the user secrets.

You will likely want a separate instance of application insights for your developer from your web/slot in production in the real world.  You could use the same one for demo/learning purposes.

## IMPORTANT STEPS

Issues with the new OIDC Federated Identity are plenty, but I've solved them so you don't have to

To get GitHubActions to work against your Azure with ability to run templates:

1. Create a service principal:

    First, use the portal or the CLI to create a service principal/app registration.

    You must give it rights so its typically *eaiser* to just use the az cli in my opinion

    ```bash
    rg=<your-rg-name-here>
    sub=<your-sub-id-here>
    az ad sp create-for-rbac --name <any-app-name-here> --role contributor --scopes /subscriptions/$sub/resourcegroups/$rg 
    ```  

    When the output is done, grab the following info:
    - appId
    - tenant

    Ultimately, you'll also need your subId in a minute as well

1. Open the app registration, add two federated credentials

    You'll need two credentials - one on the main branch for general login and one on pull request if you are going to trigger the action from a pull request and need to be able to affect azure

    [Review the slide deck for more information]

1. Add credentials into Github Secrets for Actions

    - AZURE_CLIENT_ID : your app id generated above
    - AZURE_SUBSCRIPTION_ID : your sub id
    - AZURE_TENANT_ID : easily found in the portal but also was output in the generation above

2. Start small

    Run a quick GitHub to Azure action to ensure you can log in

    ```bash
    on:
    push:
        branches:
        - main
    workflow_dispatch:

    permissions:
        id-token: write
        contents: read
    jobs: 
    build-and-deploy:
        runs-on: ubuntu-latest
        steps:
        - name: 'Az CLI login'
            uses: azure/login@v1
            with:
            client-id: ${{ secrets.AZURE_CLIENT_ID }}
            tenant-id: ${{ secrets.AZURE_TENANT_ID }}
            subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
    
        - name: 'Run az commands'
            run: |
            az account show
            az group list
            pwd 
    ```  

    [Get the Yaml from my GIST](https://gist.github.com/blgorman/ada77d3c50c120323044d6f12e520a74)

1. Check the subject

    The error is almost always in the subject not being set correctly.  Review the error and find the subject being requested, then go add that as a valid requestor on your app registration