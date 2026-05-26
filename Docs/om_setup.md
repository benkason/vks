# Setup environment to use OM CLI & BOSH CLI

This document explains how to connect to a given OpsMan to run `om` CLI commands and `bosh` CLI commands.

## Option 1: Use Environment Variables

1. Set the following environment variables:
    ```shell
    export OM_TARGET=<opsman_url>
    export OM_USERNAME=<opsman_username>
    export OM_PASSWORD=<opsman_password>
    export OM_SKIP_SSL_VALIDATION=1
    ```

2. Test your configuration:
    ```shell
    om products
    ```
    You should be presented with a list of available and staged products. If you are not, check the values of the environment variables are correct.

3. Log in to the BOSH director and Credhub for your foundation:
    ```shell
    eval "$(om bosh-env)"
    ```
    You can now execute `bosh` CLI commands against your foundation, such as `bosh vms`.

## Option 2: Use Environment File

1. Create a `env.yaml` file with the following contents:
    ```yaml
    ---
    target: https://opsman.example.com
    connect-timeout: 30
    request-timeout: 1800
    skip-ssl-validation: true
    username: admin
    password: mysupersecretpassword
    decryption-passphrase: myultrasecretdecryptionpassphrase
    ```
    > **Note**: `decryption-passphrase` is optional

2. Test your configuration:
    ```shell
    om --env env.yaml products
    ```
    You should be presented with a list of available and staged products. If you are not, check the values in your environment file are correct.

3. Log in to the BOSH director and Credhub for your foundation:
    ```shell
    eval "$(om --env env.yaml bosh-env)"
    ```
    You can now execute `bosh` CLI commands against your foundation, such as `bosh vms`.

> :bookmark: Complete documentation for the OM CLI can be found [here](https://github.com/pivotal-cf/om/blob/main/docs/README.md)

> :bookmark: Complete documentation for the BOSH CLI can be found [here](https://bosh.io/docs/cli-v2/)
