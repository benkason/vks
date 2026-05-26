# Certificate rotation

## Documentation links

Official documentation for rotating certs can be found by going to
`https://<opsman fqdn>/certificates` and viewing the list of expiring certs.
Each cert has a link to the rotation procedure on the far right side of the
table.
Example:

![opsman certs page](./images/opsman_certs_page.png)

The following steps are based on Location, Type and Configurability of the
certificate that is due to expire.

Overall, everything related to certificates is documented under this section of
the Opsman docs: https://docs.vmware.com/en/VMware-Tanzu-Operations-Manager/2.10/vmware-tanzu-ops-manager/security-pcf-infrastructure-certs-index.html

## Location: ops_manager, Type: Leaf, Configurable: No

Original doc: [Rotating Non-Configurable Leaf Certificates](https://docs.vmware.com/en/VMware-Tanzu-Operations-Manager/2.10/vmware-tanzu-ops-manager/security-pcf-infrastructure-rotate-non-configurable-certs.html)

Perform the following steps from PKS CLI Jump box for required environment (Sandbox/Preprod/Prod)

1. Set the following environment variables:
    ```
    export OM_TARGET=<opsman_url> e.g (https://opsman_url)
    export OM_USERNAME=<opsman_username>
    export OM_PASSWORD=<opsman_password>
    export OM_SKIP_SSL_VALIDATION=1
    ```

1. Regenerate the certs in Opsman:

    ```bash
    om regenerate-certificates
    ```

1. Go to the Opsman UI and select "Review Pending Changes" with default checks enabled.

1. Click "Apply Changes".

1. Once changes are applied successfully, go back to the Opsman certificates page then check which other certificates needs to be rotated.

## Location: credhub, Type: Leaf, Configurable: No

Original doc: [Tanzu Kubernetes Grid Integrated Edition Certificates](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid-Integrated-Edition/1.15/tkgi/GUID-certificate-concepts.html)

:warning: Never use the CredHub Maestro `maestro regenerate ca/leaf –-all`
command to rotate TKGI certificates.
TKGI requires you to rotate certs separately based on their CA and the type of
cert.

1. Go to the Ops Manager UI > BOSH tile > Credentials tab.
    Select BOSH Commandline Credentials, as shown in the below image.

    ![opsman bosh credentials](./images/opsman_bosh_credentials.png)

    On that page, copy the values of:
    + BOSH_CLIENT
    + BOSH_CLIENT_SECRET
    + BOSH_CA_CERT
    + BOSH_ENVIRONMENT

1. SSH into the Ops Manager VM:

    ```bash
    ssh -i ~/.ssh/opsman ubuntu@<opsman_fqdn_or_ip>
    ```

1. Export the following variables, with the values you got from the previous
    step:

    ```bash
    export CREDHUB_SERVER=https://<BOSH_ENVIRONMENT>:8844
    export CREDHUB_CLIENT=<BOSH_CLIENT>
    export CREDHUB_SECRET=<BOSH_CLIENT_SECRET>
    export CREDHUB_CA_CERT=<BOSH_CA_CERT>
    ```

1. Use the maestro CLI to list all Credhub-managed certs expiring within the
    next 3 months:

    ```bash
    maestro list --expires-within 3m
    ```

    Look at the 'name' field for each cert, and run the following procedures
    based on which certs are in the list:

    Cert name                   | Steps to follow
    --------------------------- | ---------------
    `/bosh_dns_health_client_tls` | [Rotate BOSH DNS certificates](#rotate-bosh-dns-certificates)
    `/bosh_dns_health_server_tls` |
    `/dns_api_client_tls`         |
    `/dns_api_server_tls`         |
    `tls-nsx-t`                   | [Rotate TKGI NSX-T certificates](#rotate-tkgi-nsx-t-certificates)
    `tls-nsx-lb`	                |
    Other                       | See public VMware documentation: https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid-Integrated-Edition/1.14/tkgi/GUID-certificate-concepts.html

## Rotate BOSH DNS certificates

Original doc: [How to rotate bosh-dns certificates in TKGI 1.9+ using maestro](https://kb.vmware.com/s/article/88187)

1. To get a list of certificates signed by /opsmgr/bosh_dns/tls_ca run the following command.

    ```
    maestro --json topology --name /opsmgr/bosh_dns/tls_ca | jq .topology[].signs[].name
    ```
    You will get example output as mentioned below

    ```
    "/bosh_dns_health_client_tls"
    "/bosh_dns_health_server_tls"
    "/dns_api_client_tls"
    "/dns_api_server_tls"
    ```

1. To check the validity of certificates

    ```
    maestro --json topology --name /opsmgr/bosh_dns/tls_ca | jq '.topology[].signs[] | "\(.name) \(.versions[].valid_until)"'
    ```
    **Sample output :**
    ```
    "/bosh_dns_health_client_tls 2021-12-18T02:30:06Z"
    "/bosh_dns_health_server_tls 2021-12-18T02:30:06Z"
    "/dns_api_client_tls 2021-12-18T02:30:07Z"
    "/dns_api_server_tls 2021-12-18T02:30:07Z"
    ```

1. To regenerate certificates use the following command

    ```
    maestro regenerate leaf --signed-by /opsmgr/bosh_dns/tls_ca
    ```
    **Sample output:**

    ```
    regenerated:
    - name: /bosh_dns_health_client_tls
    certificate_id: 0b402fa6-04a3-492f-849d-cde95f5cff88
    - name: /bosh_dns_health_server_tls
    certificate_id: b6e7b8f2-edb4-4c2e-a1a5-332cd9f28c37
    - name: /dns_api_client_tls
    certificate_id: 0ae32163-a76a-4d14-8fa8-79e5402b9511
    - name: /dns_api_server_tls
    certificate_id: 5ac68fdb-01ef-4d50-a9f3-92281e57a74c
    ```

1. Login to Opsman UI
1. Apply pending changes
1. Then upgrade cluster one by one manually using the following command.

    ```
    pks upgrade-cluster <clustername>
    ```
    **example:**
    ```
    pks upgrade-cluster petclinic
    ```

### Rotate TKGI NSX-T certificates

![NSX-T_Cert](./images/NSX-T_Cert.png)

Original doc: [Rotate NSX-T Certificates for Kubernetes Clusters](https://docs.vmware.com/en/VMware-Tanzu-Kubernetes-Grid-Integrated-Edition/1.14/tkgi/GUID-nsxt-certs-rotate.html)

```
pks rotate-certs <clustername> --only-nsx
```
For example:
```
pks rotate-certs petclinic --only-nsx
```

**Note**: Do it only for the cluster which you see TKGI NSX-T cert is in need
of rotation

:warning: Users who have Ingress certificates will be impacted by this change.
When the NSX-T cert is rotated, NSX-T stops serving user-configured certs for
Ingress endpoints, instead reverting back to the default self-signed NSX load
balancer cert.
If there are users who have configured an Ingress certificate, you will need to
work with them to apply the following workaround once rotation is complete.

1. Get credentials for the cluster you are working on:

    ```bash
    pks get-credentials <clustername>
    ```

1. First, run the following command to list the Ingress objects that are
    impacted:

    ```bash
    kubectl get ingress -A | sed 's/, /,/g' | awk '$6~/443/ {print "Name: "$2", Namespace: "$1", Hostname: "$4}'
    ```

    The output will look something like this:

    ```
    Name: argocd-server, Namespace: argocd, Hostname: argocd.dev.apps.nga.api.mtn.com
    Name: sm-ingress, Namespace: comviva, Hostname: smdev.mtn.com.ng
    Name: edgemicro-mtn-ng, Namespace: default, Hostname: preprods-nigeria.api.mtn.com
    ```

1. Log into the NSX-T UI and go to System > Certificates.

1. For each Ingress hostname in the above output, search for certs whose "Issued
    To" field matches that hostname.
    The "Where Used" field should show as 0.

    ![NSX-T certs page](./images/nsxt_certs.png)

    Note the Name (UUID) of each cert that you find.

1. Use BOSH SSH to the cluster's master VM:

    ```bash
    bosh -d <cluster deployment name> ssh master/0
    ```

1. For each cert UUID you copied, run the following command to delete the cert
    from NSX-T:

    ```bash
    CERT_UUID=<cert uuid>
    curl -k \
        --cert /var/vcap/jobs/pks-nsx-t-prepare-master-vm/config/nsx_t_client.crt \
        --key /var/vcap/jobs/pks-nsx-t-prepare-master-vm/config/nsx_t_client.key \
        -X DELETE \
        "https://ojnsxmanagercls.mtn.com.ng/api/v1/trust-management/certificates/${CERT_UUID}"
    ```

1. logout from bosh and login to PKSCLI jumpbox

1. Delete and re-create each affected Ingress object (you got their names and
    namespaces in a previous step):

    ```bash
    INGRESS_NAME=<ingress name>
    INGRESS_NAMESPACE=<ingress namespace>

    kubectl get ingress "$INGRESS_NAME" -n "$INGRESS_NAMESPACE" -o yaml > "$INGRESS_NAME".yml
    kubectl delete -f "$INGRESS_NAME".yml
    kubectl apply -f "$INGRESS_NAME".yml
    ```

1. Once you complete the above step for an Ingress, NSX-T should resume serving
    the correct user-configured certificate.
    You can test this with the following command:

    ```bash
    openssl s_client -connect <ingress_hostname>:443 -servername <ingress_hostname> </dev/null 2>/dev/null | grep 'subject='
    ```

    The output should look something like this:

    ```bash
    subject=/C=XX/ST=StateName/L=CityName/O=CompanyName/OU=CompanySectionName/CN=<ingress_hostname>
    ```
    The output can also be something like this in case wild card certificate is there
    ```bash
    subject=/C=XX/ST=StateName/L=CityName/O=CompanyName/OU=CompanySectionName/CN=<*.mtn.com.ng>
    ```    



## Location: ops_manager, Type: Leaf, Configurable: Yes

![Configurable_cert](./images/Configurable_cert.png)

Original doc: [Rotating Configurable Leaf Certificates](https://docs.vmware.com/en/VMware-Tanzu-Operations-Manager/2.10/vmware-tanzu-ops-manager/security-pcf-infrastructure-rotate-configurable-certs.html)

1. Login to Opsmanager and go to TKGI tile
1. Click on setting and TKGI API
1. Under "Certificate to secure the TKGI API" click change
1. Click Generate rsa Certificate
1. Click Save at the bottom of each pane in which you added new leaf certificates.
1. Return to the Ops Manager Installation Dashboard.
1. Click Review Pending Changes.
1. Click Apply Changes.
