# Timelock Policy for Contract Upgrades

The **Timelock Policy** module introduces a delay mechanism for contract upgrades on the Sui blockchain, ensuring sufficient time for review and consensus before upgrades are executed.

---

## Features

- **Time-Based Upgrade Locking**: Enforces a delay (24 or 48 hours) between upgrades.
- **Customizable Delays**: Adjust delay duration dynamically between predefined values (24 or 48 hours).
- **Protected Upgrade Management**:
  - Can use a multisig address for collaborative control over upgrades.
  - Optionally make the policy immutable for maximum security, but, note this action is irreversible.

---

## How It Works

1. **Timelock Enforcement**:

   - The `TimelockCap` enforces a time delay before an upgrade can be authorized.
   - Tracks the time of the last authorized upgrade to ensure compliance.

2. **Customizable Delays**:

   - Configure delays dynamically between 24 or 48 hours using the `set_delay` function.

3. **Optional Immutability**:
   - Once the `TimelockCap` is stable, you can call `make_immutable` to enforce the policy permanently.

---

## Usage Guide

### Setting Up the Timelock Policy

1. **Publish the Timelock Policy**:
   ```bash
   sui move build
   sui client publish
   ```
2. **Create a TimelockCap**:
   Wrap your contract's `UpgradeCap` with a `TimelockCap` and configure the delay. Use the `new_timelock` function in the policy module:
   ```move
   let timelock_cap = timelock_policy::timelock_upgrade::new_timelock(
       upgrade_cap,    // The UpgradeCap of the contract
       MS_24_HOURS,    // Delay duration (24 hours)
       &mut ctx
   );
   ```
3. **Authorize an Upgrade**:
   Use the `authorize_upgrade` function to validate that the required delay has passed since the last upgrade:
   ```move
    let upgrade_ticket = timelock_policy::timelock_upgrade::authorize_upgrade(
      &mut timelock_cap,
      policy,         // Upgrade policy type (e.g., compatible or additive)
      digest,         // Digest of the new package version
      &mut ctx
    );
   ```
4. **Commit the Upgrade**:
   After authorization, finalize the upgrade by calling `commit_upgrade`:
   ```move
   timelock_policy::timelock_upgrade::commit_upgrade(
    &mut timelock_cap,
    receipt         // The UpgradeReceipt object from the upgrade execution
   );
   ```
5. **(Optional) Update the Delay**:
   Adjust the delay duration between the predefined values (24 or 48 hours) using `set_delay`:

   ```move
   timelock_policy::timelock_upgrade::set_delay(
       &mut timelock_cap,
       MS_48_HOURS     // New delay duration (48 hours)
   );
   ```

6. **(Optional) Make the TimelockCap Immutable**:
   If you wish to secure the policy permanently and prevent further modifications, call `make_immutable`:

   ```bash
   sui client call --gas-budget 1000000 \
       --package <POLICY_PACKAGE_ID> \
       --module 'timelock_upgrade' \
       --function 'make_immutable' \
       --args <TIMELOCK_CAP_ID>
   ```
