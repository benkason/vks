# Retrieve Config YAML for Products & BOSH Director

## Products

The steps below show how to retrieve the config yaml for a staged product in OpsMan. 
This config yaml file can later be used with the `om configure-product` command.

1. Setup your environment to connect to your chosen OpsMan (see: [Setup environment to use OM CLI](om_setup.md))

1. List current products in OpsMan:
    ```
    om products
    ```
    > Note the name of product for which you want the config yaml

1. List pending changes to ensure staged configuration does not differ from deployed configuration. If the configuration is changed then you must apply changes (or revert them) before continuing.
    ```
    om pending-changes
    ```

1. Retrieve the staged config for your product:
    ```
    om staged-config \
        --product-name=<product_name> \
        --include-credentials \
        --include-placeholders > config.yaml
    ```

## BOSH Director
1. Setup your environment to connect to your chosen OpsMan (see: [Setup environment to use OM CLI](om_setup.md))

1. Retrieve the staged config for your BOSH director:
    ```
    om staged-director-config \
        --no-redact \
        --include-placeholders > director.yaml
    ```

