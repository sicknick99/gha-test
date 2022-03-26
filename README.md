Change in README to trigger GHA to `dev` branch
# gha-test
## Goal 
To reduce the time it takes to run the GitHub Actions tests on WIP branches.
1. Make a `main` branch that GitHub Actions will run the brownie tests with the number of hypothesis iterations (`max_examples`) equal to 50 (the default).
2. Make a `dev` branch that GitHub Actions will run the brownie tests with the number of hypothesis iterations  (`max_examples`) equal to 3.

Because `brownie` does not allow for config specification, but instead requires the configuration file to be in the root directory and named `brownie-config.yaml`, we cannot simply create two separate config files (one for `main` and one for `dev`). 
`brownie` does however allow you to pass in environment variables into the config file.

We must make it so the config file works both locally and can be executed by GitHub Actions.

Therefore, to achieve the goal, we do the following:
1. In `brownie-config.yaml` we define `max_examples` and the `MAX_EXAMPLES` environment variable.
1. For local use, we include the line `export MAX_EXAMPLES=50` for branches being pushed to `main`, and `export MAX_EXAMPLES=3` (or which ever number of iterations desired) for branches in flux that will be pushed to `dev`. (SN TODO: Right now have it set to `export MAX_EXAMPLES=${ME}`).
  For use in GitHub actions, we first get the base branch (expected to be either `main` or `dev`) then have several checks:
    1. If the base branch is `main`, then we check that the source/compare branch is `dev`. If it is not, then fail and prevent merge. (SN TODO: Check this is the desired action with the team). Then `ME` is then set to `50` (SN TODO: `ME` vs just setting `MAX_EXAMPLES`, will that work?)
    1. If the base branch is `dev`, we allow any other branch to be merged in and `ME` is set to `3`.
1. Execute remaining test Actions.

1. Any merges into `main` MUST always be from `dev` (SN TODO: put hard check in GHA for this)


# token-mix

A bare-bones implementation of the Ethereum [ERC-20 standard](https://eips.ethereum.org/EIPS/eip-20), written in [Solidity](https://github.com/ethereum/solidity).

For [Vyper](https://github.com/vyperlang/vyper), check out [`vyper-token-mix`](https://github.com/brownie-mix/vyper-token-mix).

## Installation

1. [Install Brownie](https://eth-brownie.readthedocs.io/en/stable/install.html), if you haven't already.

2. Download the mix.

    ```bash
    brownie bake token
    ```

## Basic Use

This mix provides a [simple template](contracts/Token.sol) upon which you can build your own token, as well as unit tests providing 100% coverage for core ERC20 functionality.

To interact with a deployed contract in a local environment, start by opening the console:

```bash
brownie console
```

Next, deploy a test token:

```python
>>> token = Token.deploy("Test Token", "TST", 18, 1e21, {'from': accounts[0]})

Transaction sent: 0x4a61edfaaa8ba55573603abd35403cf41291eca443c983f85de06e0b119da377
  Gas price: 0.0 gwei   Gas limit: 12000000
  Token.constructor confirmed - Block: 1   Gas used: 521513 (4.35%)
  Token deployed at: 0xd495633B90a237de510B4375c442C0469D3C161C
```

You now have a token contract deployed, with a balance of `1e21` assigned to `accounts[0]`:

```python
>>> token
<Token Contract '0xd495633B90a237de510B4375c442C0469D3C161C'>

>>> token.balanceOf(accounts[0])
1000000000000000000000

>>> token.transfer(accounts[1], 1e18, {'from': accounts[0]})
Transaction sent: 0xb94b219148501a269020158320d543946a4e7b9fac294b17164252a13dce9534
  Gas price: 0.0 gwei   Gas limit: 12000000
  Token.transfer confirmed - Block: 2   Gas used: 51668 (0.43%)

<Transaction '0xb94b219148501a269020158320d543946a4e7b9fac294b17164252a13dce9534'>
```

## Testing

To run the tests:

```bash
brownie test
```

The unit tests included in this mix are very generic and should work with any ERC20 compliant smart contract. To use them in your own project, all you must do is modify the deployment logic in the [`tests/conftest.py::token`](tests/conftest.py) fixture.

## Resources

To get started with Brownie:

* Check out the other [Brownie mixes](https://github.com/brownie-mix/) that can be used as a starting point for your own contracts. They also provide example code to help you get started.
* ["Getting Started with Brownie"](https://medium.com/@iamdefinitelyahuman/getting-started-with-brownie-part-1-9b2181f4cb99) is a good tutorial to help you familiarize yourself with Brownie.
* For more in-depth information, read the [Brownie documentation](https://eth-brownie.readthedocs.io/en/stable/).


Any questions? Join our [Gitter](https://gitter.im/eth-brownie/community) channel to chat and share with others in the community.

## License

This project is licensed under the [MIT license](LICENSE).
