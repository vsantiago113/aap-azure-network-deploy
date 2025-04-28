# A Step by step guide on how to setup AAP to work with Azure

## Requirements

    - AAP 2.5 up and running
    - Azure subscription
    - RHEL9 Server

## Create a service principal

1. Go to [Azure Portal](https://portal.azure.com/#home)
    1. Go to [Active Directory](https://portal.azure.com/#view/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/~/Overview)
        1. Click on [App registrations](https://portal.azure.com/#view/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/~/RegisteredApps)
        1. Click on [New registration](https://portal.azure.com/#view/Microsoft_AAD_RegisteredApps/CreateApplicationBlade/quickStartType~/null/isMSAApp~/false)
        1. Provide a name then click `Register`
        1. Click on `Certificates & secrets`
        1. Click on `New client secret`
        1. Enter a description then click `Add`
        1. Store the `Value` secure somewhere, this is the value of your secret
            1. Write this down for next step. This is your `Secret ID`
1. Provice access to the service principal
    1. Go to [Subscriptions](https://portal.azure.com/#view/Microsoft_Azure_Billing/SubscriptionsBladeV2)
    1. Click on your subscription from the list
    1. Click on `Access control (IAM)`
    1. Click `Add` then select `Add role assignment`
    1. Select the `Privileged administration roles` tab
    1. Select `Contributor` then click `Next`
    1. Check the box for `User, group, or service principal`
    1. Click `+ Select members`
    1. On the search under `Select members` search for the name of your service principal and select it then click `Select`
    1. Click `Next` then click `Review + assign`

## Create the environment variables for Ansible to authenticate with Azure and ServiceNow

### ServiceNow

1. Go to [ServiceNow Developer Portal](https://developer.servicenow.com/dev.do#!/)
    1. On the upper right corner click on `Request Instance`
    1. Select the location
    1. Click `Request`
    1. When the instance is ready you will see a popup title `Your instance is ready!`
    1. From the body of the popup write down the `Username` and `Current Password` to use it later.
    1. Click `Open Instance`
    1. Sign in
    1. This is your ServiceNow instance

### Azure

1. Go to [Azure Portal](https://portal.azure.com/#home)
    1. Go to [Active Directory](https://portal.azure.com/#view/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/~/Overview)
        1. Click on [App registrations](https://portal.azure.com/#view/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/~/RegisteredApps)
        1. Click on the app you created
        1. Write down the `Application (client) ID`
        1. Write down the `Directory (tenant) ID`
    1. Go to [Subscriptions](https://portal.azure.com/#view/Microsoft_Azure_Billing/SubscriptionsBladeV2)
        1. Write down the `Subscription ID` from your Subscriptions
    1. Find the secret value from when you created the service principal

### Create environment variables for Azure and ServiceNow for Test server

    1. Edit the .bashrc file
        ```bash
        vi ~/.bashrc
    1. Scroll all the way to the bottom and pasted the following environment variables. Replace "REDACTED" with the values from the ServiceNow and Azure above.
        ```
        ```text
            export AZURE_SUBSCRIPTION_ID="REDACTED"
            export AZURE_CLIENT_ID="REDACTED"
            export AZURE_SECRET="REDACTED"
            export AZURE_TENANT="REDACTED"

            export SN_HOST="REDACTED"
            export SN_USERNAME="REDACTED"
            export SN_PASSWORD="REDACTED"
        ```

### Create the credential type for ServiceNow in AAP

1. Go to [Using modules from Automation Controller](https://galaxy.ansible.com/ui/repo/published/servicenow/itsm/docs/general_usage_patterns/)
1. Go to your Ansible Automation Platform instance and Login
    1. Go to `Automation Execution` -> `Infrastructure` -> `Credential Types`
    1. Click `Create credential type`
    1. Give it a name, in our example the name is `ServiceNow-ITSM`
    1. Under `Input configuration` paste the content below
    ```yaml
    fields:
      - id: SN_HOST
        type: string
        label: Snow Instance
      - id: SN_USERNAME
        type: string
        label: Username
      - id: SN_PASSWORD
        type: string
        label: Password
        secret: true
    required:
      - SN_HOST
      - SN_USERNAME
      - SN_PASSWORD
    ```
    1. Under `Injector configuration` paste the content below
    ```yaml
    env:
      SN_HOST: '{{ SN_HOST }}'
      SN_PASSWORD: '{{ SN_PASSWORD }}'
      SN_USERNAME: '{{ SN_USERNAME }}'
    ```
    1. Click `Create credential type`

### Create the credential for Azure
1. Go to your Ansible Automation Platform instance and Login
    1. Go to `Automation Execution` -> `Infrastructure` -> `Credentials`
    1. Click `Create credential`
    1. Enter the credential name
    1. Select your organization
    1. Select the credential type `Microsoft Azure Resource Manager`
    1. Fill up the `Type Details` with the values you created earlier.
    1. Click on `Create credential`

### Create the credential for ServiceNow
1. Go to your Ansible Automation Platform instance and Login
    1. Go to `Automation Execution` -> `Infrastructure` -> `Credentials`
    1. Click `Create credential`
    1. Enter the credential name
    1. Select your organization
    1. Select the credential type `ServiceNow-ITSM` which is the example name we used
    1. Fill up the `Type Details` with the values you created earlier.
    1. Click on `Create credential`

## Create the Execution Environment

1. Create a file named `execution-environment.yml`
    1. To get the latest requirements.txt file go to [requirements.txt](https://github.com/ansible-collections/azure/blob/dev/requirements.txt)
    1. Put the following content in the file
        ```yaml
        ---
        version: 3

        images:
        base_image:
            name: registry.redhat.io/ansible-automation-platform-24/ee-minimal-rhel8:latest

        dependencies:
        system:
            - gcc [platform:rpm]
            - git [platform:rpm]
            - openssh-clients [platform:rpm]
            - openssl-devel [platform:rpm]
            - sshpass [platform:rpm]
        python:
            - future
            - six
            - pywinrm
            - ansible-pylibssh
            - azure-cli
            # TODO: Paste here the content of the requirements.txt file from the Azure Git Repo
        galaxy:
            collections:
            - name: ansible.posix
            - name: ansible.windows
            - name: community.windows
            - name: community.postgresql
            - name: microsoft.ad
            - name: community.general
            - name: azure.azcollection
        ansible_core:
            package_pip: ansible-core
        ansible_runner:
            package_pip: ansible-runner

        options:
        package_manager_path: /usr/bin/microdnf

        ```
    1. Save the file
1. Build the execution environment
    ```bash
    ansible-builder build -f execution-environment.yml -t azure_demo_ee
    ```
1. To see your newly created image
    ```bash
    podman image list
    ```

## Publish your automation execution environment to Automation Content
1. For simplicity, create the temporary environment variables
    ```bash
    export AUTOMATION_HUB_IP="[automation-content-IP-address]"
    export USERNAME="[username]"
    export PASSWORD="[REDACTED]"
    export EE_NAME="azure_demo_ee"
    ```
1. Login to `Automation Content`
    ```bash
    podman login -u="$USERNAME" -p="$PASSWORD" "$AUTOMATION_CONTENT_IP_ADDRESS"
    ```
1. Re-tag your execution environment image
    ```bash
    podman tag "localhost/$EE_NAME" "$AUTOMATION_CONTENT_IP_ADDRESS/$USERNAME/$EE_NAME"
    ```
1. Push your execution environment image to Automation Content
    ```bash
    podman push "$AUTOMATION_CONTENT_IP_ADDRESS/$USERNAME/$EE_NAME"
    ```
1. Logout of Automation Content
    ```bash
    podman logout "$AUTOMATION_CONTENT_IP_ADDRESS"
    ```
