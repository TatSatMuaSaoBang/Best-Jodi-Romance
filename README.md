# Datable Jodi: 1.6 update notes

## The spouse room bug
Stardew 1.6 removed `Data/SpouseRooms` and `Data/SpousePatios`. They are now the `SpouseRoom` and `SpousePatio` fields of `Data/Characters`. Your old patch edited an asset the game no longer reads, so Jodi had no room registered and the game fell back to Abigail's. The map file itself was fine (6x9, standard layer names, spouse marker tile on Paths at 3,4).

## Changes in content.json
- Format 1.27.0 -> 2.5.0 (set the same version as MinimumVersion for Pathoschild.ContentPatcher in manifest.json).
- `Data/SpouseRooms` / `Data/SpousePatios` patches removed; replaced by one `Data/Characters` patch for Jodi: `CanBeRomanced`, `SpouseRoom`, `SpousePatio`.
- Patio animation converted from the old frame string to `[frame, ms]` pairs. Old format was 100 ms per listed frame, so repeated frames were merged (e.g. "19 19 19" -> [19, 300]). Pixel offset `0 -3` -> `SpriteAnimationPixelOffset`.
- Both `Data/NPCDispositions` relationship patches (and the invatorzen.idcsm one) -> `FriendsAndFamily` via TargetField. Labels are plain text now ("eldest son"); Kent's "youngest_boy" typo is fixed to match Jodi's.
- `79480007/f Jodi 2500/x jodiMessage` dummy event -> a `Data/TriggerActions` entry (DayStarted, 2500 friendship points, AddMail jodiMessage tomorrow, applied once).
- Festival targets `Data/Festivals/fall27.xnb` / `summer28.xnb` -> no `.xnb` suffix (those patches never applied).
- Added `Strings/schedules/Jodi` `Tue.000`. The marriage_Tue schedule pointed at it but it was never defined. The wording is a placeholder from me; change it to whatever you like.

## Other files
- MarriageDialogueJodi.json: `"summer 23"` key -> `"summer_23"` (it never fired); typo fixes (sweatheart, beatuiful, apreciate, suprising, a missing period).
- SpousePatioJodi.tmx: removed the Farm map properties (Warp, Spouse, Fish, Arch, Debris, Outdoors, TileBrush) that were copied into the patio map.

## Not verified (no game or assets available here)
1. `Characters/Jodi_Beach` and `Portraits/Jodi_Beach` are EditImage patches. As far as I know vanilla has no Jodi beach assets, so if SMAPI says the target asset doesn't exist, switch those two to `Load` with full-size sheets (event 79480005 shows frame 16 of the beach sprite).
2. `jodi_exercise` has dialogue text but no animation frames in `Data/animationDescriptions`, so the 1300 exercise stop won't animate.
3. Patio tilesheet patches use Y 1264 on the four outdoors tilesheets and the map declares 400x1312. Check that vanilla 1.6 `*_outdoorsTileSheet` is still 1264 tall before the patch.
4. Wedding strings `Utility.cs.5371` / `5377` and the Kent popcorn event key `100/f Kent 750/p Kent/p Jodi` (set to null) are unchanged. Confirm in game that they still match 1.6.
5. `dumped_Guys` / `secondChance_Guys` are not defined, only the Girls versions. Add them if male farmers should get Jodi-specific lines.
6. Flower Dance: with `CanBeRomanced` she may be picked as a dance partner. If you don't want that, add `"FlowerDanceCanDance": false`.
7. manifest.json wasn't provided; it needs `ContentPackFor` Pathoschild.ContentPatcher and `MinimumApiVersion` 4.0.0.

---

# Extra dialogue and story arc (assets/Dialogue/JodiExtra.json)
Included from content.json as the last patch. About 140 new lines: everyday dialogue, her back pain, her insecurity, complaining about her situation, and a diamond reaction.

## How the story flows
1. **Phase 1, opening up (default).** 16 lines at 4/6/8 hearts: sore back, feeling invisible, doubting why you chose her, gray hairs, the boys not helping.
2. **Phase 2, pulling away.** At 9 hearts (2250 points, which means dating) she is scared: you lose 1 heart (-250 points), she sends a note (`jodiDistance`) and sets the flag `jodiPulledAway`. Her dialogue switches to avoidant and anxious lines.
3. **Phase 3, devoted.** She only sends the mountain letter after pulling away and getting back to 10 hearts. If you say yes at the river event she sets the flag `jodiDevoted` (event command `addMailReceived`, in the accept branch only), and you get a love letter next day. Her normal and marriage dialogue switch to fully devoted. If you refuse, she doesn't become devoted.

