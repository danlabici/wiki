# MsgServerInfo

This message is sent by the server early in the login sequence, before the `ANSWER_OK` talk message in [MsgTalk](msgtalk.md). It sets a global server type flag on the client which controls whether the Shopping Mall (Conquer Points) shop is accessible.

## Table of Contents

* [Patch 5517](#patch-5517)

## Patch 5517

✅ **Verified (Client)**: Confirmed by reverse engineering the client binary.

#### Message Definition

| Pos | Type   | Name                                | Description         | Example |
|:----|:-------|:------------------------------------|:--------------------|:--------|
| 0   | UInt16 | [MsgSize](index.md#message-header)  | Size of the message | 8       |
| 2   | UInt16 | [MsgType](index.md#message-header)  | Type of message     | 2079    |
| 4   | UInt32 | [ServerTypeFlag](#server-type-flag) | Server type         | 0       |

#### Server Type Flag

| Value | Effect                                                                                                                                                        |
|:------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 0     | Sets global: `g_nServerType = 2000`. Shopping Mall shown if player meets eligibility (Level >= 70 \|\| RebirthCount > 0 \|\| EMoney > 0 \|\| BoundEMoney > 0) |
| 1     | Sets global: `g_nServerType = 2001`. Shopping Mall disabled.                                                                                                  |
| > 1   | No effect. `g_nServerType` will not change.                                                                                                                   |
