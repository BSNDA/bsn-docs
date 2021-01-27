# Cross-Chain Request to the Chainlink Oracle

The chainlink oracle network has been integrated with BSN-Irita-Hub. The chainlink node as the service provider will be able to offer a wide range of off-chain data for the BSN-Irita-Hub network.

This integration will greatly expand cross-chain use cases on BSN-Irita-Hub, which enables developers on application chains to initiate interchain requests for external data, such as market prices like gold and cryptocurrencies.

The documentation is intended to introduce how to request oracle data across chains for Dapp developers.

## Common Oracle Service

### Oracle Service

The oracle module on layer one of BSN-Irita-Hub serves as a special provider named `Module Service Provider`. The binding service name is `oracle-price` that is pre-defined in BSN-Irita-Hub `iService`.

The service definition schemas are as follows:

```json
{
    "input": {
        "$schema": "http://json-schema.org/draft-04/schema#",
        "title": "iservice-oracle-price-input-body",
        "description": "Interchain Service Oracle Price Input Body Schema",
        "type": "object",
        "properties": {
            "pair": {
                "description": "exchange pair",
                "type": "string",
                "pattern": "^[0-9a-zA-Z/]+-[0-9a-zA-Z/]+$"
            }
        },
        "additionalProperties": false,
        "required": [
            "pair"
        ]
    },
    "output": {
        "$schema": "http://json-schema.org/draft-04/schema#",
        "title": "iservice-oracle-price-output-body",
        "description": "Interchain Service Oracle Price Output Body Schema",
        "type": "object",
        "properties": {
            "rate": {
                "description": "exchange rate",
                "type": "string",
                "pattern": "^(?:[1-9]+\\d*|0)(\\.\\d+)?$"
            }
        },
        "additionalProperties": false,
        "required": [
            "rate"
        ]
    }
}
```

### Oracle Feed

BSN-Irita-Hub can create different feeds in the oracle module on demand, and the chainlink oracle will provide the corresponding feed values which are aggregated on-chain as per the given rule as the final result.

Once there exist expected feeds on BSN-Irita-Hub, the consuming contracts only need to invoke the `oracle-price` service for the oracle data and parse the response by the output schema above.

### Oracle Contract

For convenience, the general EVM-based oracle contract has been deployed on the BSN network as a proxy to interact with the `oracle-price` service.

## Chainlink node defined oracle service

Consumers can directly initiate oracle requests to the chainlink nodes by the means of the general interchain service invocation process.

The services provided by the chainlink nodes can be found in the corresponding service market contract on the application chains.
