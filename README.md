# Chains - Skillchains reborn

## Authors

* Based on original work from [Ivaar for windower](https://github.com/Ivaar/Skillchains)
* First porting on Ashita v4 by [Sippius](https://github.com/Sippius/Ashita-v4-addons/tree/main)
* Maintain by MultiFr3d for ashita v4.16

## Usage
### Active Battle Skillchain Display.

**Designed for and tested on retail.**

Displays a text object containing skillchain elements resonating on current target, timer for skillchain window and a list of weapon skills that can skillchain based on the weapon you have currently equipped.

### Commands
The following commands may be used to adjust the window position.

    /chains visible       -- displays text box - click and drag it to desired location

    /chains move <x> <y>  -- reposition the window to the defined x, y coordinates

    /chains scale <value> -- set font scale

The following commands toggle the display information.

    /chains color   -- colorize properties and elements

    /chains pet     -- smn and bst pet skills

    /chains spell   -- sch immanence and blue magic spells

    /chains weapon  -- weapon skills

### Notable features that have changed
- Display configuration is not currently stored per job and not all options are supported.
- Information is displayed through IMGUI instead of a font object.
- BLU and SCH spells only display when the associated abilities are active.
- Skillchains are calculated on each render cycle for the active target rather than once for each target. This allows for real time updates based on active abilities and equipped weapon at the cost of additional workload.
- Because the addon has been recoded and logic changed, it may not act exactly the same.

### Noteable features that are the same
- Display format (with and without color)
- Support for spells under Immanence, Chain Affinity and Azure Lore
- Support for Pet and NPC weaponskills
- Support for Aeonic weapons and ultimate skillchains (limited testing)

### Known Issues/limitations
- Chain Affinity only works with BLU main
- Azure Lore duration is hard coded to 30 seconds (no check for relic hands)
- Cannot detect when another player cancels their spell abilities
- Aeonic testing is limited due to lack of weapon to test with

## AscensionXI changes

This is the AscensionXI server's fork of MultiFr3d's chains. The upstream
addon assumes retail skillchain rules; AscensionXI deviates in two places,
and each would otherwise show the player something untrue.

**Formless Fists (Monk).** On AscensionXI a Monk's Formless Fists flags the
next hand-to-hand weapon skill: it stores a mantra and forms no skillchain.
The server drops the skillchain properties from that one use, and the action
packet is byte for byte an ordinary weapon skill, so unpatched chains opens a
skillchain window that does not exist - and, worse, treats the weapon skill as
a fresh opener when a real window was already standing. The fork watches for
the effect on the job ability's action packet, the same way it already watches
Immanence and Chain Affinity, and when the flagged weapon skill lands it
leaves every window untouched. This covers anyone in your alliance, not just
you. The buff bar is a fallback for your own flag if the addon was loaded
after you armed it.

**Spellchain (Red Mage).** AscensionXI gives Red Mage a Spellchain ability
that reuses the Immanence effect, so a Red Mage main holding it now gets the
element list that was previously shown to Scholars only.

**Text scale (`/chains scale`).** Upstream's scale command multiplied the
window's width and left the font alone, so a value below 1 clipped the
display instead of shrinking it, and a value above 1 only padded it. It now
scales the text: the window auto-sizes to the scaled content and treats
350 x scale as a minimum width, so bigger text widens the box rather than
being cut off by it. The idea came from a request raised on NerfOnline's
Horizon fork of this addon; the implementation here is our own, and none of
that fork's data is used - its skillchain properties are rebalanced for
HorizonXI and disagree with retail on seventeen weapon skills.

**Onslaught boss weakness.** When an Onslaught boss is engaged, the server
draws a weakness and tells the party about it in chat. The fork does not read
chat for it, and it does not poll: shortly after it loads, and again after
every zone-in, it sends one request over the AscensionXI addon channel
(packet 0x1E0, op 0x90, the same channel DLAC uses). That request subscribes
you, and from then on the server pushes a snapshot whenever the fight's state
changes: the boss's id, the weak element, the procs already banked, and the
working opener/closer pairs it computed from every present party member's
usable weapon skills (players only - trusts are never on the roster). Those
frames never reach the retail client. `/chains refresh` asks again by hand.

While the boss is targeted the fork opens a second window of its own (drag it
where you want it; the position is saved) with the weakness and who opens with
what and who closes with what to break it:

    Fallen: Wasp Sting  >  Abraxis: Raging Fists  =  Liquefaction

Pairs you are part of come first, then pairs between two others, then
self-chains; higher chains first within each. If the server found no pair on
the present roster the window falls back to what you alone can contribute:
the weapon skills you can open with and the closer property a partner then
needs, and the ones you can close with and the opener property that must be
standing. The ordinary chains window is unchanged except that, on the boss,
the results that break the weakness are listed first and marked, and the rest
are greyed out - they are still real skillchains, just not the one the boss
wants. The window closes when the boss falls or you leave the zone; a wipe
keeps it, because the server keeps the weakness.

On a server that does not answer the op the fork stops asking until you
change zones or `/chains refresh`. `/chains debug` prints each snapshot as it
arrives.

The three server-rules changes above do nothing on a retail-rules server: the
Formless Fists effect id is never sent, no other server hands a Red Mage
Immanence, and no other server answers the Onslaught op.

## Acknowledgments
All credit goes to Ivaar for the original skillchains implementation which was used as the tempalte for how to accomplish the desired results and how to deal with some of the corner cases.

Special thanks to Atom0s and Thorny. Many of their addons are used as examples of how to accomplish various tasks.
