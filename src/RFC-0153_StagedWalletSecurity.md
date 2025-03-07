# RFC-0153/StagedWalletSecurity

## Staged Wallet Security

![status: stable](theme/images/status-stable.svg)

**Maintainer(s)**: [Yuko Roodt](https://github.com/neonknight64), [Cayle Sharrock](https://github.com/CjS77)

# Licence

[The 3-Clause BSD Licence](https://opensource.org/licenses/BSD-3-Clause).

Copyright 2021 The Tari Development Community

Redistribution and use in source and binary forms, with or without modification, are permitted provided that the
following conditions are met:

1. Redistributions of this document must retain the above copyright notice, this list of conditions and the following
   disclaimer.
2. Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following
   disclaimer in the documentation and/or other materials provided with the distribution.
3. Neither the name of the copyright holder nor the names of its contributors may be used to endorse or promote products
   derived from this software without specific prior written permission.

THIS DOCUMENT IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS", AND ANY EXPRESS OR IMPLIED WARRANTIES,
INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL,
SPECIAL, EXEMPLARY OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY,
WHETHER IN CONTRACT, STRICT LIABILITY OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF
THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

## Language

The keywords "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED",
"NOT RECOMMENDED", "MAY" and "OPTIONAL" in this document are to be interpreted as described in
[BCP 14](https://tools.ietf.org/html/bcp14) (covering RFC2119 and RFC8174) when, and only when, they appear in all capitals, as
shown here.

## Disclaimer

This document and its content are intended for information purposes only and may be subject to change or update
without notice.

This document may include preliminary concepts that may or may not be in the process of being developed by the Tari
community. The release of this document is intended solely for review and discussion by the community of the
technological merits of the potential system outlined herein.

## Goals

This Request for Comment (RFC) aims to describe Tari's ergonomic approach to securing funds in a hot wallet.
The focus is on mobile wallets, but the strategy described here is equally applicable to console or desktop wallets.

## Related Requests for Comment

- [RFC-0150: Wallets](RFC-0150_Wallets.md)

## Description

### Rationale

A major UX hurdle when users first interact with a crypto wallet is the friction they experience with the first user
experience.

A common theme: I want to play with some new wallet X that I saw advertised somewhere, so I download it and run it.
But first I get several screens that

- ask me to review my seed phrase,
- ask me to write down my seed phrase,
- prevent typical "skip this" tricks like taking a screenshot,
- ask to confirm if I've written down my seed phrase,
- force me to write a test, either by supplying a random sample of my seed phrase, or by getting me to type in the
  whole thing.

After all this, I play with the wallet a bit, and then typically, I uninstall it.

The goal of this RFC is to **get the user playing with the wallet as quickly as possible**. _Without_ sacrificing
security whatsoever.

### A staged approach

This RFC proposes a smart, staged approach to wallet security. One that maximises user experience without compromising
safety.

Each step enforces more stringent security protocols on the user than the previous step.

The user moves from one step to another based on criteria that

1. the user configures based on her preferences, or
2. uses sane predefined defaults.

The criteria are generally based on the value of the wallet balance.

Once a user moves to a stage, the wallet does _not_ move to a lower stage if the requirements for the stage are no longer
met.

Users may also jump to any more advanced stage from their wallet settings / configuration at any time.

#### Stage zero - zero balance

When the user has a _zero balance_, there's no risk in letting them skip securing their wallet.

Therefore, Tari wallets SHOULD just skip the whole seed phrase ritual and let the user jump right into the action.

#### Stage 1a - a reminder to write down your seed phrase

Once the user's balance exceeds the `MINIMUM_STAGE_ONE_BALANCE`, they will be prompted to review and write down their
seed phrase. The `MINIMUM_STAGE_ONE_BALANCE` is any non-zero balance by default.

After the transaction that causes the balance to exceed `MINIMUM_STAGE_ONE_BALANCE` is confirmed, the user is presented
with a friendly message: 

```text
You now have _real_ money in your wallet. If you accidentally delete your wallet app or lose
your device, your funds are lost, and there is no way to recover them unless you have safely kept a copy of your
`seed phrase` safe somewhere. Click 'Ok' to review and save the phrase now, or 'Do it later' to do it at a more
convenient time.
```

If the user elects _not_ to save the phrase, the message pops up again periodically. Once per day, or when the balance
increases -- whichever is less frequent -- is sufficient without being too intrusive.

#### Stage 1b - simple wallet backups

Users are used to storing their data in the cloud. Although this practice is frowned upon by crypto purists, for small
balances (the type you often keep in a hot wallet), using secure cloud storage for wallet backups is a fair
compromise between keeping the keys safe from attackers and protecting users from themselves.

The simple wallet backup saves the spending keys and values of the user's wallet to a personal cloud space (e.g. Google Drive,
Apple iCloud, Dropbox).

This solution does not require any additional input from the user besides providing authorisation to store in the cloud.
This can be done using the standard APIs and Authentication flows that each cloud provider publishes for their platform.

In particular, we do not ask for a password to encrypt the commitment data. The consequence is that anyone who gains
access to this data -- by stealing the user's cloud credentials -- _could_ steal the user's funds.

Therefore, the threshold for moving from this stage to Stage 2, `STAGE_TWO_THRESHOLD_BALANCE` is relatively low;
somewhere in the region of \\$10 to \\$50.

The seed phrase MUST NOT be stored on the cloud in Stage 1b. Doing so would result in all _future_ funds of the user being
lost if the backup were ever compromised. Since the backup is unencrypted in Stage 1b, we store the minimum amount of data
needed to recover the funds and limit the potential loss of funds in case of a breach to just that found in the commitments in the
backup, which should not be more than \\$50.

Therefore, stage 1b backups are really just exporting and importing of UTXO data. The consequence of this is 
that restoring from a 1b backup does not restore the emoji id. On the other hand, you can easily import into any 
other wallet (e.g. into another wallet on another device owned by the same user).

Backups MUST be authorised by the user when the first cloud backup is made and SHOULD be automatically updated after each
transaction is confirmed.

Wallet authors MAY choose to exclude Stage 1b from the staged security protocol.

As usual, the user MUST be able to configure `STAGE_TWO_THRESHOLD_BALANCE` to suit their particular needs.

When this threshold is reached, the user SHOULD be prompted with a call to back-up their data:

```text
Your wallet now holds a fair amount of value, which you probably don't want to lose. Unlike a physical wallet's 
notes, it's possible to make a copy of your wallet's contents that you can use to recover your funds in case you 
lose them.
 
You should think about making a copy of your wallet's coins, which we will keep up date date after every transaction.

If you have a copy of your seed phrase saved somewhere safe, and don't want to back up your coins into the cloud, you 
can skip this step.
```

The user can pick between `backup my coins` or `I have my seed phrase. Skip backups for now`.

If the user chooses to skip the backup, do not prompt them for stage 1b backups again.

#### Stage 2 - full wallet backups

Once a user has a significant balance (over `STAGE_TWO_THRESHOLD_BALANCE`), Stage 2 is active. Stage 2 entails a full,
encrypted backup of the user's wallet to the cloud. The user needs to provide a **password** to perform and secure the encryption.

This makes the user's fund safer while at rest in the cloud. It also introduces an additional point of failure: the user
can forget their wallet's encryption password.

Stage 1b and 2 are similar in functionality but different in scope (Stage 2 allows us to store all the wallet
metadata, rather than just the commitments). For this reason, Stage 1b is optional.

Backups MUST be authorised by the user when the first cloud backup is made and SHOULD be automatically updated after each
transaction.

When migrating from Stage 1 to Stage 2, the Stage 1b backups SHOULD be deleted.

Once the stage 2 threshold is reached, the user is again presented with a call-to-action:

```text
                   😎💰💰💰😎
It's awesome that you're using [this Tari wallet] so much!
You're a high-roller, so it's time to add another layer of security 
to your wallet backups.

Your funds will be safer if we encrypt your cloud backups with an 
additional password of your own choosing. You need to make sure that 
this password is very string and that YOU DO NOT FORGET it!

(Whatever happens, you can always restore from your seed phrase, so 
make sure you're still keeping this safe and secreted away).

As an added benefit, password-encrypted backups allow us to backup your 
transaction history as well!            
```

The user can choose to `Upgrade my backups`, `do this later`, `check my backup settings`. 

The latter option takes the user to the wallet settings, where they can configure the various thresholds and 
manually decide which backup strategy they want to pursue.

They should be prompted periodically (once per day is sufficient) if they choose to defer the upgrade.

#### Stage 3 - Sweep to cold wallet

**Pending hardware wallet support**

Above a given limit -- user-defined, or the default `MAX_HOT_WALLET_BALANCE`, the user should be prompted to transfer
funds into a cold wallet. The amount to sweep can be calculated as `MAX_HOT_WALLET_BALANCE` - `SAFE_HOT_WALLET_BALANCE`.

If the user ignores the prompt, they SHOULD be reminded one week later. From the second prompt onward, users SHOULD be given
an option to re-configure the values for `MAX_HOT_WALLET_BALANCE` and `SAFE_HOT_WALLET_BALANCE`.

Assuming one-sided payments are live, the user SHOULD be able to configure a `COLD_WALLET_ADDRESS` in the wallet.

For security reasons, a user SHOULD be asked for their 2FA confirmation, if it is configured, before broadcasting the
sweep transaction to the blockchain.

A suitable prompt for the user once the `MAX_HOT_WALLET_BALANCE` threshold is reached is:

```text
🐋🐋🐋🐋🐋🐋🐋🐋  ❗❗❗ Whale Alert ❗❗❗  🐋🐋🐋🐋🐋🐋🐋🐋
Your mobile wallet holds way more funds than would normally be 
considered safe to walk around with in your pocket.

You should really consider sweeping most of your balance into a 
cold- or hardware wallet.   
```

Possible actions from this prompt include:
* `Set up my cold wallet address`, if the cold wallet address has not been configured.
* `Transfer funds to my cold wallet`, if the cold wallet _has_ been set up. The usual transaction confirmation flow 
  SHOULD be followed here, and a stealth one-sided payment SHOULD be the default transaction type.
* `I'll deal with this myself`, which takes the user to their backup settings.

To avoid spamming the user, do not fire this prompt more than once every three days.

#### Security hygiene

- From stage 1 onwards Users should be asked periodically whether they still have their seed phrase written down.
  Once every two months is sufficient.

# Change Log

| Date       | Change               | Author              |
|:-----------|:---------------------|:--------------------|
| 2022-11-10 | Initial stable       | Adrian Truszczyński |
| 2022-12-12 | Update user messages | CjS77               |
| 2022-12-20 | Fix typo             | CjS77               |