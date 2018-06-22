# Integration of Open Service Broker on Azure in helm charts

This is a reference code on how we created a helm chart to deploy a service that connects to a custom created database.

1. Make sure that there is a file `env.sh` with all the needed credetials. You will need to have a contributor SP, you can use my script to create it:

    ```bash
    export KUBECONFIG= <- Location to your kubernetes cluster config file ->
    export AZURE_SUBSCRIPTION_ID= <- Subscription ID ->
    export AZURE_TENANT_ID= <- Tenant ID ->
    export AZURE_CLIENT_ID= <- Service Principal Client ID ->
    export AZURE_CLIENT_SECRET= <- Service Principal secret/password ->
    ```

2. Install the service catalog in your cluster:

    ```bash
    ./install-serv-broker.sh
    ```

3. Test out the service broker with the Azure Wordpress example:

    ```bash
    helm install azure/wordpress --name osba-quickstart --namespace osba-quickstart
    ```

    Remember, it takes about 10-15 minutes to get the service intance running and binded to your cluster. You can run the following command to monitor the progress:

    ```bash
    kubectl get all -n osba-quickstart
    ```

4. Let's install the [service catalog CLI:](https://github.com/Azure/service-catalog-cli) 

    ```bash
    curl -sLO https://servicecatalogcli.blob.core.windows.net/cli/latest/$(uname -s)/$(uname -m)/svcat
    chmod +x ./svcat
    mv ./svcat /usr/local/bin/
    svcat --version
    ```

    You can now see the available classes that your service catalog has:

    ```bash
    svcat get classes
    ```

    And if needed you can trigger a sync to obtain the latest version of the catalog in your cluster:

    ```bash
    svcat sync broker osba
    ```

    You can use it to describe the previous provisioned MySQL instance, with commands like `svcat get intances` and `svcat describe`.

5. Update the chart values and make sure they are the ones you currently need. Explore the `service-broker-sample-chart`folder.Inside the file called `values.yaml` we will be able to set up the configuration for the Database:

    ```yaml
    externalDatabase:
        azure:
            ## The Azure region in which to deploy Azure Database for MSSQL
            location: westeurope
            ## The plan to request for Azure Database for MSSQL
            servicePlan: basic
            ## The Azure Resource Group where this Database will be installed
            resourceGroup: coolresourcegrupo
    ```
    The information of the code to be run can be reached by providing the proper registry:

    ```yaml
    image:
        repository: myawesomeregistry.mine/myimage
        tag: cooltag
        pullPolicy: Always
        pullSecretsName: myawesomeregistry.mine.azurecr.io
    ```

    With the classes we have obtained we can see the options that our cluster has to offer. In the case of the cluster provisioned for this tutorial this is the information we have on the cluster:

    ```bash
    $ svcat get classes

                NAME                         DESCRIPTION                             UUID
    +-------------------------------+--------------------------------+--------------------------------------+
    azure-postgresql-9-6-database   Azure Database for PostgreSQL    25434f16-d762-41c7-bbdd-8045d7f74ca6  
                                    9.6-- database only
    azure-sql-12-0-database         Azure SQL 12.0-- database only   2bbc160c-e279-4757-a6b6-4c0a4822d0aa  
    azure-mysql-5-7-dbms            Azure Database for MySQL 5.7--   30e7b836-199d-4335-b83d-adc7d23a95c2  
                                    DBMS only
    azure-mysql-5-7-database        Azure Database for MySQL 5.7--   6704ae59-3eae-49e9-82b4-4cbcc00edf08  
                                    database only
    azure-mysql-5-7                 Azure Database for MySQL 5.7--   997b8372-8dac-40ac-ae65-758b4a5075a5  
                                    DBMS and single database
    azure-sql-12-0-dbms             Azure SQL 12.0-- DBMS only       a7454e0e-be2c-46ac-b55f-8c4278117525  
    azure-postgresql-9-6            Azure Database for PostgreSQL    b43b4bba-5741-4d98-a10b-17dc5cee0175  
                                    9.6-- DBMS and single database
    azure-postgresql-9-6-dbms       Azure Database for PostgreSQL    d3f74b44-79bc-4d1e-bf7d-c247c2b851f9  
                                    9.6-- DBMS only
    azure-sql-12-0                  Azure SQL Database 12.0-- DBMS   fb9bc99e-0aa9-11e6-8a8a-000d3a002ed5  
                                    and single database
    ```

    These are the available options that OSBA can deploy for this cluster, confirm you are using a valid version of the SQL database. And this is defined in the `service-broker-sample-chart/templates/sql-instance.yaml`:

    ```yaml
    clusterServiceClassExternalName: azure-sql-12-0
    ```
6. Deploy the chart

    ```bash
    helm install --name osba-sql-chart service-broker-sample-chart
    ```

    Wait some minutes (10-15) and when the pods are all up and running you can test it. Obtain the service external ip with `kubectl get svc`. You can use curl to do a get request to the Ip you get:

    ```bash
    export SVC_IP=" << Service IP >>"
    curl  -H "Authorization: Bearer eyJiOiJIUzI1NiJ9.eyJhdWQiOiJodHRwczo" $SVC_IP/health
    ```

## Resources

- <https://github.com/Azure/open-service-broker-azure/blob/master/docs/modules/mssql.md>
- <https://docs.bitnami.com/kubernetes/how-to/create-your-first-helm-chart/>
