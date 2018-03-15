# Cloud Foundry PostgreSQL Service Broker

This repository contains a BOSH release for a Cloud Foundry PostgreSQL service
broker.

```shell
git clone https://github.com/Infra-Red/postgresql-service-broker-boshrelease.git ~/workspace/postgresql-service-broker-boshrelease
cd ~/workspace/postgresql-service-broker-boshrelease
git submodule update --init --recursive
```

## Dependencies

- [BOSH CLI v2](https://bosh.io/docs/cli-v2.html#install)
- [BOSH](https://bosh.io/docs/init.html)
- [CF](https://github.com/cloudfoundry/cf-deployment)
- [PostgreSQL](https://github.com/cloudfoundry/postgres-release)

## Deployment

Populate a vars file (using `manifest/vars-file.yml` as a template), save it to
`secrets/vars.yml`. You will need values from both your cloud-config and secrets
from your cf-deployment.

To deploy:

```shell
bosh upload-stemcell https://s3.amazonaws.com/bosh-aws-light-stemcells/light-bosh-stemcell-3541.9-aws-xen-hvm-ubuntu-trusty-go_agent.tgz
bosh deploy --vars-file secrets/vars.yml manifest/cf-postgres-template.yml
```

## Registering the Service Broker

After registering the service broker, the PostgreSQL service will be visible in the Services Marketplace; using the [CLI](https://github.com/cloudfoundry/cli), run `cf marketplace`.

### BOSH errand

```
$ bosh -e YOUR_ENV -d postgresql-service-broker run-errand broker-registrar
```

### Manually

1. First register the broker using the `cf` CLI.  You must be logged in as an admin.

    ```
    $ cf create-service-broker shared-postgresql-broker BROKER_USERNAME BROKER_PASSWORD URL
    ```

    `BROKER_USERNAME` and `BROKER_PASSWORD` are the credentials Cloud Foundry will use to authenticate when making API calls to the service broker. Use the values for manifest properties `jobs.postgresql-service-broker.properties.broker.username` and `jobs.postgresql-service-broker.properties.broker.password`.

    `URL` specifies where the Cloud Controller will access the PostgreSQL broker. Use the IP address of `postgresql-service-broker` instance and port of `postgresql-service-broker` job. By default, port is set to `8080`.

    For more information, see [Managing Service Brokers](http://docs.cloudfoundry.org/services/managing-service-brokers.html).

2. Then [make the service plan public](http://docs.cloudfoundry.org/services/managing-service-brokers.html#make-plans-public).

## De-registering the Service Broker

The following commands are destructive and are intended to be run in conjuction with deleting your BOSH deployment.
```
$ bosh -e YOUR_ENV -d postgresql-service-broker run-errand broker-deregistrar
```

### Manually

Run the following:

```
$ cf purge-service-offering a.postgresql
$ cf delete-service-broker shared-postgresql-broker
```