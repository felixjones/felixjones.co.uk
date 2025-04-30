---
layout: post
title: "My Pokémon ROM Hack"
date: 2025-04-30
description: "Wild SIDE'HACK appeared!"
image: "assets/pokegreen/side_hack.png"
---

Yes, yes even I am guilty of picking up side projects when I have a side projects going on.

At least this one is in the Game Boy family!

---

<p align="center">
<img src='{{ "/assets/pokegreen/side_hack.png" | absolute_url }}' alt="Screenshot of the original Pokemon Game Boy game, but it has been hacked to feature both a female trainer, and Game Boy shaped wild Pokemon. The text box reads: Wild SIDE'HACK appeared!" width="640"/>
</p>

A couple of months ago I was ill with a cold, and I did what most people do when they have a cold: They browse Reddit.

I stumbled upon a few posts in the Game Boy subreddits from Redditors showing off their custom made "Pokémon Green" cartridges.

For those not in the know: The original Pokémon games released in Japan in 1996 as Pokémon Red and Green, so these custom cartridges were an attempt to restore what is often seen as a lost Pokémon game.

Somehow my cold-stricken head decided to lead me into ordering all the materials I would need to create my own "Pokémon Green" cartridge.

# If a thing is worth doing

This isn't the first time I've come across custom Pokémon cartridges, but I've never had a desired to make my own.

A lot of the times it is individuals creating custom carts to avoid paying the inflated prices for legitimate, original games, but custom Green cartridges are more interesting.

The Japanese releases of Red and Green have differences to the international Red and Blue, not to mention that 1996 is before Pokémon made its international debut, so there was no international branding associated with Pokémon. Combined with the fact that there was no international "Pokémon Green Version", there's a gap to be filled.

