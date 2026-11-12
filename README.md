# Ultima V: Warriors of Destiny, taken apart

Ultima V (Origin Systems, 1988) was written for the Apple II and then rewritten, ported and re-ported for most of the computers of its day. This is what came out of reading the original disks of nearly all of them, starting with the Apple II: how the program is built, what it hides, where it is broken, and how each later version was made. Nothing here is guesswork unless it says so. Findings marked "read in the code" were read from the machine code and not run; "seen in emulation" means watched in the running game in MAME. Ultima V and everything in it belong to Electronic Arts, which still sells the game (the MS-DOS version, on GOG); this write-up contains no part of it beyond short quotations.

## Sources and method

The Apple II analysis rests on 4am's flux-level images of the original disks of two releases; a third release, from April 1988, survives only as a crack and was used for comparison. Every other version came from public archive dumps, original-disk dumps where they exist (MS-DOS 1.14 and 1.16, Amiga, FM Towns). Findings were checked in the running game in MAME.

## How the Apple II game is built

Ultima V on the Apple II is hand-written 6502 assembly in about sixty separately loaded modules, swapped in and out of a 64K or 128K machine by a small loader of Origin's own called DINKEYDOS (credited in the game as "Operating System by Henry Roberts"). It moves a 4 KB kernel into the language card at boot: a jump table of fourteen calls (load, run, save, find a volume, read or write a block) and a disk driver. The game proper lives in the second language-card bank at the same addresses, so every disk access switches banks and pauses the music.

A start-up module identifies the machine, loads a resident library and plays the intro. The saved party goes straight into zero page, and one of three engines runs: one for Britannia and the Underworld, one for the 32 towns, castles, keeps and dwellings, one for the eight dungeons. Combat is its own module, kept in auxiliary memory on a 128K machine so a fight needs no disk access. Commands are overlays loaded on demand and cached by letter.

## The game loops

Each place in the game has its own engine with its own main loop: outdoors, towns, dungeons and combat, with the title menu in front and conversations inside a command. A turn is one pass of a loop that spends game time. All of them share one routine for time, light and food and one for the view, but they differ in how much time a move costs and in what else moves. The Apple II and MS-DOS versions run the same cycle with a few differences, listed at the end.

### Start-up and the title menu

1. The machine is identified (Apple: model and 128K memory; MS-DOS: the video adapter named on the command line), and fonts and drivers are loaded.
2. The opening plays: logo, flames, signature, then the scripted story. Any key skips it; holding J at start goes straight to the menu.
3. The menu waits for a key. Left alone, it replays the story and comes back.
4. Create, Transfer, Introduction and Acknowledgements run and return to the menu. The sixth item is "Activate Music" on a 128K Apple and "Return to the View" on MS-DOS.
5. Journey Onward loads the saved game ("No active game" if there is none) and hands over to the engine for the party's location: outdoors, a town, or a dungeon. On the Apple II each engine is a separate program loaded from disk, and on a 128K machine the combat engine is parked in auxiliary memory.

### Outdoors (Britannia and the Underworld)

1. Darkness and the party: a darkness tile without a light spell blacks out the view; an all-asleep party skips to the monsters; an all-dead party ends the game.
2. The midnight moongate is checked, the view and status are redrawn, and the waterfall is tested.
3. A key is read. When sailing and no key comes, the ship drifts with the wind instead, which costs time.
4. The command runs. A step checks the terrain, and slow ground costs 2 or 4 extra minutes and gives the monsters extra moves. Commands that spend no time skip the rest of the turn.
5. End of turn: 2 minutes pass, then bridge trolls, swamp, fire and lava, the Sacred Quest gate, Underworld earthquakes, party upkeep, rough-sea damage and the falls.
6. The monsters: none under Time Stop; every other turn under Quickness or on a horse or carpet. A spawn roll compares a terrain number (higher in the Underworld and at night) with a roll of 1 to 30. Each monster next to the party attacks, the rest move; ships and sea monsters fire from up to 4 tiles away.
7. A monster that reaches the party prints "Attacked!" and starts combat. Entering a town or dungeon ends the loop.

### Towns, castles, keeps and dwellings

1. On entry the map and its 32 object slots load, every person is placed where their schedule puts them for the hour, and the Shadowlords are checked.
2. The party and the moongate are checked, then a key is read. A drunk party hiccups half the time, raises the alarm and staggers.
3. The command runs. Digits next to the harpsichord play notes. Walking off the map ends the loop.
4. End of turn, only if time was spent: 1 minute passes, then lamps at 05:00 and 20:00, sleepers wake (1 in 16), hazards (trapdoors, swamp, fire), party upkeep and the door timer (doors close after 4 turns).
5. Horses wander and people move by schedule and mode, with at most one new path search per turn. A hostile person attacks, a guard arrests, and someone who wants to talk starts a conversation.

