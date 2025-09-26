# UtilityToken (ERC-20)

## Project overview

This repository contains a sample **Utility ERC‑20 token** implemented in Solidity and built with **Foundry** (Forge + Anvil). The contract demonstrates production-oriented patterns that should be used when creating utility tokens for a real project: OpenZeppelin audited building blocks, role-based access, EIP‑2612 `permit`, vesting support via `VestingWallet`, and tests with Foundry.

---

## What is a Utility Token?

A utility token grants holders **access, functionality, or usage rights** inside a platform or application. It is typically not intended as an investment vehicle (although market reality and legal classification vary). Example utilities include:

* Paying for in‑app services (API calls, compute credits).
* Unlocking premium features or content.
* Minting or redeeming in‑app items.
* Discounting fees or enabling loyalty mechanics.

**Important**: whether a token is legally a "utility" token or a security depends on how it is marketed, distributed, and used. Always consult legal counsel before public sales.

---

## Why ERC‑20?

ERC‑20 (EIP‑20) is the standard interface for fungible tokens on Ethereum and is supported by wallets, exchanges, and DeFi infrastructure. Benefits:

* Interoperability across wallets and exchanges.
* Simple, well‑understood interface (`transfer`, `approve`, `transferFrom`, `balanceOf`, etc.).
* Widely audited implementations (OpenZeppelin) reduce risk.

Common extensions used here:

* **ERC20Permit (EIP‑2612)**: gasless approvals via signed permits (better UX).
* **AccessControl**: role-based permissioning for minting/pausing.
* **Pausable**: emergency stop for transfers.
* **VestingWallet**: off‑the‑shelf vesting for team/advisor allocations.

---

## Contract architecture & design choices

1. **OpenZeppelin building blocks** — (`ERC20`, `ERC20Permit`, `AccessControl`, `Pausable`, `VestingWallet`) .
2. **Roles** — `DEFAULT_ADMIN_ROLE`, `MINTER_ROLE`, `PAUSER_ROLE` give fine‑grained control and allow assigning privileges to a multisig (Gnosis Safe) instead of a single private key.
3. **Permit** — `ERC20Permit` improves UX by allowing off‑chain approvals that the token holder signs; relayers can submit the permit transaction on behalf of users.
4. **Vesting** — team/advisor tokens are held in `VestingWallet` instances with configurable start and duration; vesting wallets do linear releases and require beneficiaries to call `release` to pull vested tokens.
5. **Pausable** — emergency pause for transfers and other public functions.
6. **Non‑upgradeable by default** — simpler & safer; upgradeability via proxies requires careful auditing.

---

## Constructor summary (example: `UtilityTokenVested`)

```solidity
constructor(
    uint256 initialSupply,
    address admin
) ERC20("MyUtility", "MUT") ERC20Permit("MyUtility") {
    // forward admin to Ownable/AccessControl or mint to admin
}
```

* `initialSupply` — total token units at deployment (consider decimals: token uses 18 decimals by default).
* `admin` — recommended to be a multisig/Gnosis Safe address; assign roles and initial minted tokens to this address.

---

## Common ERC‑20 functions

* `totalSupply()` — total tokens in existence.
* `balanceOf(address)` — token balance of an address.
* `transfer(address,uint256)` — move tokens from caller to `to`.
* `approve(address,uint256)` — allow `spender` to use caller's tokens.
* `allowance(address,address)` — remaining allowance `spender` has.
* `transferFrom(address,address,uint256)` — move tokens from `from` using allowance.
* Events: `Transfer`, `Approval`.

Extensions:

* `permit(owner,spender,value,deadline,v,r,s)` — EIP‑2612 gasless approval.

---

## Build & Test (Foundry)

### Prerequisites

* Foundry (`foundryup`) installed. See [https://foundry.paradigm.xyz](https://foundry.paradigm.xyz) for install.
* `anvil` for a local node (optional but recommended).

### Setup

```bash
# clone the repo
git clone <repo-url>
cd foundry-utility-token

# install dependencies
e.g. forge install OpenZeppelin/openzeppelin-contracts

# build
forge build
```

### Run tests

```bash
# run all tests with verbose output and gas report
forge test -vvvv --gas-report

# run a single test file
forge test --match-path test/UtilityTokenVested.t.sol -vvvv
```

### Fast local deploy (Anvil)

```bash
# run anvil in a separate terminal
anvil

# in another terminal deploy to local chain
forge script script/Deploy.s.sol --broadcast --rpc-url http://127.0.0.1:8545 --private-key <LOCAL_KEY>
```

---

## Tokenomics (example)

* Total supply: `1,000,000,000` tokens (with 18 decimals).
* Example allocation:

  * Ecosystem / Incentives — 40%
  * Team — 20% (12‑month cliff, 24‑month linear)
  * Advisors — 10% (6‑month cliff, 18‑month linear)
  * Treasury — 15%
  * Public Sale — 10%
  * Liquidity — 5%

---

## Gas & optimization notes

* `transfer` is optimized in OpenZeppelin; heavy operations like loops and many storage writes are the main gas costs.
* `VestingWallet::release` is a user‑triggered function; frequent small releases cost more gas cumulatively — consider UX where beneficiaries batch releases.

---

## License

MIT



