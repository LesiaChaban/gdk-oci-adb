# Configure the Application to use the OCI Autonomous Database Instance

## Introduction

This lab provides instructions to configure the application to connect to the OCI Autonomous Database instance using credentials stored as secrets in the OCI Vault.

The [Micronaut Oracle Cloud](https://micronaut-projects.github.io/micronaut-oracle-cloud/latest/guide/index.html) project provides integration between Micronaut applications and Oracle Cloud, including using Vault as a distributed configuration source.

Estimated Lab Time: 05 minutes

### Objectives

In this lab, you will:

* Confirm the Application dependencies
* Configure the Application to connect to the OCI Autonomous Database instance
* Configure the Application to read secrets stored in OCI Vault
* Configure OCI Instance Principal Authentication

## Task 1: Confirm the Application dependencies

1. In VS Code, open _oci/pom.xml_.

2. Confirm the _oci/pom.xml_ file contains a dependency on the `micronaut-oraclecloud-vault` library.

    _oci/pom.xml_

    ``` xml
    <dependency>
        <groupId>io.micronaut.oraclecloud</groupId>
        <artifactId>micronaut-oraclecloud-vault</artifactId>
        <scope>compile</scope>
    </dependency>
    ```

3. Confirm the distributed configuration feature is enabled using the `micronaut.config-client.enabled` flag in the _bootstrap-oraclecloud.properties_ file.

    _oci/src/main/resources/bootstrap-oraclecloud.properties_

    ``` properties
    micronaut.config-client.enabled=true
    ```

4. Confirm the following snippet in the _bootstrap-oraclecloud.properties_ file. This snippet configures the application to read secrets stored in OCI Vault when the application is running on an OCI Compute Instance. You'll set the value of the externalized configuration variables `COMPARTMENT_ID` and `VAULT_ID` in subsequent steps.

    _oci/src/main/resources/bootstrap-oraclecloud.properties_

    ``` properties
    #a
	oci.config.instance-principal.enabled=true
	oci.vault.config.enabled=true
	#b
	oci.vault.vaults[0].compartment-ocid=${COMPARTMENT_ID}
	#c
	oci.vault.vaults[0].ocid=${VAULT_ID}
    ```

    a) The application is configured to use `OCI Instance Principal Authentication` when it is running on an OCI Compute Instance.

    b) The OCID of the `Compartment` that contains pre-provisioned secrets for the database credentials (`ADB_WALLET_PASSWORD`, `ADB_USER`, and `ADB_USER_PASSWORD`).

    c) The OCID of `Vault`.

5. The application uses externalized configuration (`ADB_WALLET_PASSWORD`, `ADB_USER`, and `ADB_USER_PASSWORD`) to establish a database connection. Here, the application will automatically connect to OCI Vault and retrieve the values of the secrets `ADB_WALLET_PASSWORD`, `ADB_USER`, and `ADB_USER_PASSWORD` from OCI Vault. Micronaut will automatically generate and download the Wallet and configure the data source using the retrieved secrets.

    _oci/src/main/resources/application-oraclecloud.properties_

    ``` properties
    # <2>
    datasources.default.walletPassword=${ADB_WALLET_PASSWORD}
    # <3>
    datasources.default.username=${ADB_USER}
    # <4>
    datasources.default.password=${ADB_USER_PASSWORD}
    ```

## Task 2: Configure the Application to connect to the OCI Autonomous Database instance

Add the Autonomous Database instance ID to the application configuration.

