# Installing the Insights Client
The Insights Client requires the installation of the Insights Core. To begin, create the Insights repository directories at the same level as quipucords. Clone the following repositories::
```
git clone git@github.com:RedHatInsights/insights-client.git
git clone git@github.com:RedHatInsights/insights-core.git
```

## Setting Up a Virtual Environment
The insights-client will need to be installed inside of the same virtual environment as the QPC client::

    cd ../location/of/qpc/client
    pipenv shell
    make install

## Essential Preparation Steps
For QPC to access the Insights Client locally on Mac, we need to checkout the `os-x-test` branch:
```
cd insights-client
git fetch origin os-x-test && git checkout os-x-test
```

**NOTE:** Ensure that you are connected to the VPN before continuing to the next stage.

## Insights Client Configuration
You will need to uncomment, add, or modify the following values in the file located at `insights-client/etc/insights-client.conf` in order to be authorized to upload. The `<your_username>` and `<your_password>` are variables based off your login for https://access.redhat.com/.

### CI Configuration
```
auto_config=False
username=username
password=redhat
http_timeout=20
base_url=ci.cloud.paas.upshift.redhat.com/api
cert_verify=False
auto_update=False
legacy_upload=False
```

### Production Configuration
```
auto_config=False
username=<your_username>
password=<your_password>
http_timeout=20
```

### Optional flags (to workaround issues with client)

```
# Optionally you can add the upload_url to point to the entire path for upload
upload_url=https://ci.cloud.paas.upshift.redhat.com/api/ingress/v1/upload
```


## Building the Insights Client
### Building with Insights Client on Mac
After configuration is setup, you will need to build the Insights client:
```
make insights-client
```

### Building with Insights Client on RHEL
After configuration is setup, you will need to build the Insights client::
```
sudo sh lay-the-eggs.sh
```

### Download Insights Core RPM
Download the last stable version of the Insights core::
```
curl https://api.access.redhat.com/r/insights/v1/static/core/insights-core.egg.asc > last_stable.egg.asc
sudo mv last_stable.egg.asc /var/lib/insights/last_stable.egg.asc
curl https://api.access.redhat.com/r/insights/v1/static/core/insights-core.egg > last_stable.egg
sudo mv last_stable.egg /var/lib/insights/last_stable.egg
```
_Note:_ You may need to create the ``/var/lib/insights`` structure.

## Insights Upload Command
To upload a tar.gz file using the Insight Clients you will need to run the following command:
```
sudo BYPASS_GPG=True insights-client --no-gpg --payload=test.tar.gz --content-type=application/vnd.redhat.qpc.insights+tgz
```
_WARNING:_ If a `machine-id` is not present in the `/etc/insights-client` directory, your first upload attempt will fail. However, the `machine-id` will be created for you by the Insights client, so your second attempt will work.

## QPC Upload Command
To upload a QPC Insights report using the QPC Client you will need to run the following command::
```
qpc insights upload (--scan-job scan_job_identifier | --report report_identifier | --no-gpg)
```

_Note:_ If you are developing on a mac, you will need to use the ``--no-gpg`` argument.

## Helpful Insights Debug Commands
To check your connection status using the Insight Clients you will need to run the following command::
```
sudo BYPASS_GPG=True insights-client --no-gpg --test-connection
```

To check the version of your insights-client and Insights core you will need to run::
```
sudo BYPASS_GPG=True insights-client --no-gpg --version --debug
```

## Clean Up
If you make any changes to your configuraiton, Insights client, or Insights core in order to prevent caching issues you will need to run the following commands::
```
make insights-clean
```

_Note:_ After removing these files, you will need to rebuild the Insights client egg & download the Insights core again.