Check-out this [BoundaryBreak](https://www.youtube.com/@BoundaryBreak) episode that explores some of the differences:

<iframe width="560" height="315" src="https://www.youtube.com/embed/RMZa3Eh-6ug?si=2wtFXC_8ST9AisrR" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## It is worth doing well

Something I've said for a long while regarding retro game projects is that I believe they're most successful as art pieces, a love letter to a bygone era in gaming.

If I were to make a retro game I would want to make a physical cartrige, a box, and a manual to "complete the package" (throw in a poster or something, too). I imagine most people would get enjoyment out of the high quality physical goodies, but as a game developer I get enjoyment at the thought of high quality software being amongst the physical goodies.

Now that I have bought into a high quality Pokémon Green display project, I thought I should follow through on my standards and make a high quality Pokémon ROM hack to go with it.

# Pokémon ROM hacks

Pokémon is quite famous in the ROM hacking scene, specifically the fact that many of the classic games have decompilation projects that make producing ROM hacks a cinch.

The pokered project is available on GitHub https://github.com/pret/pokered

## Shin Pokémon Green

Whilst the US release of Pokémon Red and Blue have been decompiled, the original Japanese release of Red and Green have not. So I needed a base: Someone who has restored the Japanese content into the pokered project.

[jojobear13's shinpokered project](https://github.com/jojobear13/shinpokered) does exactly that...And more!

<p align="center">
<img src='{{ "/assets/pokegreen/animfadeboxsmall.gif" | absolute_url }}' alt="Animated GIF fading between the different box arts of the Shin Pokémon project." width="300"/>
</p>

[The summary](https://github.com/jojobear13/shinpokered?tab=readme-ov-file#summary) lists some of the "Additional Master features that go beyond engine modifications and fixes", and a lot of these go quite far beyond what the original games provide.

I didn't actually want these additions, I wanted to make something that feels more purposefully pure to the Japanese release of Green. A sort of "What if Pokémon released internationally a year earlier?".

Luckily the project provides a [lite branch](https://github.com/jojobear13/shinpokered/tree/lite) that only includes fixes and none of the "Additional Master features".

# Are you a boy? Or are you a girl?

Shinpokered is quite a well organised project: It's possible to configure it quite extensively from just the makefile.

I noticed that it even keeps some of the code and assets from the "Additional Master features", in particular the macro `_FPLAYER` enables the code and assets for a female trainer.

I decided to break my rule of "What if Pokémon released internationally a year earlier?" and add in an option to select a girl player character.

I feel this can still be in the spirit of a "What if" because a female trainer for that generation did exist in the manga, as well as on the cover of the Japanese strategy guides, as covered in this video by [The Obsessive Gamer](https://www.youtube.com/@TheObsessiveGamer):

<iframe width="560" height="315" src="https://www.youtube.com/embed/xSeA3MzcoGY?si=EuiTWvxf2zBAOWK4" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

Let's take a look at Shinpokered's female trainer:

<p align="center">
<img src='{{ "/assets/pokegreen/bgb00026.png" | absolute_url }}' alt="Screenshot of the name entry screen for shinpokered, with the female player sprite displayed. The character is clearly an edit of the boy sprite." width="160"/>
<img src='{{ "/assets/pokegreen/bgb00025.png" | absolute_url }}' alt="Screenshot of the starting home in shinpokered, with the female overworld sprite." width="160"/>
</p>

Unfortunately this character is not the legendary Game Freak character, but is an edit of the boy trainer sprite:

<p align="center">
<img src='{{ "/assets/pokegreen/pokegreen-9.png" | absolute_url }}' alt="Screenshot of the name entry screen for pokered, with the original boy player character." width="160"/>
</p>

I looked around at other implementations of playable girl trainers, and I feel most of them miss the mark. Many of them implement the Leaf character from the Game Boy Advance games, some of them do implement the "lost" female trainer but with spritework that feels closer to modern Pokémon art styles than the original Game Boy art styles.

<p align="center">
<img src='{{ "/assets/pokegreen/4205screenshot2.png" | absolute_url }}' width="160"/>
<img src='{{ "/assets/pokegreen/avatar-feminin-yellow-legacy.png" | absolute_url }}' width="160"/>
<img src='{{ "/assets/pokegreen/8100screenshot3.png" | absolute_url }}' width="160"/>
</p>

It was becoming clear that I needed to introduce yet another playable girl ROM hack so I could create the character that I was looking for in my custom Pokémon Green ROM hack.

# Finding the Style

I noticed that the games feature multiple sprites for the rival trainer as the player progresses from the start of the game, all the way to facing the rival as the League champion in the final battle.

This provided a gradient of art styles going from a more "junior" starting trainer, an intermediate, and then the champion. I decided that the player is somewhere in the middle as an "intermediate", so that was the style I was aiming for.

<p align="center">
<img src='{{ "/assets/pokegreen/sheet.png" | absolute_url }}'/>
</p>

Looking back, my first draft reminds me of some of the early Pokémon Gold "dealer" sprites from Game Freak.

<p align="center">
<img src='{{ "/assets/pokegreen/lineup.png" | absolute_url }}'/>
</p>

The pose was inspired by this panel from the Pokémon manga (the "Yellow" arc):

<p align="center">
<img src='{{ "/assets/pokegreen/Green_Y_chapter.png" | absolute_url }}'/>
</p>

When working on digital artwork I use the "mirror" trick where you flip the image you're working on to gain a fresh look at what you're working on. It helps with identifying perspective errors, but also results in images that look good even when mirrored, and I ended up keeping the mirrored direction.

<p align="center">
<img src='{{ "/assets/pokegreen/progress.gif" | absolute_url }}'/>
</p>

There was a moment when I realised this lacked any "main character energy", which is why I added the Pokéball earrings to the character (as shown in the Yellow arc of the manga).

I requested feedback from [@sarahboev](https://bsky.app/profile/sarahboev.bsky.social) (whom I originally reached out to for this project, before taking it on myself) and they kindly pointed out that the dithering work on the hair is not characteristic of the original Pokémon games: notice that almost all trainer sprites use solid blocks of colour for the hair, and dithering is only used for textiles such as clothing.

<p align="center">
<img src='{{ "/assets/pokegreen/hair.png" | absolute_url }}'/>
</p>

Here's what the final sprite looks like on the trainer screen:

<p align="center">
<img src='{{ "/assets/pokegreen/pokegreen-17.png" | absolute_url }}'/>
</p>

# Battle Sprite

This one was more straight forward than the front sprite, although I did end up with something that looks more fitting for the second generation of games, rather than the first.

<p align="center">
<img src='{{ "/assets/pokegreen/back.png" | absolute_url }}'/>
</p>

I may have gone too far with the anti-aliasing, and I brought the character closer to the viewer which probably makes it feel more modern. Regardless, I noticed that the 2x resolution psuedo depth-of-field for generation 1 backsprites makes these details forgivable.

The pose is partly inspired from their appearance in the Gold, Silver & Crystal arc of the manga:

<p align="center">
<img height='415' src='{{ "/assets/pokegreen/Green_GSC_chapter.png" | absolute_url }}'/>
</p>

I did give up with rendering the gloves with the excuse that sprite consistency wasn't a thing even with Game Freak.

# Overworld Sprites

These were probably the most difficult sprites to work with as the canvas size is reduced to just 16x16 pixels, and the amount of colours available is reduced to 3.

<p align="center">
    <video controls>
        <source src='{{ "/assets/pokegreen/overworld.webm" | absolute_url }}' type="video/webm"/>
    </video>
</p>

I'm not 100% happy with the result.

<p align="center">
<img src='{{ "/assets/pokegreen/walk_down.gif" | absolute_url }}'/>
<img src='{{ "/assets/pokegreen/walk_up.gif" | absolute_url }}'/>
<img src='{{ "/assets/pokegreen/walk_side.gif" | absolute_url }}'/>
</p>

But I am happy versus my other attempts.

<p align="center">
<img src='{{ "/assets/pokegreen/sprite.png" | absolute_url }}'/>
</p>

Adding the earring to the side sprite very much helped tie the sprite back to the trainer graphic, however I was unable to find a good way to show the earrings in the up/down sprites.

Another feature that helped add depth was the light "shine" on the hair. I was holding off from doing this, but seeing a similar hair "shine" on the Pokémon Center attendants convinced me to try it.

<p align="center">
    <video controls>
        <source src='{{ "/assets/pokegreen/heal.webm" | absolute_url }}' type="video/webm"/>
    </video>
</p>

# Titlescreen and Diploma

Shinpokered uses the front sprite for the Diploma screen:

<p align="center">
<img src='{{ "/assets/pokegreen/pokegreen-11.png" | absolute_url }}'/>
</p>

The original code actually pulls this image from the title screen trainer graphic

<p align="center">
<img src='{{ "/assets/pokegreen/player_title_green.png" | absolute_url }}'/> <img src='{{ "/assets/pokegreen/pokegreen-12.png" | absolute_url }}'/>
</p>

Shinpokered does not display the female trainer sprite on the title screen, which is something I wanted to change.

## Titlescreen Changes

Before I reveal the girl trainer title screen sprite, I want to show something else I have changed

<p align="center">
<img src='{{ "/assets/pokegreen/logo.png" | absolute_url }}'/>
</p>

Changing the logo is probably controversial, however I wanted to keep the feeling of a "Pokémon Green" release, but without reverting to the Japanese logo.

This might look bland in comparison to the official international logo, but I did borrow the look and font from the same Japanese media that the female trainer originally appeared in:

<p align="center">
<img height='400' src='{{ "/assets/pokegreen/guide.webp" | absolute_url }}'/>
</p>

And I am happy this captures the spirit of "What if Pokémon released internationally a year earlier?"

<p align="center">
<img src='{{ "/assets/pokegreen/pokegreen-13.png" | absolute_url }}'/>
</p>

### Girl Titlescreen Sprite

I struggled with this sprite, and I'm still not entirely happy with the end result (specifically the leg pose)

<p align="center">
<img src='{{ "/assets/pokegreen/diploma_sprites.png" | absolute_url }}'/>
</p>

Here's what I settled with:

<p align="center">
<img src='{{ "/assets/pokegreen/pokegreen-14.png" | absolute_url }}'/> <img src='{{ "/assets/pokegreen/player_title_f.png" | absolute_url }}'/>
</p>

The pose is from this image of the character from the Pokémon manga:

<p align="center">
<img src='{{ "/assets/pokegreen/GreenAdventures.png" | absolute_url }}'/>
</p>

See how I replaced the Pokédex with an original Game Boy. When I saw the manga image I thought "did the Pokédex even have its design in 1996?" - the Pokédex sprites in Oak's lab are re-used book sprites and mentions pages

<p align="center">
<img src='{{ "/assets/pokegreen/bullshot.png" | absolute_url }}'/>
</p>

I ended up looking through the original Japanese instruction manuals and other media at the time of the release and found no artwork of the Pokédex as we know it.

I considered changing the Pokédex out for a book, a Pokéball, or nothing at all - but I like the personality that I've given the girl character (in contrast to the very stern-looking boy character), so I tried throwing in some other technology that appears in-game: The original Game Boy!

<p align="center">
<img src='{{ "/assets/pokegreen/gb.png" | absolute_url }}'/>
</p>

The Game Boy is actually mirrored in the final sprite, this was because I wanted to keep a distinctive A and B button visible, but flipping those ended up blending them into the hand somewhat.

### Random Titlescreen Sprite

I could have read the selected player type from the save file and display that on the title screen, but what about cases when creating a new game? I believe the title screen should remain neutral as to the contents of the save file.

Also, the changes I've made may already not be to everyone's tastes so I decided to hold back and default display the boy sprite until the player is at the Continue/New Game menu, from there pressing B will roll RNG and display either the girl or boy sprite.

That relegates the female title screen sprite to somewhat of an easter egg, and I'm okay with that.

# Other Changes

## Script

There's still some gendered language in the script, I've left that in, but I have made a small alteration in the Game Freak office:

<p align="center">
<img src='{{ "/assets/pokegreen/pokegreen-18.png" | absolute_url }}'/>
</p>

Yes technically the artist didn't draw this character, and I'm not going to insert myself into the Game Freak offices of the game - so I switched the dialogue to a nod to the history of the "lost female trainer".

---

There's a secret that I ported from shinpokered, but I won't spoil that!

## Gameplay

The shinpokered lite improvements are all here, so smarter trainer AI, fixed bugs, etc.

### One quality of life feature...

I ported one thing that frustrated me about the original games:

<p align="center">
<img src='{{ "/assets/pokegreen/pokegreen-19.png" | absolute_url }}'/>
</p>

This screenshot might not look special to players used to games after the first generation, but at the upper left is a caught Pokémon indicator. That would have been so darn useful for my complete Pokédex runs!

### Starter Pokémon

This change I think will be most controversial. To allow the player to mimic the manga scenario, as well as to introduce some variety, when playing as the girl trainer the rival will pick the Pokémon that is **weak** to your choice.

<p align="center">
    <video controls>
        <source src='{{ "/assets/pokegreen/battle.webm" | absolute_url }}' type="video/webm"/>
    </video>
</p>

This makes the girl trainer a sort of "easy" option, something that I'm okay with as generally the rival battles aren't much of a problem. I also think this is the type of strange decision that is characteristic of the weirdness of the first generation of Pokémon.

### Default names

Girl trainer ROM hacks always change the default names, shinpokered no exception (opting VIOLET, CLAIRE, and JILL).

<p align="center">
<img src='{{ "/assets/pokegreen/bgb00026.png" | absolute_url }}'/>
</p>

GREEN, LEAF, and CRYSTAL are also common choices, but I wanted default names that felt more in the spirit of the original default names for the Japanese releases of Red, Green, and Blue:

* RED/GREEN/BLUE (the edition of the game)
* SATOSHI/SHIGERU/TSUNEKAZ
* JACK/JOHN/JEAN

<p align="center">
<img src='{{ "/assets/pokegreen/pokegreen-9.png" | absolute_url }}'/>
</p>

BLUE makes sense to keep, as that is the name of the manga character in Japan (localised to Green in the west, as Blue was used for the rival).

SATOSHI/SHIGERU/TSUNEKAZ are quite male Japaneese names, so for that reason I opted to select a name of a woman known to work at Game Freak during Red and Green's development: [Atsuko Nishida](https://en.wikipedia.org/wiki/Atsuko_Nishida).

Luckily JEAN is a gender neutral name, so keeping that choice made most sense.

<p align="center">
<img src='{{ "/assets/pokegreen/pokegreen-20.png" | absolute_url }}'/>
</p>

I can see where other ROM hacks decided to use the name JILL as the rhyme of Jack & Jill provides a female counter part name to the default of "JACK". But I am happy with JEAN.

# ROM Hack Release

I am currently testing the ROM hack before I release an IPS patch for it. I will also open source my changes as a branch of shinpokered.

[Follow me on the Fediverse](https://retrodev.social/@xilefian) or [Bluesky](https://bsky.app/profile/xilefian.retrodev.social) to get notified of the release.