Married-life dialogue has two full overrides of the base file: a scared-but-trying set (42 lines) and a devoted set (45 lines). If neither flag is set, the original MarriageDialogueJodi.json lines apply.

## Diamond
`AcceptGift_(O)72` has three versions (default, pulling away, devoted). Each one starts a `JodiDiamond` conversation topic, so she brings the diamond up again for a week afterwards, in both normal and marriage dialogue.

## Check in game
- The `$action AddConversationTopic ...` at the start of the diamond lines is the one thing I couldn't confirm against the docs. If the follow-up line doesn't appear the next day, move the topic into a trigger action or tell me and I'll rework it.
- At 9 hearts the pull-away happens the first morning after you reach 2250 points. To test, use `debug` friendship commands in SMAPI and sleep.
- Re-testing: remove the flags with `debug` commands, or use a fresh save. The trigger actions are marked as applied, so they won't repeat on their own.
- I did not add a schedule change for avoiding you, only dialogue. Say the word if you want her to physically stay away from you during phase 2.

---

# Heart events batch 1 (assets/Events/JodiHeartEvents.json)
Included from content.json as the last patch. Four talk-heavy scenes, all using the `$q`/`$r` answer pattern from your original events, with response lines in Characters/Dialogue/Jodi.

| Event ID | Where | Needs | Scene |
|---|---|---|---|
| 79480010 | Sam's house | 6 hearts, evening | Empty nest: Jodi cries over Sam's old jacket. Staying silent gives the most friendship (+100). |
| 79480011 | Sam's house | 7 hearts, after 79480010, daytime | Vincent's family drawing with Kent rubbed out. Honest answers are rewarded, a dismissive one costs -20. |
| 79480012 | Sam's house | 8 hearts, after 79480011, Vincent home | Vincent's fever. She panics and the player can steady her or brush it off. |
| 79480013 | Mountain lake | 12 hearts, flag `jodiDevoted` | After the river: she is happy and feels guilty about it. Ends with the music box song. |

Each event queues a next-morning conversation topic (`JodiEmptyNest`, `JodiDrawing`, `JodiFever`, `JodiRiver`) through Data/TriggerActions, so she refers back to it for a week. `JodiRiver` also works in marriage dialogue.

## Check in game
- Event coordinates in Sam's house are reused from your original events (viewport 6 5, Jodi at 6 5, Vincent at 8 6). If something overlaps furniture, adjust the numbers in the fever event first.
- The wiki event docs are from before 1.6, so I only used commands that appear there or already in your events. `addConversationTopic` as an event command was not documented, so topics go through trigger actions instead.
- Friendship values in `$r` are raw points (250 per heart).
- She has no tear portrait in the files I saw. Scenes use `$s`; swap in a portrait index if you add one.
- Event 79480012 needs Vincent in the house, so it may take a few days to trigger.

---

# Children and family dialogue (assets/Dialogue/JodiFamily.json) + two more scenes (assets/Events/JodiHeartEvents2.json)
Both are included from content.json after the earlier files so their lines override the earlier marriage lines where keys overlap.

## Before the baby (married, no children yet)
6 lines per phase (scared / devoted) replacing Indoor_Day_3, Indoor_Night_4, Outdoor_3, Good_3, Good_4 and Rainy_Night_4. Scared Jodi wonders if she's too old and is afraid to ask; devoted Jodi says outright that she wants one. They apply until the flag `jodiHasKid` exists.

## During pregnancy (flag `jodiExpecting`)
10 lines per phase replacing all ten Indoor_Day / Indoor_Night keys while the flag is set. Scared: back pain, fear of being too old, the boys feeling pushed aside. Devoted: kicks, knitting a bad blanket, telling Sam and Vincent.

