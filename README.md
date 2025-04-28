# AAP 2.5 Webinar Demo

## Create an execution enviroment

1. Resources
    - [Register a Microsoft Entra app and create a service principal](https://learn.microsoft.com/en-us/entra/identity-platform/howto-create-service-principal-portal)
    - [Ansible collection for Azure](https://galaxy.ansible.com/ui/repo/published/azure/azcollection/docs/)
    - [GitHub Azure](https://github.com/ansible-collections/azure/tree/dev)
    - [Publishing an automation execution environment](https://docs.redhat.com/en/documentation/red_hat_ansible_automation_platform/2.4/html/creating_and_consuming_execution_environments/assembly-publishing-exec-env#proc-customize-ee-image)
    - [Install the Azure CLI on Linux](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli-linux?pivots=dnf)

1. System Requirements
    - python3
    - python3-pip
    - podman
    - git
    - ansible-core

1. Python Requirements
    - ansible-navigator
    - ansible-builder

1. Azure CLI Commands
    ```bash
    az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_SECRET --tenant $AZURE_TENANT

    az network watcher configure --locations centralus --enabled false

    az provider register --namespace Microsoft.Network
    az provider register --namespace Microsoft.Compute

    az provider show -n Microsoft.Compute --query "registrationState"

    ```