### Dungeons

1. On entry the frame is drawn, the dungeon's wall set loads, and the level's one wandering monster is picked.
2. A minute passes before the key is read, not after (none under Time Stop, every other pass under Quickness).
3. While waiting for a key the 3D view is redrawn, with step and drip sounds.
4. The command runs: move, turn, climb, search, cast.
5. End of turn: sleepers wake (1 in 16), the wandering monster moves and starts combat if it reaches the party, a room cell starts the room's fight, field and trap cells act (sleep, poison, fire, bomb, pit), and party upkeep runs.

There is no wind, no moon and no townsfolk below ground; light comes only from torches and the light spell.

### Combat

1. The 11 by 11 field is drawn and both sides counted.
2. The loop scans the 32 combatant slots. Each has a wait counter; when it reaches zero the combatant acts and the counter is reset to 36 minus its dexterity, so a dexterity of 30 acts every 6th pass and 18 every 18th.
3. A party member takes a command; a monster (or a charmed party member, or one holding the Sword of Chaos) is run by the monster logic: wake up, special ability, attack, move, or flee. Nothing acts under Time Stop.
4. Terrain effects and a recount follow each action.
5. No party left: "BATTLE IS LOST". No enemies left: "VICTORY!", and the field stays open until the party walks off it.

### Conversations

A conversation runs inside the Talk command. The person's attitude decides what happens: no response, a conversation, a merchant, fleeing, hostility, or a guard's toll. A conversation greets, then loops on "Your interest?": an empty line or BYE ends it, built-in words (NAME, JOB, BYE and the like) are tried first, then the person's keywords, and anything else gets "I cannot help thee with that." Answers carry codes that can join the party, take gold, give an item, ask the Avatar's name, change karma, call the guards, or ask a question with its own answers. Talking probably costs no game time.

### Shared each turn

- Time and light: minutes and hours, the meal hours, lamps, the Shadowlords' move at midnight, days and months, and the light level from the hour, torches and spells. Quickness halves the minutes.
- The view: tile and figure animation, a change of wind (1 in 64), moongates rising and sinking, the 11 by 11 view with line of sight, and ambient sounds. It also runs while the game waits for a key, so the world animates in real time while the clock moves only on a turn.
- Party upkeep: poison costs 1 hit point; at 06:00, 12:00 and 18:00 each living member eats, and with no food the party is "Starving!"; spell effects count down.

The order is the same in every engine: checks at the top of the loop, the view, the key and command, then time, hazards and upkeep, then the monsters or townsfolk. Time therefore passes after the player acts and before the world answers, except in dungeons, where the minute comes first.

### Side by side

| Concern | Outdoors | Town | Dungeon | Combat |
|---|---|---|---|---|
| Time per move | 2 minutes (4 or 6 on slow ground) | 1 minute | 1 minute, before the key | 1 minute per 10 actions (MS-DOS); none found on the Apple II |
| Food | yes | yes | yes | no |
| Light | day and night, darkness tiles | lamps at 05:00 and 20:00 | torch and spell only | from the map |
| Wind | yes, moves ships | changes, no effect | no | no effect |
| Moons and moongates | yes | moongate check | no | no |
| Who else moves | monsters and ships | townsfolk on schedules, horses | one wandering monster | combatants by 36 minus dexterity |
| How a fight starts | spawn roll by terrain | hostile person or guard | the wanderer, or a room | (is the fight) |
| How it ends | entering a place | walking off the map | climbing out | victory, defeat or leaving the field |

### Where the Apple II and MS-DOS loops differ

- Combat order: MS-DOS scans the combatant slots upward from 0, the Apple II downward from 31, so on the Apple creatures in high slots act before the party in the same pass.
- Returning from combat: MS-DOS returns into the engine that started the fight; the Apple II reloads that engine and starts it from the top.
- The combat clock: MS-DOS advances time every 10 actions; no time step was found in the Apple combat loop.
- Town turn order: MS-DOS applies hazards, then upkeep; the Apple II upkeep, then hazards.
- Sleepers outdoors: the Apple II wakes them 1 time in 16 each turn; no such step was found in the MS-DOS outdoor loop.
- Built-in conversation words: the Apple II tests 12, the MS-DOS version 34.
- Structure: MS-DOS runs every loop inside one program; on the Apple II each engine loads the next one from disk.

## The copy protection

The protection is in the sector labels. On tracks 3 to 34 of the Program disk the sectors are numbered 11h to 20h instead of 0 to Fh, and every address field there is followed by three extra zero bits. DINKEYDOS reads the address-field epilogue twice, the second time one instruction later without waiting for the drive, and subtracts 11h from the sector number only when the second read still sees the protected byte, which only the extra bits make possible. A normal disk with standard epilogues takes the other branch and keeps its sector numbers, so a plain sector copy simply works. The scheme stopped nibble copiers, not sector copies.

