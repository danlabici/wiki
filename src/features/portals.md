# Portals and Passageways

> ⚠️ __WARNING__
>
> The document is written by observing client patch 5517 & leaked TQ Server Binary. There may be significant differences between client patches.  

When the player lands on a portal tile, the client sends a `CHANGE_MAP` [MsgAction](../network/messages/msgaction.md) with the x, y coordinates of the portal tile to the server. The server resolves the destination and replies with an `ENTER_MAP` [MsgAction](../network/messages/msgaction.md) containing the destination map ID and coordinates the client should update the hero to.

## Passageway (Portal Tiles)

Each [DMap](../files/formats/dmap.md) file contains an non-consecutive indexed array of `PassageInfo` (below) with the coordinates of the portal tiles. Each element of this array is called a passageway. Every time the hero moves, the client checks whether the hero's coordinates match an entry in the `PassageInfo` array, if so, sends a `CHANGE_MAP` [MsgAction](../network/messages/msgaction.md) containing the x, y coordinate of the portal tile to the server.

```
PassageInfo {
    UInt32 posX, // Portal Tile x
    UInt32 posY, // Portal Tile y
    UInt32 idx   // Passageway Index
}
```

The passageway index is used in the client-side check to see if the tile is a portal: `GetExitIndex(x, y) >= 0` (`-1` means a non-portal tile). Importantly, this index value is **not** sent in the message to the server.

## Portal (Landing Destination)

In the leaked binaries, the server loads its own copy of each [DMap](../files/formats/dmap.md) with the same `PassageInfo` array. Upon receiving a `CHANGE_MAP` [MsgAction](../network/messages/msgaction.md), it looks up the passageway index with the reported `x, y`, queries the `cq_passway` table to get the destination map and portal id, then queries the `cq_portal` table to get the destination coordinates.

The server returns the map id and destination coordinates to the client via `ENTER_MAP` [MsgAction](../network/messages/msgaction.md). The client then loads the game map and sets the x, y position of the hero.

Separating portal tiles (passageway) from destination coordinates means the server can send a player to any portal destination point without the hero necessarily walking through a portal tile.


## Security Considerations

Client DMap modification has no effect on portal resolution as the server uses its own copy. However, the reported `x, y` in the message is client-supplied and must be validated server-side to confirm the coordinates correspond to a real portal tile using the server's own map data, and confirm the hero is within a couple of tiles of the reported position.
