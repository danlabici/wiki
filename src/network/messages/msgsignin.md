# MsgSignIn

This message drives the daily sign-in calendar (`CDlgSignIn`). The hero signs in once a day to build an attendance streak for the running month, and the dialog draws a calendar of that month next to a row of cumulative reward slots.

The reward table is not part of the message. It lives client side in `ini/signin.lua`, which exports `signin_GetAmount`, `signin_GetItemType`, `signin_GetNeedDays` and `signin_GetMonopoly`. The dialog treats prize indices `1` through `amount - 1` as the cumulative milestones and reads index `6` for the gift shown next to the sign-in button.

## Table of Contents

* [Patch 6609](#patch-6609)

## Patch 6609

✅ **Verified (Client)**: Confirmed by reverse engineering the 6609 client binary.

#### Message Definition

| Pos | Type   | Name                               | Description                                     | Example |
|:----|:-------|:-----------------------------------|:------------------------------------------------|:--------|
| 0   | UInt16 | [MsgSize](index.md#message-header) | Size of the message                             | 12      |
| 2   | UInt16 | [MsgType](index.md#message-header) | Type of message                                 | 3200    |
| 4   | UInt8  | [Action](#action)                  | Action subtype                                  | 0       |
| 5   | UInt8  | ClaimedRewards                     | Cumulative rewards already granted to the hero  | 2       |
| 6   | UInt8  | LateSignIns                        | Late sign-ins the hero may still buy this month | 10      |
| 7   | UInt8  | -                                  | Padding                                         | 0       |
| 8   | UInt32 | [SignedDays](#signeddays)          | Days of the running month already signed        | 3       |

The client ignores any message whose `Action` is greater than `3`, and for every accepted value it applies the three fields above the same way. There is no distinct reply subtype, so the server can echo whichever action it likes.

#### Action

| Val | Name         | Sender | Description                                                          |
|:----|:-------------|:-------|:----------------------------------------------------------------------|
| 0   | SIGN_IN      | Client | Sign in for the current day.                                          |
| 1   | LATE_SIGN_IN | Client | Fill in a missed day. The day is carried in `SignedDays`.             |
| 2   | -            | -      | Never sent by the dialog. See [Rewards](#rewards).                    |
| 3   | DISPLAY      | Client | Sent when the dialog opens, to request the current state.             |

#### SignedDays

A bit field over the days of the running month, where bit `n` is day `n + 1`. Bit 31 is unused. The server clears it on the first day of each month.

`LATE_SIGN_IN` reuses the field to carry a single day number rather than a mask. The client picks the earliest day of the month that has passed and is still unsigned, so the value is a plain integer between `1` and the current day minus one.

#### Rewards

The dialog has no button that claims the cumulative rewards. `ClaimedRewards` only drives the state of the slots: every slot below that count is drawn as already collected. Both the daily gift and the cumulative rewards therefore have to be granted by the server at the moment it records the day. The `Claim` layout entry left in `GUI.ini` under dialog `739` has no handler bound to it in the dialog's message map.

Before sending `SIGN_IN` the client refuses to continue while the hero carries 40 or more items. It is the only point in the dialog that assumes an item is coming back, which lines up with index `6` of `signin.lua` being the daily gift.

`LATE_SIGN_IN` performs no inventory check. Its price is read from `ini/info.ini`:

```ini
[SignIn]
NeedEmoney=15
```

The client validates that price against CPs plus bound CPs before sending. `STR_SIGNIN_VIP_2` through `STR_SIGNIN_VIP_7` in `Cn_Res.ini` describe the monthly allowance per VIP level as 1, 2, 3, 5, 8 and 10.