**Limitation:** pregnancy is not documented in any game-state query or Content Patcher token I could find, so nothing sets `jodiExpecting` automatically. Until something does, these lines never show. Options: (a) set the mail flag by hand while testing, (b) I write a small SMAPI mod that detects the pregnancy and sets and clears the flag, or (c) drop the idea and keep only before and after.

## After the baby
- `PLAYER_HAS_CHILDREN` (confirmed in the wiki) sets `jodiHasKid` / `jodiTwoKids` the morning after a birth and starts the conversation topics `JodiNewBaby` / `JodiSecondBaby` for a week.
- `OneKid_0` to `OneKid_4` and `TwoKids_0` to `TwoKids_4` have separate scared and devoted versions (20 lines), using `%kid1` and `%kid2`.

## Two more scenes (SamHouse, during the pull-away, before the lake event)
| Event ID | Needs | Scene |
|---|---|---|
| 79480014 | 8 hearts, flag `jodiPulledAway`, daytime | Torn-up letters to the player. Sliding a blank sheet of paper across the table gives the most friendship. |
| 79480015 | After 79480014, rainy evening | Sitting in silence. No dialogue choices, only a message box. |
Both stop appearing once event 79480005 (the lake) has been seen.

## Check in game
- Child-count triggers use the confirmed `PLAYER_HAS_CHILDREN` query, but the whole flow (birth night, next morning) still needs a test with a real save.
- Replacing keys means one of the random daily lines is swapped for a family line, so she won't talk about children every day.
- Not done yet: Kent's homecoming scene, anniversary event and porch talks.

---

# Dialogue expansion (assets/Dialogue/JodiDialogueExpansion.json)

Added 68 dialogue lines while keeping the existing three-stage relationship arc intact.

- **Opening-up phase:** 12 additional seasonal weekday lines, plus two Green Rain reactions and three spouse interaction reactions.
- **Pulling-away phase:** 14 seasonal weekday lines plus two Green Rain reactions and three spouse interaction reactions about Jodi's uncertainty, boundaries, friendship with Caroline, and making room for herself.
- **Devoted phase:** 14 warmer seasonal weekday lines plus two Green Rain reactions and three spouse interaction reactions about shared plans, affection, travel, and enjoying life beyond household responsibilities.
- **Marriage dialogue:** 13 additional lines across affection, rainy nights, and visits with Caroline. The scared and devoted versions remain gated by their existing flags.

These are dialogue-only additions; no new heart events, schedules, or pregnancy mechanics were introduced. In-game SMAPI testing is still needed.


---

# Past-focused, time- and item-triggered conversations (assets/Dialogue/JodiPastConversations.json)

Added a new conversation layer built around Jodi's established history and routine. The dialogue is triggered by where/when the player speaks to her, or by specific gifts, and includes player questions using Stardew's built-in dialogue choices.

## Time and location triggers
- **Tuesday exercise class at Pierre's:** questions about Caroline, the exercise group, and Jodi finding time for herself.
- **Sunday chapel visit:** conversations about Kent's deployment, waiting for letters, faith, and the emotional burden of managing the household alone.
- **Spring clinic visits:** dialogue about Vincent's checkups, being a young mother, and remembering to care for herself.
- **JojaMart on Wednesday/Friday:** the household budget while Kent was away and how Jodi learned to postpone her own needs.
- **Late evening at home:** the boys' childhood, Jodi's choice to let them enjoy childhood without many chores, and the quiet after bedtime.
- **Green Rain:** Jodi's instinct to keep Sam and Vincent safe, with different wording for the first and later years.

## Object triggers
Unique gift reactions for Largemouth Bass, Pancakes, Rhubarb Pie, Crispy Bass, Chocolate Cake, Eggplant Parmesan, Fried Eel, and Vegetable Medley; plus a birthday-gift conversation. The item reactions tie back to the dinner event, her family cooking, Sam and Vincent's childhood, and the memory of Granny's cottage. The existing Diamond story reaction remains untouched.

## Player questions and follow-ups
The new dialogue uses the built-in question/response commands and unique IDs. Chosen responses can start short-lived conversation topics, so Jodi can continue a story when the player speaks to her again. This reuses the question and conversation-topic patterns already present in the mod's heart events and gift dialogue.

These are dialogue additions; they do not change Jodi's walking schedule or add new cutscenes. In-game verification is still needed for all location/time conditions and the exact gift reactions.
