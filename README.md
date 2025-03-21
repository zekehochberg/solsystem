# SolSystem

Friendly Solana RPC API client supporting both Http and Websockets. 

Includes some endpoints specific only to Helius and the Metaplex DAAS API for
retrieving asset data.


## Why + Plans
After trying the official solana python + solders packages I was left feeling like they were an afterthought of development and weren't supported too well. The API wasn't to my liking so I decided to develop something more pythonic and cleaner.

As I work on solana projects I plan to add further functionality to this API. One of the goals is to integrate a DEX API as well, likely from jupiter.


## Version Note

This API was built with Python 3.12 and the latest package versions available at the time. The purpose was to take advantage of the great typing additions that have come to python and pydantic up to version 3.12. The library can fairly easily be backported to earlier versions of python, but it was not in the current scope of the author's work. 


## Examples

Using the `SyncClient` to return account info for a particular public key. Common configuration parameters are provided via the Configuration Object. The response object here will be a fully typed solsystem Response object with populated fields.

```python
from SolSystem import (
    SyncClient,
    GetAccountInfo,
    Configuration,
    Encoding,
)
def main():
    with SyncClient(rpc_endpoint = "<RPC ENDPOINT URL>") as client:
        response = client.request(
            method = GetAccountInfo(
                account = "<BASE58 PUBLIC KEY>",
                configuration = Configuration(
                    encoding = Encoding.JSONPARSED,
                )
            )
        )
        # Note response is fully typed as a Response[Account] object
        print(response.model_dump_json(indent = 2))
```


Here we use the `AsyncClient` to get the current account balance and display the balance in both SOL and Lamports. We can use the Lamports response object to easily convert and perform arithmetic operations on the value.

```python
import asyncio
from SolSystem import (
    AsyncClient,
    GetAccountBalance,
    Configuration,
    Commitment,
)
async def main():
    async with AsyncClient(rpc_endpoint = "<RPC ENDPOINT URL>") as client:
        resopnse = await client.request(
            method = GetAccountBalance(
                account = "<BASE58 PUBLIC KEY>",
                configuration = Configuration(
                    commitment = Commitment.CONFIRMED,
                )
            )
        )
        # Note response is fully typed as a Response[Lamports] object
        print(response.model_dump_json(indent = 2))

        print(F"Lamport Value: {response.value}")
        print(F"Sol Value: {response.value.sol}")
```

The `Websocket` client works as a factory which creates subscribed clients. Each call to `subscribe` on the factory will create a sepearte object that manages the subscription for its specific method. We then use an async iterator or a loop to recieve messages on each subscription.


```python
import asyncio
from SolSystem import (
    WebsocketClient,
    WsGetAccountInfo,
    Configuration,
    Commitment,
)
async def main():
    async with WebsocketClient(
            end_point = "<WS RPC ENDPOINT URL>",
            message_limit = 3,
    ) as client_factory:
        # We will only recieve 3 messages per subscription
        account_subscription = await client_factory.subscribe(
            method = WsGetAccountInfo(
                account = "<BASE58 PUBLIC KEY>",
                configuration = Configuration(
                    commitment = Commitment.FINALIZED,
                )
            )
        )

        async for message in account_subscription:
            print(F"Owner: {message.value.owner}")
            print(F"Balance: {message.value.lamports}")
            print(F"Data: {message.value.data}")
            
        await account_subscription.unsubscribe()
```


## Development

The project uses `pdm` as both the package manager and the build tool for simplicity. For development simply pull and run `pdm install` provided you have the correct python version and pdm installed already.

Simple tests are available in the `/tests/` folder for confirming that response and request models are working correctly.

The project was developed in an environment running ruff and pylance so type checking was done through them.