## EASUX: the hidden memory editor

The Official Book of Ultima (Shay Addams, first edition 1990; quoted here from the 1992 second edition, pages 71-72) describes "a mini-editor that could read or write to any memory location within the game, or call up any subroutine from outside the Ultima program". Its example: with the right addresses "you could instantly elevate all eight virtues to 99", and Garriott says that "with patience, you could literally rewrite the entire game" with it. The book says it was left in for customer service: if a player's game crashed, ORIGIN could talk them through a fix over the phone instead of asking for the disks back. It closes: "The five-letter password required to activate this feature remains top-secret, but it follows the pattern described above", meaning FLIPFLOP, the word the book has just told readers to yell outdoors. The editor is still in the shipped game, identical in both original releases.

Outdoors, Yell `EASUX`. Hold the Solid-Apple key to read a byte (type a four-digit hex address); hold Open-Apple to write one (address, then value, repeating while the key is held); hold both to call a subroutine at the address. In MAME the party's karma was read as 69, written as 42 and read back as 42, using nothing but key presses. Read as "EA SUX" the password fits Origin's open hostility to Electronic Arts at the time (the game's swear-word list ends with "ELECTRONIC ARTS"), but nothing in the game states that reading.

The password was not found in any source checked: a web search, the Ultima Codex Easter-egg page (which cites the same book pages for FLIPFLOP but not the editor), Eeggs, a search of the rec.games.computer.ultima.series newsgroup, and the two Easter-egg collections posted there in 1999 and 2000 (the first lists FLIPFLOP for the Apple II, neither mentions the editor). The Cutting Room Floor's Apple II page (last edited 9 May 2023) lists two Easter eggs, Toshi's credit and FLIPFLOP, and not the editor. The editor exists on the Apple II only; no other version carries it.

## Other secrets and jokes

All of these were confirmed in the program bytes.

- Lord British is named "Rodent of Unusual Size" in the Apple combat engine's name table, on the tile of his crowned figure, while the real giant rat there is just "Giant Rat". No town places that tile in a fight, so honest play never shows the name. The MS-DOS version calls him "Lord British" in the same table; the rodent joke itself survives in both, on the rat's Look text and a Keep sign reading "BEWARE RODENTS OF UNUSUAL SIZE".
- FLIPFLOP, Ken Arnold's prank on Garriott according to the Official Book, turns every tile and letter upside down when yelled outdoors, and the 3D view in a dungeon. With the music running it also mirrors the screen, a 180-degree turn.
- Toshi Morita's credit dissolves pixel by pixel when Ctrl-^ (Shift-Ctrl-6) is pressed on the credits screen. The book tells it as an arrow he planted pointing at his name before he left Origin, which the team turned into an erasure; it gives the key as Shift-Control-1, which does not match the code.
- Gravestones read "POOR RICHARD BURIED ALIVE TRYING TO FINISH ULTIMA V" and remember a Lebling who "KICKED THE DRAGON ONCE TOO OFTEN".
- The arms dealer's sell screen draws a box reading "SOFTWARE PIRACY CAUSES IMPOTENCY" and covers it straight away. A skeleton described as "an unfortunate software pirate" turns up one time in 256.
- An unused 49th spell is "Frotz" (Infocom's Enchanter) and an unused class is "Adept of Woznir". Both are in the MS-DOS version too.
- Every guild lists a grappling hook behind a key that answers "That offer was for a limited time only." There is a fourth guild, The Guild of the Broken Lock, and a second ship priced at 87,000 gold.
- Ctrl-@ prints the karma, location, level and position in hex. Ctrl-^ with both joystick buttons held at the start of a game casts spells without reagents.
- The wishing well turns "Ferrari" or "Porsche" into a fast horse.
- The swear-word list ends in "ELECTRONIC ARTS", but the Apple code only ever tests the first seven entries, and the MS-DOS loop stops one entry short of the end, so the last word is never checked in either.
- The shipped save files are leftover test games. The first release's party is led by a character called "Shithead"; the second release's party starts in the Underworld.

## Bugs and exploits

- Reagent duplication in Mix (Apple II, seen in emulation). Reagent counts are stored in BCD but compared with the request in binary and subtracted with no floor. With 15 garlic, asking for 20 mixtures of a spell that needs garlic left the party with 95 garlic and 23 mixtures. The MS-DOS line keeps counts in binary and refuses a short mix; the Commodore 64 has the same bug, instruction for instruction (read in the code).
- Character creation asks the wrong question about 4.5 percent of the time (seen in emulation): one entry in the gypsy's question table points at the Honesty and Sacrifice card instead of Compassion and Sacrifice, so Honesty can score twice. Both Apple releases.
- The end scroll's playing time is counted from day 1, month 4, year 139 without borrowing, so it shows one year too many when the game ends before month 4 (read in the code).
- Read in the code on the Apple II: experience gains add a binary 1 to a BCD value; An Sanct treats location 20h as a dungeon; a scroll acts on a stale party member; the dungeon command reports untrapped chests as trapped, breaks a key on every one and reads the wrong member's dexterity; the bar's food cap stores 63h; the arms dealer ignores the Leather Helm when selling.
- Two exploits in the IBM-line code, confirmed by running the FM Towns version's own code: Thrud's gift of a weapon can be collected again and again (each one sold for 95 to 136 gold), and Jeremy's keys can be taken repeatedly for 3 karma each. The Apple version's equivalents were not tested.

## Two families of code

Ultima V exists in two separate code bases.

The Apple II code went to the Commodore 64: the same module names, text stored the same way. The C64 has no EASUX, keeps FLIPFLOP only in the dungeon, where it is the only word Yell recognises, and has the Mix bug.

Everything else descends from the IBM PC version, a separate program in C by a different team (credited in the FM Towns version as John Fachini, Herman Miller, Ed Nelson, Cheryl Chen and Steve Meuse). The ports all use its data files and none carries the Apple extras.

| Version | Date | What it is |
|---|---|---|
| Apple II, three releases | February, March, April 1988 | The original. The second release rewords disk prompts, fixes a handful of bugs and corrects "Valor" to "Valour"; the April release fixes two decimal counters and the dungeon potion colours |
| Commodore 64 | 1988 | The Apple code |
| MS-DOS 1.14 | June 1988 | The IBM program, Microsoft C 5.0, overlays linked with Phoenix PLINK86 |
| MS-DOS 1.16 | October 1988 | A fix release, relinked from mixed objects (see below) |
| MS-DOS hard-disk rebuild | January 1992 | On the Ultima Collection CD: 1.16 code with the disk-swap prompts and the copy-protection start-up removed |
| Atari ST | files dated February 1990 | Ported by Binary Vision from an IBM source state between 1.14 and 1.16; Lattice C 3.04.01 |
| Amiga | 1989 | Keith Jackson, DMA Systems; the ST port's game code with the ST's compiler |
| Sharp X68000 | files dated May 1990 | Pony Canyon, calling itself "1.17J"; Japanese text; GCC |
| FM Towns | May 1992 | Fujitsu, ported from 1.16; MetaWare High C |
| NES | 1992-93 | A new and much simpler program by Origin, scrubbed for Nintendo |

Two two-disk 3.5-inch MS-DOS editions turned out to be relinks of 1.14 (June 1988) and 1.16 (October 1988, from "Ultima: The Second Trilogy"). A TOSEC entry labelled "Ultima V (1989)(Origin)(beta)" for the Amiga is a different, unidentified game: its world has places called Chryse, Havilah and Laodicea and none of Ultima V's words.

The NES version keeps the plot and the virtues and drops nearly everything personal: every in-joke, the swear list, the Rodent, Frotz, and every mention of God, Hell, churches, wine or ale. Its credits are Origin's own staff (Alan Gardner and Robert Herrick, design by Dr. Cat, music by Martin Galway).

## Data formats

None of the game's data uses a standard format.

On the Apple II:

- Conversations are Huffman-coded, one 512-byte raw disk block per character (block F0 minus n on the town side), decoded with a tree held inside the TALK module. A record holds the name, description, greeting, job and bye texts, keyword and answer pairs, and labelled question sections, with embedded codes for the Avatar's name, joining the party, payment, giving an item, karma up or down, calling the guards, and branches on whether the character knows the Avatar or on karma.
- Character creation and the story use the same Huffman scheme with their own trees.
- Maps live in raw disk blocks outside the file system. The outdoor world, Britannia and the Underworld alike, is 256 chunks of 16 by 16 tiles in blocks 152-279 of its disk side. The 32 by 32 settlement maps sit in blocks F8 plus 2n. Each dungeon is one block: eight levels of 8 by 8 cells, the cell type in the high nibble. Combat battlefields are 256-byte records in blocks 90h-97h, and dungeon rooms are 256-byte records with their own creatures and triggers.
- Tiles: SHAPES holds two banks of 256 tiles of 16 by 16 pixels, stored one byte per page so the drawing code can index by tile number.
- Pictures: the intro pictures are run-length coded and filled column by column from the right; the dungeon wall pieces use an LZ format with a 256-byte ring.
- FONT3 is a 95-glyph proportional font.
- The saved party is a 48-byte record loaded straight into zero page.
- Music: 15 tunes in MUFF format, played through Mockingboard-family cards or a Passport MIDI card.

In the IBM line:

- Pictures (the .16 files) start with a 4-byte little-endian unpacked size, then LZW codes read least significant bit first, 9 to 12 bits wide, with 100h to reset and 101h to end. Unpacked, a file holds a 16-bit count and 32-bit offsets, then for each picture a 16-bit width in bytes, a 16-bit height and three bit-planes. The X68000 uses the same format at twice the resolution.
- X68000 tiles: TILES.CH holds 256 tiles of 32 by 32 pixels, 384 bytes each in three bit-planes; NPC.CH holds 256 sprites of 512 bytes, the same three planes plus a transparency mask.
- X68000 conversations: a 512-byte header, little-endian word offsets, and text XOR-coded with a key of 90h plus the conversation number minus one.

## Which compiler built each version

Each IBM-line version was built with an identifiable compiler, and the code each compiler generates leaves fingerprints in the game.

MS-DOS 1.14 was compiled with Microsoft C 5.0. MS-DOS 1.16 is a mixed relink: the source files Origin changed were recompiled with Microsoft C 5.1, every other file kept its 5.0 object, and the whole was linked with the 5.1 runtime library. The tell is the order of local variables on the stack. Both compilers sort them by a checksum of the variable's name, but 5.1 drops a carry term, so the same names land in a different order in exactly the recompiled files. Where the compiler changes inside one overlay, a boundary between Origin's source files must lie there, so the release also shows how the source was divided.

The changed files show what Origin fixed for 1.16: camping no longer kills party members who are still poisoned, leaving a companion at an inn no longer corrupts the active-member index, a found dungeon bomb trap is cleared from the map, searching a table can no longer duplicate two special items, and the Shard's text spells "Shadowlord" correctly. One change went wrong: the moongate sound in 1.16 still reads a fixed address that pointed into 1.14's sound table, which had moved.

The overlay layout PLINK86 produced can be read in full from ULTIMA.EXE. The addresses below are from 1.14; 1.16's root code is 16 bytes longer, so its overlay areas sit a little higher.

- Everything runs in one 64K code segment, small model. Root code occupies CS:0000-72CF, Phoenix's overlay manager starts at CS:72D0 and the overlay table at CS:7770.
- The table is a 14-byte header, then one 16-byte record per overlay (the last two words of each record give the paragraph base and end of the next overlay; the header gives the first's), then the file names, then 164 entry thunks of 12 bytes each: a far call into the manager, the overlay number, and a far jump to the target. Callers reach a thunk with a plain near call; the manager loads the overlay, the far jump lands in the same code segment, and the function's near return goes straight back to the caller.
- Four overlay areas, nested by size, each starting where the largest overlay of the area before it ends: town, outdoors, dungeon and intro at CS:81C0; flames, NPCs, combat, Blackthorn, look, dungeon look, outdoor helpers, shops and the ending at CS:A280; SJOG, commands, spells and talk at CS:BF60; the second spell file, status screens, combat helpers, the inn and shipwright shops and fonts at CS:E170.
- Overlay files are raw code linked at their area base, with no header and no relocations; INTRO.OVL alone has a 16-byte prefix. The last table record is DATA.OVL, the data segment.

The Atari ST and Amiga programs were built with Lattice C 3.04.01 for the Atari ST (MetaComCo, 1987), with no options. The Amiga program uses the ST compiler's A6 stack frames, which no Amiga compiler uses because A6 holds library bases there, and shares most of its game code with the ST byte for byte. The likely story, though nothing states it, is that the ST port came first and the Amiga was built from it.

The FM Towns program was built with MetaWare High C, the compiler in Fujitsu's own High C kit; a dozen small code idioms show the game used a release slightly earlier than the kit's 1.73.

The X68000 program was built with GCC 1.x. Its code is that of Mariko's published X68000 GCC 1.42 without the output changes Mariko documented as his own, and the machine description in that source names the earlier port it was based on, "last modified for Human68K on SHARP X68000 by Masaki KONDO, Oct 29 1989", which is most likely the compiler Pony Canyon used; no copy of it was found. Sharp's own XC compiler, the obvious candidate, does not fit: XC 2.0 promotes short arithmetic to 32 bits where the game's code works in 16.

## How each version was built

A compiler can be identified by compiling a single function with it and comparing the result with the original, with relocation fields (the addresses the linker fills in) masked: the right compiler and settings give identical bytes. What follows is each version's images, format, compiler, settings and layout.

### Apple II

- Images: flux-level WOZ images of the original disks of two releases, eight sides each. The two differ in more than the transfer utility. A third release, 12 Apr 1988, survives only as a cracked copy.
- Disk format: every side is standard 16-sector 6-and-2 GCR, volume 1, 35 tracks, and all 560 sectors per side decode with good checksums. On the Program side only, tracks 3-34 carry address-field sector numbers 11h-20h instead of 0-Fh. Read as physical 0-Fh, those sectors match a plain DOS-order sector copy. A plain copy has lost the protection in the format, not in the code. Some WOZ readers drop the sector that straddles the track start.
- File system: every side is a ProDOS volume of 280 blocks. The kernel (DINKEYDOS, loaded by the boot block at 2000) reads only type 06 files from the volume directory. Seven data sides end in blocks marked used but owned by no file, read by block number. The Program side has one such block, 279.
- Compiler: none. The game is hand-written 6502 code with no symbols: 61 modules, 207,938 bytes of program files.
- Memory map: the kernel sits in language-card bank 1 at D000-DFFF. The game uses bank 2: MAIN.SUBS at E000, HTXT at D800, UPDATE.HIMEM at D000-D7FF. STARTUP runs at 8000, TEMP.SUBS at 1580, ENTER.PLAY at B000.
- Copy protection: the kernel's address-field reader at D944 subtracts 11h from the sector number when the latch still holds AB on a second, un-waited read. It is a sector-number trick, not a code change, so the original and a plain copy run the same code.
- Emulation: MAME 0.289 boots the original WOZ images and passes the check.

### MS-DOS 1.14 and 1.16

- Images: flux-level dumps of original, uncracked 1.14 and 1.16 disk sets.
- Disk 1 reads 712 of 720 sectors. The missing eight are the protection track (cylinder 6, side 0) and hold no file. Disks 2-4 are plain 360K FAT12.
- Program files: ULTIMA.EXE, DATA.OVL and 23 overlay files, 205,008 bytes in v1.14 and 204,880 in v1.16. There are no symbols.
- Compiler for v1.14: Microsoft C 5.0. Proof: the 1987 run-time library string, and four library modules (CRT0, ULDIV, LDIV, STDENVP) that match only the 5.0 small-model library. MSC 5.0 also places string literals in a CONST segment, word-aligned.
- Compiler for v1.16: it is a mixed relink. Thirteen source files were recompiled with Microsoft C 5.1. The rest are 5.0 objects. The library is 5.1's. So the compiler differs from file to file.
- Assembly: the hand-written parts assemble with Microsoft Macro Assembler 5.10.
- Host: both compilers run under DOSBox 0.74.
- Settings: small model, full optimisation (/Ox) plus /Or, and Pascal calling convention (/Gc: arguments left to right, callee pops, upper-case names without underscore). There are no stack probes, as /Ox gives. A function with parameters that ends in a plain return was declared cdecl.
- Code-generation facts:
    - Local-variable slots are decided by the names, not the declaration order. The name is read as 16-bit little-endian words, summed with wraparound. In MSC 5.0 the key is the low byte plus the high byte, mod 16, which includes the carries out of the low byte. In 5.1 the key is just the sum of the bytes, mod 16. Slots go from [bp-2] downwards in ascending key. Within one key the later declaration comes first.
    - Origin declared each table used by one function as a static inside that function. The original then keeps its values in registers across calls. File-level externs reload them. `const` changes nothing.
    - Unsigned compares written as `(unsigned)v == C` give `mov ax,reg; cmp ax,imm`.
    - An explicit `return` or `break` inside a branch decides the exit layout of the whole function.
    - Identical tails are cross-jumped.
    - A dense switch is `add ax,ax; xchg ax,bx; jmp cs:[bx+table]`. Sparse switches are compare chains.
    - The right operand of a relation is evaluated first. Constants above 32767 without a cast are long.
    - String literals are emitted in source order and not merged. A text printed from several places was one array.
    - memset and memcpy are compiler intrinsics.
    - Stored statics keep their values across calls.
    - 5.1 differs from 5.0: hidden down-counters (`mov cx,N` and `loop`), `[bx+si]` indexing, reloaded global bytes in loops.
- Compiler memory: long identifiers exhaust the optimiser. With 20-character names a function lost loop optimisation that 6-character names kept. Silent loss of loop optimisation also happens when too much code precedes a function in one file. Origin's names were short.
- Linking and layout (PLINK86-style, Phoenix overlay manager):
    - Everything is one 64K code segment, small model.
    - Root code (including 15 runtime library modules from 017E, with Origin's code from 087E) occupies 0000-72CF in v1.14.
    - The Phoenix overlay manager sits at 72D0 and the overlay table at 7770.
    - The table has a 14-byte header, then 24 records of 16 bytes (the last is DATA.OVL, ended by FFFF). Then come the file names and 164 entry thunks of 12 bytes each.
    - A thunk is a far call to the manager, a word holding the overlay number, then a far jump to the function. Callers reach a thunk with a plain near call.
    - There are four nested overlay areas, each as large as its biggest member. In v1.14 the bases are 81C0, A280, BF60 and E170 (A: TOWN, MAINOUT, DUNGEON, INTRO; B starts at the end of the largest in A; C at the end of COMBAT; D at the end of SJOG).
    - Overlay files are raw code linked at their area base, with no header and no relocations. INTRO.OVL alone has a 16-byte prefix.
    - DATA.OVL is last, right after area D. It holds the initialised data and begins with the run-time copyright string.
    - Data order: the overlays' string blocks first, in table order, then the root modules.
    - v1.16 differs: root 0000-72DF, manager at 72E0, table at 7780, areas at 81D0, A290, BF80 and E1E0. DATA.OVL is 48,480 bytes plus the 16-byte prefix. The start-up block at 81D0-86EF is encrypted.
    - The EXE header is 2,048 bytes with 409 relocations. Near calls inside one object carry no fixup.
- Written in C: about 68.5 percent of each program (140,446 of 205,008 bytes in v1.14; 140,421 of 204,880 in v1.16). The rest is the run-time library, the EXE header, data, and about 129 hand-written assembly routines: graphics, speaker and file wrappers, the start-up decryptor, the overlay manager and the root graphics module. The assembly shows itself by `xor r,r`, `push si; push di; push ds`, `clc`/`stc` returns and far calls through a pointer.

### Atari ST and Amiga

- Images, ST: sector images of the three disks (Boot, Intro, Play), so any protection is not preserved. The program is AUTO/U5.PRG, a GEMDOS executable with 14,754 relocations and no symbols. Data sits inside the text segment.
- Images, Amiga: Software Preservation Society IPF dumps of the original Boot and Dungeon disks (Mindscape, 1989).
- The Amiga file system is custom. Each 512-byte sector starts with a 4-byte header: next track and sector, or FFFF plus the byte count minus one in the last sector. That leaves 508 payload bytes per sector, and files are chains of these links.
- The Amiga program is a raw 68000 image loaded at 800h, with no relocation. Layout: hand-written platform layer, then variables, then the C code, then library and data.
- The two ports share one compiler and one game source. The port was made from a DOS state between v1.14 and v1.16.
- Compiler: Lattice C 3.04.01 for the Atari ST (MetaComCo, 1987).
- Host: the compiler runs in the Hatari 2.5 emulator with its built-in EmuTOS and writes GST object files.
- Settings: no options at all.
- The earlier Lattice 3.03 generates different code: it caches stored variables in registers and narrows char compares to byte compares.
- Code-generation facts:
    - Plain C in old K&R style only: no prototypes and no `const`.
    - Only the first 8 characters of an identifier count.
    - int and pointers are 32 bits and plain char is signed. Arguments are pushed as longs, right to left.
    - Locals take 4-byte slots in declaration order from the frame pointer, a char one byte, and a local struct goes below the scalars. Dead stores and unused locals stay.
    - Declaration initialisers are ordered differently from statements.
    - Calls inside one file are `bsr.w` and calls to another file are `jsr abs.l`, so the call type reveals Origin's source-file boundaries.
    - Sparse switches are (value, branch) records in reverse source order, and two or three cases become a compare chain tested in reverse. Dense switches go through a table of branches.
    - File-private statics get direct memory operands; Origin declared them static, in address order.
    - Operand order changes register choice. Separate `return` statements and separate `if` forms are not interchangeable with `&&`.
    - Odd cases: `--n; if (n == 0)` differs from `if (--n == 0)`, and `x = 0; y = 0;` differs from `x = y = 0`.
- Written in C: 75.4 percent of the ST program, including its platform layer (disk loader, screen, windows, text, timing) and Lattice's library up to 041718.
- Hand-written assembly: about 208 routines (20,944 bytes) and the start-up code. Tables sit inside the code, and Lattice's library from 041718 on is not C.

### FM Towns

- Images: a dump of the Japanese CD (one data track and two audio tracks). The game is U5_E/WORRIORS.EXP (English) and WORRIORJ.EXP (Japanese).
- Format: Phar Lap P3 flat executables, unpacked, base 0, entry 39700h, no symbols. The code is built from DOS v1.16. The two language builds are the same source in a different link order.
- Layout: game modules first, in DOS link order, then the platform layer. The Fujitsu libraries start at 2F440, the High C runtime at 37054 and start-up at 39700. Code ends at 3DD38 and data runs to 89FD8.
- Compiler: Fujitsu High C V1.7, which is MetaWare High C 1.73. It shipped in Fujitsu's High C Compiler Multimedia Kit V1.7 (L12) and Kaihatsu Kit (L13). HCD386.EXE is a real-mode DOS program. It runs under DOSBox and writes Phar Lap Easy OMF-386 objects. The manuals are Shift-JIS.
- Settings: none.
- Code-generation facts:
    - int is 32 bits. Variables the game reads with 16-bit accesses are `short`. Examples: character hp and experience, gold, food, colours, shop tables.
    - Char is unsigned by default, so the DOS signed char compares carry a `(signed char)` cast.
    - There is no `far`. Including the standard library header turns abs into a call.
    - Locals take frame slots in declaration order, first declared nearest the frame pointer. Names do not matter and nothing is sorted by size.
    - Registers edi, esi, ebx go in declaration order, parameters first. Ties depend on declaration order. The `register` keyword raises priority.
    - `x = f(); if (x...)` differs from `if ((x = f())...)`.
    - A file's own tables are addressed from the start of that file's data block, in order of first declaration. A header extern of the same name placed first moves the data.
    - Zero bytes after a function mean an object file ends there.
    - Short globals passed to int-pointer routines go through a temporary.
    - The game widens char arguments at the call, as old-style (K&R) definitions do.
- A dozen small code idioms in the game are ones High C 1.73 does not produce:
    - abs as `and; jge; neg`
    - 32-bit `% 4` and `% 16` as and/jge/sub
    - a missing redundant `and r,r`
    - `dec eax; je` for compare with 1
    - a known-zero register never reused

    The game was probably built with an earlier High C (V1.4 or V1.6), which is not publicly available. The Fujitsu libraries, the sound library and the High C runtime are library code.

### Sharp X68000

- Images: two .dim disk images (Pony Canyon, 1989) of unknown provenance. Each is a 256-byte DIM header followed by a Human68k FAT12 volume with 1024-byte sectors and 77x2 tracks. U5.X is a Human68k executable of 249,440 bytes with no symbols and no overlays. Text is 206,656 bytes, with 11,808 relocations.
- Code layout: the game's C runs to 2D600. The hand-written platform layer (IOCS, graphics, sound, kanji text) is 2D8A4-31A8C. Sharp's XC library starts at 31A8C. The version string is "1.17J". The code follows DOS v1.16, with one gem-map function in v1.14 form.
- Compiler: GCC 1.x with -O, in the form of Mariko's X68000 GCC 1.42 ("GNU C 1.30 Tool#2", GCC142.LZH from retropc.net, source published) with the HAS 3.09 assembler; both run under the run68 emulator. The game was probably compiled with Masaki Kondo's earlier 1989 X68000 GCC, which is not online. Sharp's XC 2.00 and 2.11 generate different code.
- run68 needs more than its default 1 MB of memory; its ini parser reads the first digit of the memory setting twice, so 11 is the effective maximum.
- Settings: K&R C, `short` variables, -O. Mariko's output differs from the game's in places he documented as his own changes. The game has:
    - zero into a data register is `clr.l`/`clr.w`, not moveq
    - a zero push is `clr.l -(sp)`
    - shifts by one stay `asl #1`, not add
    - stack pops are `adda.w #n,sp`

    Mariko's source therefore gives the game's compiler once those changes are undone.
- Differences after assembly:
    - Calls are `jsr abs.l` where GCC writes branch-subroutine calls.
    - Constant pushes are `pea abs.l` and string addresses are absolute.
    - Strings sit in the code just before the function that uses them.
- Code-generation facts:
    - Many DOS ints are `short`. A short parameter is read from the low word of its 4-byte slot.
    - The game's compiler honoured prototypes, converting arguments to short.
    - Top-tested loops are `if (c) do ... while (c)`, since plain `while` is rotated.
    - Sparse switches are linear compare chains in reverse source order.
    - Word arithmetic on a call result goes through a short temporary.
    - A ternary choosing a string argument is two calls in an if/else.
    - Register choice follows statement and operand order.
    - Frame slots follow declaration order.
- The game's compiler also differs from Mariko's in lazy parameter loads, 16-bit short sums, zero-register reuse and word compares on address registers. The platform layer is hand-written assembly.

## What the X68000 version changed

The X68000 port is more than a translation. It adds a romaji-to-kana input method for naming the character, with a mode bar and function-key shortcuts. Its combat hit test uses an accumulator instead of a random roll (read in the code). Its conversation engine is rewritten, with new control codes, conversation files stored XOR-coded, and no word wrap. A number of English messages from the IBM version are routed to an empty routine and never appear. Screen coordinates are doubled throughout, and the tiles and pictures were redrawn at twice the resolution.

## Open questions

- Whether the EASUX password was ever printed anywhere: in a magazine, a hint book or an Origin newsletter.
- Why the Official Book gives Shift-Control-1 for Toshi's egg when the code wants Ctrl-^.
- Who ported the X68000 version. The Ultima Codex names Riverhill Soft; the sound driver on the disk is "INFINITY SOUND DRIVER", copyright 1989 INFINITY Co., Ltd.
- Whether a PC-88 or PC-98 version existed. The X68000 program carries a font named for the PC-98, which suggests a shared Japanese code base, but no dump was found.

## Thanks

To 4am for the flux images of the original Apple II disks; to the archivists behind the archive.org, TOSEC, redump and Software Preservation Society dumps of the other versions; to retropc.net for keeping Mariko's GCC available; to Shay Addams's The Official Book of Ultima for the hint that started the editor hunt; to the MAME team; to Mariko for publishing the source of the X68000 GCC; and to the u5d project (github.com/wonst719/u5d) and raldi/u5-decomp, for their public work on the MS-DOS version.
