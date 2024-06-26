
# Leveraged Vaults contest details

- Join [Sherlock Discord](https://discord.gg/MABEWyASkp)
- Submit findings using the issue page in your private contest repo (label issues as med or high)
- [Read for more details](https://docs.sherlock.xyz/audits/watsons)

# Q&A

### Q: On what chains are the smart contracts going to be deployed?
Ethereum, Arbitrum
___

### Q: If you are integrating tokens, are you allowing only whitelisted tokens to work with the codebase or any complying with the standard? Are they assumed to have certain properties, e.g. be non-reentrant? Are there any types of [weird tokens](https://github.com/d-xo/weird-erc20) you want to integrate?
EtherFi: weETH, eETH
Ethena: USDe, sUSDe
Pendle: PT tokens
Kelp: rsETH
___

### Q: Are there any limitations on values set by admins (or other roles) in the codebase, including restrictions on array lengths?
Wherever possible, restrictions for access controlled methods have require statements embedded inside the contract. Within the Leveraged Vault framework, calls made by the Notional proxy (i.e.  `depositFromNotional` and `redeemFromNotional`) are restricted by deposit sizes and maximum leverage ratios enforced by the main Notional contract.

Calls to `VaultRewarderLib::updateRewardToken`  will extend some internal storage array, but it should be assumed that this will not extend to some length that is unreasonable since the method is controlled by the owner.
___

### Q: Are there any limitations on values set by admins (or other roles) in protocols you integrate with, including restrictions on array lengths?
The codebase integrates with Ethena, EtherFi, Kelp and Pendle. It is possible that there are limitations on values set somewhere within their ecosystem but there are no such values that we would explicitly consider in this audit.
___

### Q: For permissioned functions, please list all checks and requirements that will be made before calling the function.
VaultRewarderLib::updateRewardToken: this method allows Notional governance to whiltelist incentive tokens that vault users would receive directly. It also allows governance to set an emission rate for incentive tokens.

BaseStakingVault::forceWithdraw: allows the EMERGENCY_EXIT role to force an account into the withdraw queue in the event of a depeg event. This role cannot cause take funds from the account, it can only put the account into a withdraw queue.

___

### Q: Is the codebase expected to comply with any EIPs? Can there be/are there any deviations from the specification?
N/A
___

### Q: Are there any off-chain mechanisms or off-chain procedures for the protocol (keeper bots, arbitrage bots, etc.)?
Liquidation bots are used to liquidate vault accounts.

Reinvestment bots sell incentive tokens and reinvest them back into LP tokens for the SingleSidedLPVault. If an incentive token is whitelisted in VaultRewarderLib::updateRewardToken, this bot should be explicitly prevented from selling those tokens by revoking its permissions in the TradingModule. (this check is performed in the updateRewardToken method).
Watsons should assume that these off-chain mechanisms will always work correctly without any delays and without them going offline.
___

### Q: Are there any hardcoded values that you intend to change before (some) deployments?
N/A
___

### Q: If the codebase is to be deployed on an L2, what should be the behavior of the protocol in case of sequencer issues (if applicable)? Should Sherlock assume that the Sequencer won't misbehave, including going offline?
On L2, the Chainlink Sequencer oracle is checked and will cause any oracle checks to revert.
___

### Q: Should potential issues, like broken assumptions about function behavior, be reported if they could pose risks in future integrations, even if they might not be an issue in the context of the scope? If yes, can you elaborate on properties/invariants that should hold?
Yes, we expect that the PendlePT oracle cannot be manipulated. In the SingleSidedLP vault we have read-only reentrancy checks against BalancerV2 and Curve to prevent totalSupply manipulation. If similar exploits are possible against Pendle, EtherFi, Ethena, or Kelp and they can be used against the leveraged vault we would consider that a valid issue.
___

### Q: Please discuss any design choices you made.
Design decisions are discussed in this scoping document:
https://docs.google.com/document/d/1gWFOt9m3fwFGMHk5TxPOf8KkY1ONqY9N1wPKT02ZJfI/edit
___

### Q: Please list any known issues and explicitly state the acceptable risks for each known issue.
The Ethena library (contracts/vaults/staking/protocols/Ethena.sol) has a hardcoded trade of sUSDe to sDAI on a specific Curve pool. This is due to issues handling the withdraw of sDAI to DAI inside the trading module. While this trade works right now, in the future the liquidity for sUSDe may move to a different trading venue and any affected contracts would need to be upgraded. 

Issues surrounding an excessively long ETH withdraw queue lasting months or years would not necessarily qualify as valid. The protocol will specify collateral parameters that ensure that it can safely wait out any reasonable delay in the ETH withdraw queue.
___

### Q: We will report issues where the core protocol functionality is inaccessible for at least 7 days. Would you like to override this value?
No
___

### Q: Please provide links to previous audits (if any).
https://github.com/notional-finance/contracts-v3/tree/master-v3/audits
___

### Q: Please list any relevant protocol resources.
https://docs.google.com/document/d/1gWFOt9m3fwFGMHk5TxPOf8KkY1ONqY9N1wPKT02ZJfI/edit
___



# Audit scope


[leveraged-vaults-private @ 9d0df3326f085e4f732e03fea65e0dee4b7dbf04](https://github.com/notional-finance/leveraged-vaults-private/tree/9d0df3326f085e4f732e03fea65e0dee4b7dbf04)
- [leveraged-vaults-private/contracts/oracles/PendlePTOracle.sol](leveraged-vaults-private/contracts/oracles/PendlePTOracle.sol)
- [leveraged-vaults-private/contracts/vaults/common/VaultRewarderLib.sol](leveraged-vaults-private/contracts/vaults/common/VaultRewarderLib.sol)
- [leveraged-vaults-private/contracts/vaults/common/WithdrawRequestBase.sol](leveraged-vaults-private/contracts/vaults/common/WithdrawRequestBase.sol)
- [leveraged-vaults-private/contracts/vaults/staking/BaseStakingVault.sol](leveraged-vaults-private/contracts/vaults/staking/BaseStakingVault.sol)
- [leveraged-vaults-private/contracts/vaults/staking/PendlePTEtherFiVault.sol](leveraged-vaults-private/contracts/vaults/staking/PendlePTEtherFiVault.sol)
- [leveraged-vaults-private/contracts/vaults/staking/PendlePTGeneric.sol](leveraged-vaults-private/contracts/vaults/staking/PendlePTGeneric.sol)
- [leveraged-vaults-private/contracts/vaults/staking/PendlePTKelpVault.sol](leveraged-vaults-private/contracts/vaults/staking/PendlePTKelpVault.sol)
- [leveraged-vaults-private/contracts/vaults/staking/PendlePTStakedUSDeVault.sol](leveraged-vaults-private/contracts/vaults/staking/PendlePTStakedUSDeVault.sol)
- [leveraged-vaults-private/contracts/vaults/staking/protocols/ClonedCoolDownHolder.sol](leveraged-vaults-private/contracts/vaults/staking/protocols/ClonedCoolDownHolder.sol)
- [leveraged-vaults-private/contracts/vaults/staking/protocols/Ethena.sol](leveraged-vaults-private/contracts/vaults/staking/protocols/Ethena.sol)
- [leveraged-vaults-private/contracts/vaults/staking/protocols/EtherFi.sol](leveraged-vaults-private/contracts/vaults/staking/protocols/EtherFi.sol)
- [leveraged-vaults-private/contracts/vaults/staking/protocols/Kelp.sol](leveraged-vaults-private/contracts/vaults/staking/protocols/Kelp.sol)
- [leveraged-vaults-private/contracts/vaults/staking/protocols/PendlePrincipalToken.sol](leveraged-vaults-private/contracts/vaults/staking/protocols/PendlePrincipalToken.sol)


