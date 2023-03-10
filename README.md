# The Mayfair Witches Pass Metadata

This repository contains information on the randomization process Orange Comet used for `The Mayfair Witches Pass`.

## Introduction

_Why is randomization transparency important?_ In collections where specific tokens have attributes designed to give more rarity, the community may designate such tokens as having higher value. Transparency in any "shuffle" or added utility assures the community that the process is fair to all who participate.

After reviewing various solutions, we selected Chainlink VRF because it’s based on cutting-edge academic research, supported by a time-tested oracle network, and secured through the generation and on-chain verification of cryptographic proofs that prove the integrity of each random number supplied to smart contracts.

By integrating the industry-leading decentralized oracle network, we now have access to a tamper-proof and auditable source of randomness needed to help randomize the unique characteristics of the NFTs for the art reveal and to help randomize the distribution of rewards to NFT owners. Ultimately this creates a more exciting and transparent user experience, as users can have high confidence that the randomness used in the art reveal and winner selection process is provably fair.

For additional information check out our blog post on [The Eternal Collection Uses Chainlink VRF to Help Power Fair NFT Reveal and Winner Selection](https://orangecomet.com/eternal-collection-chainlink-vrf/)

## Randomization Process

The original file names (3333 in total) are listed here in [sorted_metadata_list.txt](./sorted_metadata_list.txt). These files were sorted alphabetically by group title and number.
Orange Comet used [Chainlink's VRF](https://docs.chain.link/docs/vrf/v2/introduction/) product to generate a random number to seed the shuffle method.

The [RandomNumberGenerator](https://etherscan.io/address/0x723f9c44472a17e8f46a86c0ab5befccbbf20ec7) contract makes the requests for the random number with parameters for tracking the purpose of the request. The method `requestRandomWords` is outlined below:

```solidity
  /**
   * @notice Makes a random request
   * Assumes the subscription is funded sufficiently; "Words" refers to unit of data in Computer Science
   *
   * @param partnerContract the address of the partner contract
   * @param totalEntries the total number of entries to randomize
   * @param totalSelections the number of selections to return
   * @param title the title used to write the logs
   */
  function requestRandomWords(
    address partnerContract,
    uint32 totalEntries,
    uint32 totalSelections,
    string calldata title
  ) external onlyOwner {
    // Will revert if subscription is not set and funded.
    uint256 requestId = COORDINATOR.requestRandomWords(
      _keyHash,
      _subscriptionId,
      REQUEST_CONFIRMATIONS,
      CALLBACK_GAS_LIMIT,
      NUM_WORDS
    );

    requests[requestId] = Params(
      partnerContract,
      totalEntries,
      totalSelections,
      title
    );
  }
```

Once Chainlink's oracles have generated a random number, the `fulfillRandomWords` function will be called with the initial request ID and resulting `randomWords`. The result will emit a `ReturnedRandomWord` event in order to signal the generator was successfully called. An example of this can be [viewed on Etherscan](https://etherscan.io/tx/0x3383c64edb01f5c5b3cb5484ad7647bdc9583bd05863db4191801d8b9f2b9334#eventlog)

```solidity
  /**
   * @notice Callback function used by VRF Coordinator
   *
   * @param requestId - the requestId from the VRF Coordinator
   * @param randomWords - array of random results from VRF Coordinator
   */
  function fulfillRandomWords(uint256 requestId, uint256[] memory randomWords)
    internal
    override
  {
    emit ReturnedRandomWord(randomWords[0], requestId, requests[requestId]);
  }
```

To ensure that `the DESIGNEE` key is awarded to a user who minted during our initial primary drop (March 7, 2023 through March 9, 2023), we will run the randomization request until the asset's token # falls within the range of minters.

## Frontend Verification

For convenience an [application](https://rng-verification.orangecomet.io/) was developed to take a `fulfillRandomWords` transaction (implemented by an Orange Comet RandomNumberGenerator contract), parse the logs and shuffle the results using [the shuffle method](./src/typescript/tools.ts). The web application also allows us to disqualify certain tokens based on conditions such as already existing utility and flagged wallets.
