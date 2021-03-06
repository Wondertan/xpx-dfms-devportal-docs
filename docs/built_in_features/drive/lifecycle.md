---
id: lifecycle
title: Drive Lifecycle
sidebar_label: Lifecycle
---

## Create the drive
First of all, a user (the [owner](../../roles/owner.md)) needs to create a new multisig account that will be used as the [drive](overview.md) account. To create the [drive](overview.md), the [owner](../../roles/owner.md) must define the desired properties. After [drive](overview.md) creation, it should transfer the sufficient amount [XPX](../../getting_started/economy.md#xpx) to the drive’s balance. These mosaics will be exchanged to [SO](../../getting_started/economy.md#so) by [replicators](../../roles/replicator.md). When the [drive](overview.md) has been created it has [**NotStarted**](state.md#notstarted) state.
The [drive](overview.md) preparation has no strict time bounds. It is just an initiation of the [drive](overview.md) storage contract. Any [replicator](../../roles/replicator.md) can join it at any time, and their reward will be calculated by the formula (reference). For each new [drive](overview.md) user must create a new multisig account.
> **The example**:
A user wants to create a new 1000MB (driveSize) [drive](overview.md) for 12 months (duration). For this, he determines that he wants 4 replicas, but the [drive](overview.md) can start when 3 [replicators](../../roles/replicator.md) joined. Also, he determines that 100% signatures (percentApprovers) are needed to accept the transaction. And every 1 month ([billing period](overview.md#billing-period)) 4000 [SO](../../getting_started/economy.md#so) (billingPrice = replicas*driveSize) will be distributed between [replicators](../../roles/replicator.md) by the formula.

## Join to the drive
Any user (the [replicator](../../roles/replicator.md)) may agree with the [drive](overview.md) offer and send the join transaction. With this transaction, the deposit ([SO](../../getting_started/economy.md#so); equals to [drive](overview.md) size) is automatically debited from the [owner](../../roles/owner.md). If there is not enough [SO](../../getting_started/economy.md#so), its join transaction will reject. This deposit will be blocked until the end of the [drive](overview.md).

A group of nReliable [replicators](../../roles/replicator.md) provides a better quality of Cross Verification. Over time, any number of [replicators](../../roles/replicator.md) could join the [drive](overview.md) contract. A new [replicator](../../roles/replicator.md) that joins when the [drive](overview.md) has no file may move to the standby mode. In the general case, the time of joining the [replicator](../../roles/replicator.md) has to fill its memory with all files in the current state of the [drive](overview.md). For this purpose, the [replicator](../../roles/replicator.md) can get the root hash of the [drive](overview.md) from Blockchain and download files from another [replicator](../../roles/replicator.md) (it must put a deposit for each downloaded file). If some [replicator](../../roles/replicator.md) joins to [drive](overview.md) later, its final reward will be calculated considering its time of joining. Replicator reward depends on the time of joining the [drive](overview.md). For the earlier joining, the reward is greater.
When the minimal count of [replicators](../../roles/replicator.md) joined, the [drive](overview.md) state changes to the [**Pending**](state.md#pending) and [billing period](overview.md#billing-period) starts.

## Billing period
One of the [replicators](../../roles/replicator.md) becomes the primary, according to [consensus](../../algorithms/consensus.md), and it creates the transaction for exchange of [XPX](../../getting_started/economy.md#xpx) to [SO](../../getting_started/economy.md#so) in the [drive](overview.md) account. If the sum on balance is sufficient, [drive](overview.md) contract starts up, the state of the [drive](overview.md) is changed to [**InProgress**](state.md#inprogress), and the contract is going to start. [Replicators](../../roles/replicator.md) obtain rewards every [billing period](overview.md#billing-period ) automatically. They also initiate a new [XPX](../../getting_started/economy.md#xpx) to [SO](../../getting_started/economy.md#so) exchange after the end of the [billing period](overview.md#billing-period ).

## Interacting with drive
During the execution of the [drive](overview.md), the [owner](../../roles/owner.md) can interact with it, like on other cloud storage. It means that it can do uploading, removing, renaming, moving, downloading files, and creating/removing directories. All commands create the new [drive](overview.md) file system. Whenever an [owner](../../roles/owner.md) makes any file operation, it changes a root hash. Blockchain is storing this hash and anyone can get an actual hash. So, all [replicators](../../roles/replicator.md) can easily monitor [drive](overview.md) changes. \
The new [drive](overview.md) file system transaction has the following properties:
- ***Drive key***
- The modifiable [drive's](overview.md) key.
- ***New root hash***
- The new hash of a local [drive](overview.md).
- ***Old root hash***
- The hash of the local [drive](overview.md) before changes.
- ***Add actions***\
New added files. It is an array of structures that consists of:
    - ***File size***
    - ***File hash***
- ***Remove actions*** \
Deleted files. It is an array of structures that consists of:
    - ***File size***
    - ***File hash***

### Upload
When an [owner](../../roles/owner.md) uploads new files, it fills the addActions array, where indicates the file hashes and sizes. Blockchain automatically transfers *fileSize \* replicas* [SM](../../getting_started/economy.md#sm) for each file from the [owner](../../roles/owner.md) account to [drive](overview.md) account. These tokens are rewards of [replicators](../../roles/replicator.md) and the [owner](../../roles/owner.md) for file spreading (the [owner](../../roles/owner.md) can also return part of its tokens). The transaction can be rejected in the following cases:
- if the [owner](../../roles/owner.md) doesn't have enough money;
- Blockchain already contains the same hash;
- not enough space on the [drive](overview.md).
Blockchain marks all [replicators](../../roles/replicator.md) that they don't put deposits for a new file.

After the download of the file, [replicators](../../roles/replicator.md) must send the transaction with hashes of downloaded files. Blockchain locks [SM](../../getting_started/economy.md#sm) equal to fileSize of each file until the file is removed. The deposit is used as a guarantee of saving. When the [replicator](../../roles/replicator.md) fails verification, Blockchain transfers the [replicator](../../roles/replicator.md)'s deposits to the [drive](overview.md) account. If the [owner](../../roles/owner.md) lies and the size of the file is not equal to the size in the transaction, [replicators](../../roles/replicator.md) won't store this file (they must also delete the relevant receipts (see Fair Streaming protocol)), in this case, all [replicators](../../roles/replicator.md) will have the same files on the [drive](overview.md) and they all can pass verification.

### Delete
When the [owner](../../roles/owner.md) wants to delete files from the [drive](overview.md), it transfers the transaction with hashes of removed files. All [replicators](../../roles/replicator.md) delete these files and blockchain returns the deposits of [replicators](../../roles/replicator.md) for this file back (If [replicator](../../roles/replicator.md) didn't put a deposit for file, blockchain will not return anything). Blockchain will reject the transaction if it contains file hashes of not existing files.
### Rename/Move/Copy/Mkdir
Only the root hash is changed, because these operations don't require the transfer of data (metadata is still sent, but it requires less space).

### Contract prolongation
The [owner](../../roles/owner.md) can prolong a [drive](overview.md) with increasing or decreasing its duration. For example, a user has a [drive](overview.md) with a duration of 12 months, but the [owner](../../roles/owner.md) decides to increase the duration to 36 months. In this case there is a need to make the transaction that contains the prolong 24 months.

### End drive
The [drive](overview.md) contract is finished when the duration is over. Besides that, the [drive](overview.md) can be terminated until the deadline. It can happen when:
- The [owner](../../roles/owner.md) decides to terminate the [drive](overview.md).
- The [drive](overview.md) does not have money to start the next [billing period](overview.md#billing-period ) (in this case, the [drive](overview.md) will be terminated by [replicators](../../roles/replicator.md)).
After the end of the [drive](overview.md) duration, all [replicators](../../roles/replicator.md) return their deposits and get rewards (see [**Finished**](state.md#finished) state).
