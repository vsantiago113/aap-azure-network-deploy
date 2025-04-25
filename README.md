# AAP 2.5 Webinar Demo

## Create an execution enviroment

1. Resources
    - [Register a Microsoft Entra app and create a service principal](https://learn.microsoft.com/en-us/entra/identity-platform/howto-create-service-principal-portal)
    - [Ansible collection for Azure](https://galaxy.ansible.com/ui/repo/published/azure/azcollection/docs/)
    - [GitHub Azure](https://github.com/ansible-collections/azure/tree/dev)

1. System Requirements
    - python3
    - python3-pip
    - podman
    - git
    - ansible-core

1. Python Requirements
    - ansible-navigator
    - ansible-builder

1. Ansible Navigator Config
    ```yaml
    ---
    ansible-navigator:
    execution-environment:
        container-engine: podman
        enabled: true
        image: localhost/aap_2-5_demo:latest
        pull:
        arguments:
            - "--tls-verify=false"
        policy: never

    images:
        details:
        - ansible_version
        - python_version
        - ansible_collections

    logging:
        level: critical
        append: False
        file: /tmp/ansible_navigator.txt

    playbook-artifact:
        enable: False

    mode: stdout

    ```

!. Ansible Azure Credentials
    ```txt
    export AZURE_SUBSCRIPTION_ID="REDACTED"
    export AZURE_CLIENT_ID="REDACTED"
    export AZURE_SECRET="REDACTED"
    export AZURE_TENANT="REDACTED"
    ```

1. Ansible ServiceNow Credentials
    ```txt
    export SN_HOST='REDACTED'
    export SN_USERNAME='REDACTED'
    export SN_PASSWORD='REDACTED'
    ```

1. How to build execution environment
    ```bash
    ansible-builder build -f execution-environment-aap-2.5-demo.yml -t azure_demo_ee
    ```

1. Azure CLI Commands
    ```bash
    az login --service-principal -u $AZURE_CLIENT_ID -p $AZURE_SECRET --tenant $AZURE_TENANT

    az network watcher configure --locations centralus --enabled false

    az provider register --namespace Microsoft.Network
    az provider register --namespace Microsoft.Compute

    az provider show -n Microsoft.Compute --query "registrationState"

    ```
