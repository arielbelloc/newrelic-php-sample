# New Relic PHP agent deployment sample

Goal for this repo is to present best practices on how to deploy the New PHP agent:

- How to always install the latest agent version;

- How to rely on env variables to automate different deployments;

- How to use a minimum set of labels to better tag the services;

- How to deploy to different New Relic APM IDs to allow data consolidation;

## Setup the required env variables

In this particular use case we are considering a scenario where a service is deployed to different cloud providers. Traffic is switched / balanced across different providers and both a provider view and a consolidated view are required.

We can easily fulfill this requirement by setting up a CLOUD_PROVIDER variable, adding an alias application name for the consolidated view as well as adding labels to better organize them:

- CLOUD_PROVIDER='YOUR_CLOUD_PROVIDER_NAME_HERE'
- NEW_RELIC_APP_NAME=sample-service-$CLOUD_PROVIDER;sample-service
- NEW_RELIC_LICENSE_KEY='YOUR_LICENSE_KEY_HERE'
- NEW_RELIC_DISTRIBUTED_TRACING_ENABLED=true
- NEW_RELIC_LABELS=cloud_provider:$CLOUD_PROVIDER;service:sample-service

## Install the latest version of the New Relic agent

I'm assuming there is a PHP entrypoint script we can customize to do the required changes to the agent configuration file before the PHP instance comes up. On this script we run sed commands to apply the variables from the previous step:

`sed -i -e "s/NEW_RELIC_LICENSE_KEY/${NEW_RELIC_LICENSE_KEY}/g" /usr/local/etc/php/conf.d/newrelic.ini`

`sed -i -e "s/NEW_RELIC_APP_NAME/${NEW_RELIC_APP_NAME}/g" /usr/local/etc/php/conf.d/newrelic.ini`

`sed -i -e "s/NEW_RELIC_DISTRIBUTED_TRACING_ENABLED/${NEW_RELIC_DISTRIBUTED_TRACING_ENABLED}/g" /usr/local/etc/php/conf.d/newrelic.ini`

`sed -i -e "s/NEW_RELIC_LABELS/${NEW_RELIC_LABELS}/g" /usr/local/etc/php/conf.d/newrelic.ini`

Finally execute the following sequence to always get the latest agent version from the New Relic repo and copy our template newrelic.ini configuration to correct location:

`apt-get update -y`

`apt-get install -y gnupg`

`echo 'deb http://apt.newrelic.com/debian/ newrelic non-free' > /etc/apt/sources.list.d/newrelic.list`

`curl -Ls https://download.newrelic.com/548C16BF.gpg | apt-key add -`

`apt-get update -y`

`apt-get install -y newrelic-php5`

`newrelic-install install`

`cp newrelic.ini /usr/local/etc/php/conf.d`

## This REPO

There is a sample-service on this REPO explaining these techniques. You can deploy both the service and required MySQL DB using docker-compose but bear in mind this service has a dependency not present on this REPO. You might wanna change the _checkSku() method to always return a valid sample response or pick at the full REPO at https://github.com/ThyWoof/robot-shop.