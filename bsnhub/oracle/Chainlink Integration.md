
# Chainlink Node Operator Deployment Documentation

As the core of BSN-Irita-Hub, `iService` (short for [`Interchain Service`](https://github.com/bianjieai/irita/blob/master/docs/core_modules/iservice.md)) aims to build the bridge which facilitates the secure and efficient cross-chain interoperability in the form of services.

In particular,
the oracle service is built in layer 1 of BSN-Irita-Hub, which is fed by a variety of external data and can be invoked across multiple application chains.

The complete integration with chainlink has been implemented officially so that chainlink is able to provide the off-chain oracle service with the existent data source and workflow.

The documentation is meant to introduce how to deploy related integration components as the chainlink node operator.

## Deploy service

Firstly define a service on BSN-Irita-Hub and bind it according to the instructions in [BSN-Irita-Hub iService Docs](https://github.com/bianjieai/irita/blob/master/docs/console/modules/iservice.md).

- Define service

Example command:

```bash
irita tx service define --name=eth-usdt-price --description="eth vs usdt price service" --schemas=./eth-usdt-svc-schemas.json --fees 4point --from node0 --chain-id irita --keyring-backend file -y --home mytestnet/node0/iritacli
```

- Bind service

Example command:

```bash
irita tx service bind --service-name eth-usdt-price --deposit=100000point --pricing='{"price":"1point"}' --qos 50 --options='{}' --fees 4point --from node0 --chain-id irita --keyring-backend file -y --home mytestnet/node0/iritacli
```

## Start the chainlink node

Install and start the chainlink node through the [chainlink offcical docs](https://github.com/smartcontractkit/chainlink).

> Note: Be sure to enable the external initiator capacity with the environment variable `FEATURE_EXTERNAL_INITIATORS` set to true.

## Start the external initiator for BSN-Irita-Hub

### Install

The installation instructions can be found in the [chainlink external initiator docs](https://github.com/smartcontractkit/external-initiator).

> Note: Make sure to check out the branch which supports the BSN-Irita-Hub initiator. E.g. `master`.

### Create the external initiator in the chainlink node

The external initiator should be added into the chainlink node. E.g.

```bash
chainlink initiators create bsn-irita http://localhost:8080/jobs
```

The access token and secret key will be returned for interaction with the external initiator.

### Configure environment variables for authentication

Configure the above-generated access tokens and secret keys for access between the chainlink node and external initiator.

```bash
export EI_IC_ACCESSKEY=<outgoing-access-token>
export EI_IC_SECRET=<outgoing-secret-key>
export EI_CI_ACCESSKEY=<incoming-access-token>
export EI_CI_ACCESSKEY=<incoming-secret-key>
```

> Note: Other environment variables should be configured according to the official docs.

### Start the external initiator

Start the external initiator with the BSN-Irita-Hub endpoint. Note that the type field should be `bsn-irita`.

```bash
external-initiator '{"name":"bsn-irita","type":"bsn-irita","url":"<BSN-Irita-Hub rpc url>"}'
```

## Start the external adatper for BSN-Irita-Hub

### Install

See the [BSN-Irita-Hub external adapter](https://github.com/secret2830/bsn-irita-adapter) and install the external adapter by the project docs.

### Create the external adapter in the chainlink node

The external adapter should be added into the chainlink node. E.g.

```bash
chainlink bridges create '{"name":"bsn-irita-adapter", "url":"http://localhost:8080"}'
```

### Start

Configure environment variables by the documentation, then start the external adapter:

```bash
external-adapter
```

## Start the external adapter for result conversion

If the service result provided by the underlying server does not conform to the output schema in the service definition, as is the case with the chainlink official external adapters, the result should be converted through the service result adapter.

The service result adapter currently only supports the conversion from number to string.

### Install

See the [service result adapter](https://github.com/secret2830/service-result-adapter) and install the adapter by the project docs.

### Create the service result adapter in the chainlink node

The service result adapter should be added to the chainlink node. E.g.

```bash
chainlink bridges create '{"name":"service-result-adapter", "url":"http://localhost:8080"}'
```

### Start

```bash
service-result-adapter
```

## Create a BSN-Irita-Hub specific job spec

Once all required components started, the workflow spec can be created in the chainlink node.

```bash
chainlink jobs create <job-spec-path>
```

Example job spec:

```json
{
    "initiators": [
        {
            "type": "external",
            "params": {
                "name": "bsn-irita",
                "body": {
                     "endpoint": "bsn-irita",
                     "serviceName": "eth-usdt-price",
                     "addresses": ["iaa1cq3xx80jym3jshlxmwnfwu840jxta032aa4jss"]
                 }
            }
        }
    ],
    "tasks": [
        {
            "type": "HTTPGetWithUnrestrictedNetworkAccess",
            "params": {
                "get": "http://localhost:8080/"
            }
        },
        {
            "type": "service-result-adapter", 
            "params": {"result_field": "last" }
        },
        {
            "type": "bsn-irita-adapter"
        }
    ]
}
```
