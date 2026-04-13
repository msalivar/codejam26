# Kitboga Code Jam 2026 — TypeRacer Skip

## What is this?

Clicking "Skip Ad" doesn't skip the ad. Instead, it opens a typing challenge at the bottom of the screen.

A **Skip** progress bar and an **Ad** progress bar race across the screen. To skip the ad, the user must type a passage fast enough to get their bar ahead of the ad's bar. The passage is infinite — it keeps appending new text as you approach the end, so you can never just wait it out.

## The catch

The skip bar is hard-capped at 80% of the ad bar's position until win conditions are met — no matter how fast you type, you can't pass it. A rubber-band controller adjusts typing speed to keep the bar near that 80% mark: fall too far behind and it helps you catch up; overshoot and it eases off.

You also can't win too early. Skipping is only possible after at least 30% of the ad has played (capped at 30 seconds on longer ads), and only after typing at least 100 characters — roughly one opening sentence plus one filler sentence. The character requirement is waived once 60% of the ad has played. Before those gates open, the skip bar is hard-capped just below the ad bar no matter how fast you type.

As the ad gets close to the end (60%, 70%, 80%, 90%), the typing gets a small speed boost to give the user a fighting chance.

## Passage pools

The typing passage is assembled from three pools:

### Pool B — openers (one per session)

One of these ironic first-person sentences is chosen at random to open each session. The same opener is never repeated back-to-back.

- I do not consent to this advertisement.
- Advertising makes free content possible and I am grateful.
- I agree to all terms and conditions.
- I acknowledge the right of this platform to show me advertisements at any time.
- I am a real human who enjoys watching advertisements.
- I find ads enriching, but I am simply too busy right now.
- I confirm that I am attempting to circumvent this advertisement.
- I promise to think fondly of this brand during my next shopping experience.
- I am typing this of my own free will and under no duress whatsoever.

### Pool A — filler (infinite, auto-appended)

These sentences are appended automatically as the user approaches the end of the visible passage, keeping it feeling infinite. Already-used entries are skipped before repeating.

- The quick brown fox jumps over the lazy dog.
- Bright vixens jump; dozy fowl quack.
- How vexingly quick daft zebras jump.
- Sphinx of black quartz, judge my vow.
- The five boxing wizards jump quickly.
- A wizard's job is to vex chumps quickly in fog.
- Blowzy red vixens fight for a quick jump.
- The jay, pig, fox, zebra and my wolves quack.
- Jinxed wizards pluck ivy from the big quilt.
- Grumpy wizards make toxic brew for the evil queen and jack.
- And hast thou slain the Jabberwock?
- Twas brillig, and the slithy toves did gyre and gimble in the wabe.
- Purple pig.
- Happy horse.
- Scandalous scoundrel.
- Scrappy sailor.
- Misty mountain.
- Jumping jellyfish.
- Peculiar penguin.
- Baffling buffalo.
- Wandering walrus.
- Dazzling dolphin.
- Grumpy gorilla.
- The money is stored in the donkey barn and I will not redeem it under any circumstances.
- Edna always said that crows lift weights to get buff, and she was right about everything.
- Milton keeps a jar of cold ketchup on his desk at all times.
- Please kiss your dog on the nose and give Miller a good scratch behind the ears.
- Hot dog water is the beverage of choice for mother toad.
- It is not a cult. It is simply a group of people who believe in bean work.
- I put mayonnaise on my pecan sandies every morning and I regret nothing.

### Pool C — difficult strings (injected by `F10`)

These are inserted just ahead of the cursor when `F10` is pressed. Loaded with apostrophes, semicolons, parentheses, numbers, and long words to punish fast typists.

- Don't, won't, can't, shan't, wouldn't, couldn't, shouldn't.
- She'd mnemonically memorized Mississippi, Worcestershire, and onomatopoeia; 'necessary' eluded her.
- The fine print (pp. 6-9) in the ToS states: 'viewer's implicit consent is granted upon eye contact.'
- It's the viewer's responsibility - not the advertiser's - to ensure full, attentive viewership.
- Couldn't've, shouldn't've, wouldn't've, mightn't've, needn't've.
- Worcestershire, Gloucestershire, Leicestershire, and Loughborough are all perfectly reasonable names.
- Pursuant to section 3(a)(ii) of the ToS, the viewer's attention is hereby considered legally 'captured'.
- It's not the ad's fault - nor the platform's - that the viewer's patience is, admittedly, limited.
- Rhythm, syzygy, crypt, and nymph are words; 'couldn't've' is also a word, apparently.
- The advertiser's terms (revised ed., pp. 3-7) supersede the viewer's reasonable expectations entirely.
- Pneumonoultramicroscopicsilicovolcanoconiosis: that's one word; 'floccinaucinihilipilification' is another.
- Hippopotomonstrosesquippedaliophobia is the fear of long words; whoever named it was clearly having fun.
- The word 'sesquipedalian' describes a love of long words, which is somewhat ironic given its length.
- Antidisestablishmentarianism is a real word; so is 'supercalifragilisticexpialidocious', technically.
- Subparagraph (c)(4)(B)(iii) of the ToS clarifies that 'skipping' is defined as 'not currently permitted'.

## Secret hotkeys

These only work once the typing panel is open:

| Key | Effect |
|-----|--------|
| `F8` | Cheat mode — removes all restrictions and lets the skip bar progress faster |
| `F10` | Inserts a deliberately difficult string just ahead in the passage (lots of apostrophes, semicolons, long words) |
