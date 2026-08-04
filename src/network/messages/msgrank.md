# MsgRank

This message serves every dynamic leaderboard in the client: the flower rankings, the fate rankings, inner strength, the "Ability Score" leaderboard (`CDlgProfessionalRank`) and its per-profession top players. A single [rank type](#rank-type) value both selects the leaderboard and, on some rank types, changes which [modes](#action) are valid.

## Table of Contents

* [Patch 6609](#patch-6609)

## Patch 6609

✅ **Verified (Client)**: Confirmed by reverse engineering the 6609 client binary.

#### Message Definition

| Pos | Type                | Name                       | Description                                    | Example |
|:----|:--------------------|:----------------------------|:------------------------------------------------|:--------|
| 0   | UInt16              | [MsgSize](index.md#message-header) | Size of the message                       | 24      |
| 2   | UInt16              | [MsgType](index.md#message-header) | Type of message                           | 1151    |
| 4   | UInt32              | [Action](#action)          | Action subtype                                  | 1       |
| 8   | UInt32              | [RankType](#rank-type)     | Selects the leaderboard                         | 80000000 |
| 12  | UInt16              | TotalCount                 | Row count of the leaderboard (`REQUEST_RANK` reply) | 42  |
| 14  | UInt8               | PageNumber                 | Requested or returned page                      | 0       |
| 15  | UInt8               | -                           | Padding                                         | 0       |
| 16  | UInt32              | EntryCount                 | Number of entries that follow                   | 1       |
| 20  | UInt32              | -                           | Padding                                         | 0       |
| 24  | `Entry[EntryCount]` | [Entries](#entry)          | Leaderboard rows                                | See below |

The client's own outgoing message is always exactly 104 bytes (`24` header bytes plus a single 80 byte entry), with every field of that one entry zeroed except for `Position`, which is unused on the request. Requests only need `Action`, `RankType` and `PageNumber` filled in.

#### Entry

Each entry is 80 bytes.

| Pos | Type   | Name              | Description                                                | Example |
|:----|:-------|:------------------|:-------------------------------------------------------------|:--------|
| 0   | UInt64 | Position          | 1-based rank position                                        | 1       |
| 8   | UInt64 | Amount            | Leaderboard score                                             | 23041   |
| 16  | UInt32 | RankType          | Only used by [`PROFESSION_TOPS`](#action); the profession's rank type | 80000002 |
| 20  | UInt32 | Identity          | [Hero ID](../identifiers.md)                                  | 1000001 |
| 24  | String | Name              | Hero name (16 bytes)                                          | Carniato |
| 40  | String | Name              | Same string, duplicated (16 bytes)                             | Carniato |
| 56  | UInt32 | Level             | Hero level                                                     | 130     |
| 60  | UInt32 | Profession        | Hero profession                                                 | 21      |
| 64  | UInt32 | LookFace          | Hero mesh                                                        | 2011    |
| 68  | UInt32 | -                 | Unused                                                            | 0       |
| 72  | UInt32 | -                 | Read only by the `30001000` rank type handler                      | 0       |
| 76  | UInt32 | -                 | Read only by the `30001000` rank type handler                      | 0       |

#### Action

The meaning of `Action` depends on `RankType`. For the rank types listed below, only modes `1` and `2` are handled; every other rank type accepts the full set.

| Val | Name             | Sender | Description                                                                 |
|:----|:-----------------|:-------|:------------------------------------------------------------------------------|
| 1   | REQUEST_RANK      | Both   | Client requests a page of `RankType`; server replies with `TotalCount`, `PageNumber` and up to 10 entries. |
| 2   | QUERY_HERO_RANK   | Server | Reports the hero's own position and score for `RankType` in a single entry.  |
| 6   | PROFESSION_TOPS   | Both   | Client requests with `RankType = 0`; server replies with one entry per profession, `RankType` in each entry selecting the profession's rank type (see [Profession Rank Types](#profession-rank-types)). |

Other values are read by dialogs that were not part of this investigation and are left undocumented here.

#### Rank Type

| Range               | Description                                                    |
|:--------------------|:------------------------------------------------------------------|
| 30000000 - 30000999  | Flower rankings                                                    |
| 60000001 - 60000004  | Fate rankings (Dragon, Phoenix, Tiger, Turtle)                     |
| 70000000             | Inner strength ranking                                              |
| 80000000             | Ability Score, overall leaderboard                                    |
| 80000001 - 80000010  | Ability Score, per profession. See [Profession Rank Types](#profession-rank-types) |
| 90000000             | Handled separately; not covered here                                |

#### Profession Rank Types

The client maps a profession's rank type back to a profession sort with the table below (`profession / 10`).

| RankType | Profession Sort | Profession   |
|:---------|:-----------------|:-------------|
| 80000001 | 1                 | Trojan       |
| 80000002 | 2                 | Warrior      |
| 80000003 | 4                 | Archer       |
| 80000004 | 5                 | Ninja        |
| 80000005 | 6                 | Monk         |
| 80000006 | 7                 | Pirate       |
| 80000007 | 8                 | Dragon Warrior |
| 80000008 | 13                | Water Taoist  |
| 80000009 | 14                | Fire Taoist   |
| 80000010 | 16                | Wind Walker  |

The Ability Score leaderboard itself is computed and stored entirely server side; the client only ever displays whatever `Amount` is sent to it.
