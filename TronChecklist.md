# Checklist for Auditing Tron Projects

## 1. Energy vs. Gas
- **Energy vs. Gas:** Confirm that the contract is optimized for TVM's energy model rather than EVM's gas. Energy price is currently 280 sun.
- **Gas-related Functions:** Verify that `GASPRICE`, `DIFFICULTY`, and `GASLIMIT` return zero as expected in TVM, and that `BASEFEE` is not used.

- [fee limits](https://www.btcschools.net/tron/tron_fee_limit.php)
- [energy/bandwidth calculation](https://www.btcschools.net/tron/tron_station.php)

## 2. Opcode Energy Consumption
- **Energy Consumption:** Check that most opcodes have equivalent energy consumption as in EVM. Review specific opcodes like `SLOAD` and `CALL` which have lower costs in TVM.
- [energy cost calculation](https://developers.tron.network/v4.4.0/docs/energy-cost)

## 3. Address Prefix 
- **Tron base58 adresses** Tron uses Base58 adresses, to get Tron address from solidity code you should add prefix `0x41` and encode it in base58, i.e. `0xa614f803B6FD780986A42c78Ec9c7f77e6DeD13C` => `41a614f803B6FD780986A42c78Ec9c7f77e6DeD13C` => `TR7NHqjeKQxGTCi8q8ZY4pL8otSzgjLj6t` 
- ([Tron address calculator](https://www.btcschools.net/tron/tron_tool_base58check_hex.php)) 
- **Address Prefix: for CREATE2** Validate that contract addresses created with `CREATE2` use the TVM prefix `0x41` instead of EVM's prefix. Verify the formula: `keccak256(0x41 ++ address ++ salt ++ keccak256(init_code))[12:]`.

## 4. Pre-compiled Contracts
- **Ripemd160 Pre-compiled Contract:** Verify the behavior of the pre-compiled contract at `0x03`. Ensure it calculates SHA-256 twice on the input instead of the standard Ripemd160.
- **Blake2F and BatchValidateSign:** Confirm that the pre-compiled contract at `0x09` is `BatchValidateSign` in TVM, as opposed to Blake2F in EVM. Reference [TIP-43](https://github.com/tronprotocol/tips/blob/master/tip-43.md) for details.

## 5. TRX Transfer Methods
- **TRX Transfer vs. TriggerSmartContract:** Validate the two methods to send TRX to contracts: Transfer and TriggerSmartContract with a `callValue`. Ensure that using Transfer does not invoke fallback functions in contracts.

## 6. TRON-specific Features
- **TRC-10 Related Opcodes:** Review usage of TRC-10 related opcodes: `CALLTOKEN (0xd0)`, `TOKENBALANCE (0xd1)`, `CALLTOKENVALUE (0xd2)`, and `CALLTOKENID (0xd3)`.
- **Contract Address Verification:** Check for correct implementation of `ISCONTRACT (0xd4)` opcode to determine if an address belongs to a contract as per [TIP-44](https://github.com/tronprotocol/tips/blob/master/tip-44.md).
- **Batch Validation:** Audit usage of batch validation opcodes such as `BatchValidateSign (0x09)` from [TIP-43](https://github.com/tronprotocol/tips/blob/master/tip-43.md) and `ValidateMultiSign (0x0a)` from [TIP-60](https://github.com/tronprotocol/tips/blob/master/tip-60.md).

## 7. Anonymous Contract and Privacy-Related Pre-compiled Contracts
- **Librustzcash Pre-compiled Contracts:** Validate correct usage of privacy-related pre-compiled contracts: `verifyMintProof (0x1000001, 0x1000002, 0x1000003)` and `merkleHash (0x1000004)`. Reference [TIP-135](https://github.com/tronprotocol/tips/blob/master/tip-135.md), [TIP-137](https://github.com/tronprotocol/tips/blob/master/tip-137.md), and [TIP-138](https://github.com/tronprotocol/tips/blob/master/tip-138.md).

## 8. Freeze/Unfreeze Functions
- **Freeze/Unfreeze Opcodes:** Verify correct implementation and usage of `FREEZE (0xd5)`, `UNFREEZE (0xd6)`, and `FREEZEEXPIRETIME (0xd7)`. Reference [TIP-157](https://github.com/tronprotocol/tips/blob/master/tip-157.md).

## 9. Contract Voting and Governance
- **Voting-related Opcodes:** Ensure correct implementation of voting-related opcodes: `VOTEWITNESS (0xd8)`, `WITHDRAWREWARD (0xd9)`, and pre-compiled contracts related to voting such as `RewardBalance (0x1000006)`, `IsSrCandidate (0x1000006)`, `VoteCount (0x1000007)`, `UsedVoteCount (0x1000008)`, `ReceivedVoteCount (0x1000009)`, and `TotalVoteCount (0x100000a)`. Reference [TIP-271](https://github.com/tronprotocol/tips/blob/master/tip-271.md).

## 10. USDT TRC-20 behavior
Due to an error in the [standard contract USDT TRC20](https://tronscan.org/#/token20/TR7NHqjeKQxGTCi8q8ZY4pL8otSzgjLj6t/code) the `transfer` function returns `0000000000000000000000000000000000000000000000000000000000000000` in data (equal `return false`) for `success true` transactions

## 11. Undefined values behavior in TronBox tests for events
Event log messages may show `undefined` in TronBox tests for memory variables. i.e. 
`emit Transfer(success, data, data.length);` will show `undefined` in case of empty data in TronBox tests for all the parameters. This may cause confusion during the testing phase.
Workaround - you can use private storage variables to save parameters values into them and use for events. 

## 12. Compatibility Considerations
- **Check for Ongoing Discussions:** Stay updated with ongoing discussions on compatible solutions, including [TIP-272](https://github.com/tronprotocol/tips/blob/master/tip-272.md) and related GitHub issues. Review the implications of any changes that may affect contract functionality.
- [Differences from EVM](https://developers.tron.network/v4.4.0/docs/vm-vs-evm)

## 13. Testing and Validation
- **Testing on Testnet:** Test all unique TVM features and differences from EVM on Tron's testnet to ensure compatibility and correct behavior. You can use [TronIDE](https://www.tronide.io/) with connected [TronLink Wallet](https://www.tronlink.org/).
Explore transactions via [TronScan](https://tronscan.org/) and [deep TX info Tool ](https://www.btcschools.net/tron/tron_get_tx_info.php)
- **Testnet Faucet** - [Tron community chat with faucet Bot](https://t.me/TronOfficialDevelopersGroupEn)   
- **Testing on TronBox:** Perform unit testing to explore edge cases and ensure the contract handles all unique TVM behaviors [TronBox install](https://developers.tron.network/reference/install) 

