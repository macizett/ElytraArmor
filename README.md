# ElytraArmor

Combine a chestplate and an elytra at an anvil into a single item that protects like the armor and flies like the elytra. Paper 1.21.11, no commands for players, no client mods.

![Merged chestplate icons: vanilla on the left, merged on the right, for every material](docs/icon-preview.png)

## Download

Grab the latest `ElytraArmor-1.0.0.jar` from [Releases](../../releases), drop it into `plugins/`, and restart the server. That is the whole setup: the first start writes the config, and the resource pack that draws armor and wings together is sent to players automatically.

Use a restart rather than `/reload`. If you are upgrading from an earlier build, delete the old `config.yml` so the new keys are written, since existing files are never rewritten.

| Requirement | Detail |
| --- | --- |
| Server | Paper 1.21.11 or a fork of it, such as Purpur |
| Java | 21, which the server itself already requires |
| Client | Vanilla; the resource pack is sent by the server |
| Dependencies | None, the Kotlin runtime is bundled in the jar |

Spigot and vanilla servers are not supported: the plugin uses Paper's data component and command APIs.

## What it does

A player wearing a merged chestplate keeps full armor protection and can glide. One anvil click turns a chestplate and an elytra into that single item; a grindstone takes them apart again.

The chest slot holds exactly one item and a vanilla client cannot be talked out of that, so the plugin does not add a second slot. Instead the chestplate itself is given the ability to fly, which means armor points, enchantments, durability and repairs all stay ordinary chestplate behaviour.

## How to use it

**Combining.** Put chest armor in one anvil slot and an elytra in the other, in either order. The result costs 100 levels by default. Vanilla refuses the pickup when the player cannot pay, exactly as it does for any other anvil work.

Enchantments from both items end up on the result, and where they overlap the higher level wins. A Protection IV chestplate plus an Unbreaking III, Mending elytra gives one item with all three.

**Splitting.** Put a merged chestplate alone in a grindstone. The armor comes back with everything it has, and the elytra comes back with the durability it had when it was merged, stripped of enchantments the way a grindstone strips anything else. No experience is paid out, and recombining costs the full price again, so splitting is an escape hatch rather than a routine.

**Flying.** Jump and glide as normal, and fireworks boost the flight exactly as they do with an elytra. Flight wears the chestplate down at the rate set in the config, and Mending and Unbreaking apply because it is ordinary item durability.

Players need no commands and no permissions. The only command is `/elytraarmor reload`, which needs `elytraarmor.reload`, default op.

## Configuration

`plugins/ElytraArmor/config.yml`, reloaded with `/elytraarmor reload`.

| Option | Default | What it does |
| --- | --- | --- |
| `anvil.enabled` | `true` | Whether chestplate and elytra combine at an anvil at all |
| `anvil.cost-levels` | `100` | Price of combining, in levels; the vanilla 40 level ceiling is lifted for it |
| `grindstone.enabled` | `true` | Set to `false` and merging is permanent |
| `glide-durability-per-second` | `0.5` | Durability a second of flight costs, as a decimal; vanilla elytras use `1.0` |
| `mode` | `anvil` | `automatic` skips the economy entirely and lets every chestplate fly on its own |
| `appearance` | `pack` | `pack` draws armor and wings together, `wings` shows elytra wings only, `armor` shows the armor only |
| `resource-pack.auto-send` | `true` | Whether the plugin hands the pack to players as they join |
| `resource-pack.url` | hosted copy | Direct download URL of the pack |
| `resource-pack.sha1` | matching hash | 40 character SHA-1 of that file |
| `resource-pack.required` | `true` | Whether a player who refuses the pack is let in |
| `resource-pack.prompt` | short line | Text shown in the client's pack dialog |
| `lore-line` | `Z elytrą` | Line added to merged armor so it is recognisable; empty for none |
| `messages` | Polish | Chat messages, in MiniMessage format |

Fractional wear is applied as a running balance rather than a dice roll, so `0.5` costs a point every second second and `0.25` every fourth. A netherite chestplate holds 592 durability, which at `0.5` is roughly twenty minutes of continuous flight before Mending is even considered.

Combat damage is never touched: a hit taken in the same tick marks the item damage that follows it as combat, not flight wear.

## The resource pack

Without a pack the client can draw either the armor or the wings, never both, because a vanilla chestplate and a vanilla elytra each define only one equipment layer. The pack adds the missing combination: for each of the seven chestplate materials an equipment asset that draws the armor layer and the elytra wings together, and an item model that shows the chestplate sprite with a pair of wings beside it. Armor trims and leather dye keep working.

The plugin sends it on join by itself, alongside any pack the server already sends through `server.properties` rather than replacing it. Nothing has to be configured.

Not interested in a pack at all? Set `appearance: wings` (or `armor`) and the plugin stops sending anything, at the cost of the player showing only one of the two.

## How it works

Since Minecraft 1.21.2 any item can carry the `minecraft:glider` data component, which is what actually makes an elytra fly. The plugin puts that component on a piece of chest armor and gets out of the way.

That is why so little of the behaviour is custom. The server revalidates gliding every tick against the same component the vanilla elytra uses, fireworks attach to a gliding player without caring what is on their back, and durability, Mending and Unbreaking are whatever the armor piece already did. The plugin only intervenes in four places: the anvil recipe, the grindstone split, the rate at which flight wears the item, and the look of the merged item.

It works with any item wearable in the chest slot, including custom armor from other plugins, although the resource pack only carries models for the seven vanilla materials. Anything else falls back to the plain wings look.

## Limits and gotchas

- One durability pool, the armor's own. There is no separate elytra wear, and no scenario where the wings break off and leave the chestplate behind. If the chestplate breaks, the whole thing breaks.
- With `resource-pack.required: false`, a player who refuses the pack sees a missing model on merged armor. With `true`, they cannot join without it.
- Flight wears armor that combat would otherwise wear. At the default rate a netherite chestplate trades roughly twenty minutes of flight for its full durability bar, which Mending then pays back.
- Splitting returns the elytra without its enchantments, and recombining costs the full level price again.
- Armor trims are kept and drawn correctly, but a merged item no longer uses the vanilla item model, so other plugins that key off `minecraft:item_model` will see this plugin's id instead.
- `mode: automatic` makes elytras pointless by design: every chestplate flies, no anvil, no grindstone, no cost.