1. From the **Autonomous Database details** screen in the Oracle Cloud Console, click **Copy** to copy the value of the database OCID.

   ![ADB Copy OCID](./images/adb-copy-ocid.jpg#input)

2. In VS Code, go to `datasources.default.ocid` in _application-oraclecloud.properties_ and replace the copied OCID value.

	_oci/src/main/resources/application-oraclecloud.properties_

    ``` properties
    # <1>
    datasources.default.ocid=ocid1.autonomousdatabase.oc1.phx.anyhqljtcrsaiiiadm5oq4qnbfabcdr3v24jfdd7xtwi4pbwsivz3ya
    ```

3. Save (`CTRL+S`) the file.

## Task 3: Configure the Application to read secrets stored in OCI Vault

1. In VS Code, open `bootstrap-oraclecloud.properties`. The application configuration uses two environment variables `COMPARTMENT_ID` and `VAULT_ID`.

    _oci/src/main/resources/bootstrap-oraclecloud.properties_

    ``` properties
	#b
	oci.vault.vaults[0].compartment-ocid=${COMPARTMENT_ID}
	#c
	oci.vault.vaults[0].ocid=${VAULT_ID}
    ```

2. Navigate to the **Identity & Security >> Compartments** section in the Oracle Cloud Console, find your work compartment in the list, open it, in the **Compartment Information** section click **Copy** to copy the value of the Compartment OCID.

3. Open a new terminal in VS Code using the **Terminal > New Terminal** menu.

4. Set the environment variable `COMPARTMENT_ID` using the compartment OCID you copied.

	``` bash
	<copy>
	export COMPARTMENT_ID=ocid1.vault.oc1...
	</copy>
	```

5. Confirm the value set by running the following command:

	``` bash
	<copy>
	echo $COMPARTMENT_ID
	</copy>
	```

6. Set the environment variable `VAULT_ID` using the Vault OCID from the **Lab 4.1** .

	``` bash
	<copy>
	export VAULT_ID=ocid1.vault.oc1...
	</copy>
	```

7. Confirm the value set by running the following command:

	``` bash
	<copy>
	echo $VAULT_ID
	</copy>
	```


## Task 2: <if type="desktop">Use</if><if type="tenancy">Configure</if> OCI Instance Principal Authentication

1. In VS Code, open `application-oraclecloud.properties`. The application is configured to use `OCI Instance Principal Authentication` when it is running on an OCI Compute Instance.

	_oci/src/main/resources/application-oraclecloud.properties_

	``` properties
	oci.config.instance-principal.enabled=true
	```

<if type="desktop">
2. The workshop environment includes a preconfigured `Instance Principal` using a `Dynamic Group` and a `Policy` in OCI to allow the application to send logs to OCI Logging.
</if>

<if type="tenancy">
2. The following steps show you how to set up an `Instance Principal` using a `Dynamic Group`-less `Policy` in OCI to allow the application to send logs to OCI Logging.

3. From the Oracle Cloud Console navigation menu, go to **Identity & Security >> Identity >> Policies**.

	![Policies Menu](https://oracle-livelabs.github.io//common/images/console/id-policies.png)

4. Go to your workshop compartment.

5. Click  **Create Policy**.

6. Enter a name and description.

7. Select your workshop compartment.

8. In the **Policy Builder** section, click **Show manual editor**.

9. Enter the following policy statement in the text area. Replace the placeholders `WORKSHOP_COMPARTMENT_NAME` with your workshop compartment name, and `WORKSHOP_COMPARTMENT_OCID` with your workshop compartment OCID.

	``` text
	<copy>
	Allow any-user to read secret-family in compartment WORKSHOP_COMPARTMENT_NAME where ALL {request.principal.type='instance', request.principal.compartment.id='WORKSHOP_COMPARTMENT_OCID'}
	</copy>
	```

	To learn more about policies to control access to OCI Logging, see [Policy Reference - Details for Logging](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/databasepolicyreference.htm).

</if>

	To learn more about the supported authentication options, see [Micronaut Oracle Cloud Authentication](https://docs.oracle.com/en-us/iaas/Content/Identity/Reference/databasepolicyreference.htm).

Congratulations! In this lab, you configured the application to connect to the OCI Autonomous Database instance using credentials stored as secrets in the OCI Vault.

You may now **proceed to the next lab**.

## Acknowledgements

* **Author** - [](var:author)
* **Contributors** - [](var:contributors)
* **Last Updated By/Date** - [](var:last_updated)
