# Honeypot Detector

A honeypot token is a token which allows users to buy it but cannot be exchanged to any other token.This means the token does not have any liquidity.

As a solution We have built a honeypot token detector which does the due- diligence on behalf of the user to check for potential honeypot properties.

The honeypot detector forks the mainet, and queries smartcontracts of jediSwap for the token queried by the user. When the simulation is run parameters like

- Sell tax
- buy tax
- Amount_out
- Reserves
  are recorded

We also take legacy data from the blochain to see if the transaction properties match.

The token qualifies as a honeypot if the following properties are detected

- Very high sell tax
- If the amount_out for a quoted token does not match the simulation transfers
- If the reserves for a token pair is too low
- If there is no token Pool

## Onchain security implementation

We also provide an Oracle which is constantly updated with the honeypot status of a token which can be used by other contracts like Lending and borrowing platforms, flashloans , Aggregators. And can avoid these tokens onchain itself.
This oracle can also be used on tokens that are not honeypot but have chances of going to a very low reserve state and quickly be notifie to balance funds.

## Tech stack

- Katana local environment
- Express
- JediSwap
- Cairo

## Setup

To run the honeypot detector locally, Clone the repository with this command or run it on codespaces
`gh repo clone gitshreevatsa/Starknet-Honeypot`
run the command

```
cd backend
npm i
npx nodemon
```

or

```
docker pull muskbuster/honeypot
```

```
docker run -p 8000:8000 -d muskbuster/honeypot:v1
```

this will start the instance of honeypot detector in your localhost and has one endpoint
you can call the endpoint using curl

```bash
curl "http://localhost:8000/?tokenAddress=<ADDRESS_OF_TOKEN>"

```

Response will be of type json

```json
{
  "data": {
    "tokenName": "Tether USD",
    "testToken": "Starknet Token",
    "tax": 0.989990964463765,
    "isHoneyPot": false,
    "tokenAddress": "0x068f5c6a61780768455de69077e07e89787839bf8166decfbf92b645209c0fb8"
  },
  "transaction": "0x5ec46eef8caffe3b1fe2d504fe310878c1ac907ea97e37dfc45c0e55e70dd84"
}
```

This will return the status of a token stating if it is a honeypot or not

## Usage

We have deployed a frontend for the detector which is directly connected to our Oracle aswell.You can simply input the address of the token you want to learn about and get to know if it is a honeypot token or not.
This can be used by the public to conduct their due-diligence for a token.

To integrate the Oracle You can simply call the get_if_honeypot function at the address

`0x6a0c05a08fbf89100b6082971eb74d9766088da91651e48e4c535945c40d61d`

in goerli for now (But has mainet data)

```cairo
use starknet::{ContractAddress, ContractState};

trait IhoneypotDispatcherTrait<T> {
    fn get_if_honeypot(self: @T, token_address: ContractAddress) -> u16;
}

struct HoneypotDispatcher {
    contract_address: ContractAddress,
}

impl IhoneypotDispatcherTrait<HoneypotDispatcher> for HoneypotDispatcher {
    fn get_if_honeypot(self: @HoneypotDispatcher, token_address: ContractAddress) -> u16 {
 HoneypotDispatcher .get_if_honeypot(token_address)
    }
}
#[starknet::contract]
mod HoneypotWrapper {
    use super::{IhoneypotDispatcherTrait, HoneypotDispatcher};
    use starknet::ContractAddress;

    #[storage]
    struct Storage {}

    impl HoneypotWrapper of IHoneypotDispatcherTrait<HoneypotDispatcher> {
        fn get_if_honeypot(
            self: @ContractState,
            contract_address: ContractAddress,
            token_address: ContractAddress,
        ) -> u16 {
            HoneypotDispatcher { contract_address }.get_if_honeypot(token_address)
        }
    }
}

```

And can be called with any other logic

## architecture

![image](https://github.com/gitshreevatsa/Statknet-Honeypot/assets/81789395/150b6731-e06e-4896-991d-965684846a8f)

## Further Advancements

- Upon full development of katana we will be simulating other aspects of the tokens and run code checks on the token contract using **amarna** static analyzer to give more and more insignts on the tokens
- We will expand this tool to other types of exchanges that do not follow uniswap AMM architecture aswell
- Enable this for Kakarot zkevm once released and all appchains using **Madara**
- Provide advanced APIs for free to build newer and better detectors
