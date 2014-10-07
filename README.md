# Team Fortress 2 for Node.js

This module provides a very flexible interface for interacting with the [Team Fortress 2](http://store.steampowered.com) Game Coordinator. It's designed to work with a [node-steam](https://github.com/seishun/node-steam) instance.

**This module is still under active development. The current version is a development preview. While it's mostly stable, features may break or change at any time without notice.**

# Setup

First, install it from npm:

	$ npm install tf2

Require the module and call its constructor with your node-steam instance:

```js
var Steam = require('steam');
var TeamFortress2 = require('tf2');

var client = new Steam.SteamClient();
var tf2 = new TeamFortress2(client);
```

To initialize your GC connection, just launch TF2 via node-steam normally:

```js
client.gamesPlayed([440]);
```

node-tf2 will emit a `connectedToGC` event when the game coordinator connection has been successfully established. You shouldn't try to do anything before you receive that event.

# Enums

There are some enums that are used by various methods and events. You can find them in `enums.js`.

# Properties

There are a few useful read-only properties available to you.

### haveGCSession

`true` if we're currently connected to the GC, `false` otherwise. You should only call methods when we have an active GC session.

### itemSchema

After `itemSchema` is emitted, this is the object representation of the parsed items_game.txt file. Before that point, this is undefined.

### backpack

After `backpackLoaded` is emitted, this is an array containing the contents of our backpack. Before that point, this is undefined.

# Methods

### Constructor(steamClient)

When instantiating your node-tf2 instance, you need to pass your active Steam.SteamClient instance as the sole parameter, as shown here:

```js
var tf2 = new TeamFortress2(steamClient);
```

### setLang(localizationFile)

Call this method with the contents of an up-to-date localization file of your chosen language if you want localized events to be emitted. You can find the localization files under `tf/resource/tf_[language].txt`.

You can call this at any time, even when disconnected. If you get an updated localization file, you can call this again to update the cached version.

### craft(items[, recipe])

Craft `items` together into a new item, optionally using a specific `recipe`. The `recipe` parameter is optional and you don't normally need to specify it. `items` should be an array of item IDs to craft.

### trade(steamID)

Sends an in-game trade request to `steamID`. The other player must be playing TF2 currently. Listen for the `tradeResponse` event for their response. If they accept, node-steam will emit `sessionStart` and you can start the trade with [node-steam-trade](https://github.com/seishun/node-steam-trade).

### setStyle(item, style)

Sets the current `style` of an `item`. The `item` parameter should be an item ID, and the `style` parameter is the index of the desired style.

### setPosition(item, position)

Sets the `position` of an `item` in the backpack. The first slot on page 1 is position 1. `item` should be an item ID.

### deleteItem(item)

Deletes an `item`. The `item` parameter should be the ID of the item to delete. **This is a destructive operation.**

### wrapItem(wrapID, itemID)

Wraps the item with ID `itemID` using the gift wrap with ID `wrapID`.

### deliverGift(gift, steamID)

Sends a `gift` to a recipient with a `steamID`. The recipient doesn't need to be playing TF2. `gift` should be the ID of the wrapped gift item.

### unwrapGift(gift)

Unwraps a `gift`. The `gift` parameter should be the ID of a received wrapped gift item.

### useItem(item)

Generically use an item. The `item` parameter should be an item ID.

### sortBackpack(sortType)

Sorts your backpack. `sortType` is the ID of the type of sort you want. I don't know which sort type is which code, so you'll have to figure that out for yourself.

# Events

### connectedToGC

- `version` - The current version reported by the GC

Emitted when a GC connection is established. You shouldn't use any methods before you receive this. Note that this may be received (after it's first emitted) without any disconnectedFromGC event being emitted. In this case, the GC simply restarted.

### disconnectedFromGC

- `reason` - The reason why we disconnected from the GC. This value is one of the values in the `GCGoodbyeReason` enum.

Emitted when we disconnect from the GC. You shouldn't use any methods until `connectedToGC` is emitted.

### itemSchema

- `version` - The current version of the schema as a hexadecimal string
- `itemsGameUrl` - The URL to the current items_game.txt

Emitted when we get an updated item schema from the GC. node-tf2 will automatically download and parse the updated items_game.txt and will emit `itemSchemaLoaded` when complete.

### itemSchemaLoaded

Emitted when the up-to-date items_game.txt has been downloaded and parsed. It's available as `tf2.itemSchema`.

### itemSchemaError

- `err` - The error that occurred

Emitted if there was an error when downloading items_game.txt.

### systemMessage

- `message` - The message that was broadcast

Emitted when a system message is sent by Valve. In the official client, this is displayed as a regular pop-up notification box and in chat, and is accompanied by a beeping sound.

System messages are broadcast rarely and usually concern item server downtime.

### displayNotification

- `title` - Notification title (currently unused)
- `body` - Notification body text

Emitted when a GC-to-client notification is sent. In the official client, this is displayed as a regular pop-up notification box. Currently, this is only used for broadcasting Something Special For Someone Special acceptance messages.

Notifications have a valid and non-empty `title`, but the official client doesn't display it.

**This won't be emitted unless you call `setLang` with a valid localization file.**

### itemBroadcast

- `message` - The message text that is rendered by clients. This will be `null` if you haven't called `setLang` with a valid localization file or if the schema isn't loaded.
- `username` - The name of the user that received/deleted an item
- `wasDestruction` - `true` if the user deleted their item, `false` if they received it
- `defindex` - The definition index of the item that was received/deleted

Emitted when an item broadcast notification is sent. In the official client, the `message` is displayed as a regular pop-up notification box. Currently, this is only used for broadcasting Golden Frying Pan drops/deletions.

### tradeResponse

- `response` - The response code. This is a value in the `TradeResponse` enum.

Emitted when a response is received to a `trade` call.

### backpackLoaded

Emitted when the GC has sent us the contents of our backpack. From this point forward, backpack contents are available as a `tf2.backpack` property, which is an array of item objects. The array is in no particular order, use the `inventory` property of each item to determine its backpack slot.

### itemAcquired

- `item` - The item that was acquired

Emitted when we receive a new item. `item` is the item that we just received, and `tf2.backpack` is updated before the event is emitted.

### itemChanged

- `oldItem` - The old item data
- `newItem` - The new item data

Emitted when an item in our backpack changes (e.g. style update, position changed, etc.).

### itemRemoved

- `item` - The item that was removed

Emitted when an item is removed from our backpack. The `tf2.backpack` property is updated before this is emitted.

### craftingComplete

Emitted when a craft initiated by the `craft` method finishes. Currently this event doesn't include any specific data.