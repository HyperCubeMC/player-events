# Player Events

[ ![GitHub release](https://img.shields.io/github/v/release/ByMartrixx/player-events) ](https://github.com/ByMartrixx/player-events/releases/latest)
![GitHub Workflow Status](https://img.shields.io/github/workflow/status/ByMartrixx/player-events/build)

<a href='https://www.curseforge.com/minecraft/mc-mods/fabric-api'><img src='https://i.imgur.com/Ol1Tcf8.png' width="150"></a>

**Note: this mod is server side only and won't work on clients**

A Fabric mod that executes and sends configurable commands and messages respectively on certain
events triggered by a player, such as Dying, Joining a server, Killing another player, etc.

The config file is located in the config directory (`config/player_events.json`) and looks like this:

```JSON
{
  "death": {
    "actions": [
      "${player} just died!"
    ],
    "broadcast_to_everyone": true
  },
  "first_join": {
    "actions": [
      "Welcome to the server ${player}! Remember to read the rules"
    ],
    "broadcast_to_everyone": false
  },
  "join": {
    "actions": [
      "Welcome ${player}",
      "/say Hello ${player}"
    ],
    "broadcast_to_everyone": true
  },
  "kill_entity": {
    "actions": [
      "${player} killed ${killedEntity}"
    ],
    "broadcast_to_everyone": true
  },
  "kill_player": {
    "actions": [
      "${player} killed ${killedPlayer}",
      "F ${killedPlayer}"
    ],
    "broadcast_to_everyone": true
  },
  "leave": {
    "actions": [
      "Goodbye ${player}!",
      "/say Hope to see you soon ${player}"
    ],
    "broadcast_to_everyone": true
  }
}
```

On the JSON file you can declare, under the `actions` array on each `<event>` object, what is going
to be sent and/or executed on that event. You can also set these messages to be sent only to the
player by setting `broadcast_to_everyone` to `false`, but this won't work with events like `leave`
(because the player isn't in the server anymore).

Every event has a `${player}` token, and each instance of this token will be replaced with the player
that triggers the event. Other events have extra tokens that work the same way.
Most (if not all) tokens have properties that can be accessed with something like `${player.name}`.
Here is a list of all the properties:
- `display`
  Entity's display name, the one you see in the player list/chat. Example: "[Team blue] Tom421"
- `uuid`
  Entity's UUID
- `x` `y` `z`
  Entity coordinates
- More coming soon™

**Supports [color codes](https://minecraft.gamepedia.com/Formatting_codes#Color_codes) too!**

Use `/pe reload` or `/player_events reload` to reload the mod config.

You can use `/pe test <event>` or `/player_events test <event>` to test the actions on a specific
event, or use `/pe test *` to test every event.

### 2.1.2 supported events
* `death` - Executed when a player dies.
* `first_join` - Executed when a player joins for first time.
* `join` - Executed when a player joins.
* `kill_entity` - Executed when a player kills an entity. Extra tokens:
    * `${killedEntity}` - the killed entity.
* `kill_player` - Executed when a player kills another player. Extra tokens:
    * `${killedPlayer}` - the killed player.
* `leave` - Executed when a player leaves.

### Troubleshooting
If you get an error when initializing the server or when an action should be executed, here are
steps on how to solve it.
- "Invalid JSON syntax in the config file": This is most likely caused by having a command with
  unescaped double quotes. To fix it:
  1. Check what the exception says in the line below. It will indicate exactly where the error is.
  2. Escape any double quotes inside the action with backslashes like `"` -> `\"`
- "Invalid escape sequence"
  This is caused because of an improperly escaped escape sequence like `\n` or `\u`. To fix it,
  just add a backslash before the escape sequence backslash, for example `\n` -> `\\n` or `\\u` -> `\\\\u`

## Developing
This part is for mod developers that would like to use the mod api.

### Compiling

1. Clone or download the repository
2. On a command prompt run `gradlew build` to compile the mod. (If you only need the API files,
   you can run `gradlew api:build` instead) . You'll find the compiled `.jar` files under
   `<repository>/build/libs` and `<repository>/api/build/libs`

### API
#### Adding the API as a dependency of your mod
Add the following snippet to your `build.gradle`:
```groovy
repositories {
    maven {
        url 'https://maven.bymartrixx.me'    
    }
}

dependencies {
    // Using the version from the gradle.properties
    modImplementation "me.bymartrixx.player-events:api:${project.player_events_api_version}"

    // Directly setting the version
    modImplementation "me.bymartrixx.player-events:api:2.0.0"
}
```
Add this snippet to your `gradle.properties` if you aren't directly setting the version to the build.gradle file:
```properties
player_events_api_version = 2.0.0
```

Also, add this snippet to your `fabric.mod.json` if you want your mod to depend on the api:
```json
{
  "depends": {
    "player_events_api": ">=2.0.0"
  }
}
```

#### Events
* `death` - `me.bymartrixx.playerevents.api.event.PlayerDeathCallback.EVENT`
* `first_join` - `me.bymartrixx.playerevents.api.event.PlayerFirstJoinCallback.EVENT`
* `join` - `me.bymartrixx.playerevents.api.event.PlayerJoinCallback.EVENT`
* `kill_entity` - `me.bymartrixx.playerevents.api.event.PlayerKillEntityCallback.EVENT`
* `kill_player` - `me.bymartrixx.playerevents.api.event.PlayerKillPlayerCallback.EVENT`
* `leave` - `me.bymartrixx.playerevents.api.event.PlayerLeaveCallback.EVENT`

#### Note
The package `io.github.bymartrixx.playerevents.api` has been moved to `me.bymartrixx.playerevents.api`,
and the package `io.github.bymartrixx.player_events.api` has been removed. Classes under the package
`io.github.bymartrixx.playerevents.api` have been deprecated and will be removed in a next release.

#### Using the events
Example snippet
```java
public class FooMod implements DedicatedServerModInitializer {
    public void onInitializeServer() {
        PlayerDeathCallback.EVENT.register((player, source) -> {
            // Do something
        });

        PlayerKillEntityCallback.EVENT.register((player, killedEntity) -> {
            // Do something
        });
    }
}
```
