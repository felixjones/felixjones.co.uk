---
layout: post
title: "That one time I made a Minecraft mod"
date: 2022-08-05
description: "I tried to rip-off ComputerCraft, but Command Blocks ruined everything."
image: "assets/ruby/minecraft-earth-features-addon_18.png"
---

# That one time I made a Minecraft mod

I remember being quite impressed by the [ComputerCraft](https://www.computercraft.info/) mod (which is apparently open source now?), and it inspired me to attempt my first Minecraft mod that basically ripped off ComputerCraft and just did things the way past-Felix thought things should be done.

Whilst I was very familiar with Lua at the time, the first programming language I had begun to toy with was Ruby, and I had recently figured out how to embed CRuby (the full-fat Ruby interpreter, mruby wasn't around at the time), so I thought I'd be really clever and make a mod that adds a "Ruby Block" to Minecraft, which would literally be a block of Ruby that executed Ruby script.

I still think Ruby would thematically be a hilarious choice for a Minecraft script language (but if you were to ask me with my sensible hat on, I'd give a different answer these days). I love the little pun of "code-block" crossing with "Minecraft block" to make it a literal block of Ruby code.

Rubies have a bit of history in Minecraft (does [RubyDung](https://minecraft.fandom.com/wiki/RubyDung) count?) where they were going to be the original currency for [villager trading](https://minecraft.fandom.com/wiki/Emerald#Trivia) (according to the unofficial Minecraft Wiki), so I guess this was me attempting to revive them without the ore block, but as something totally unexpected — in hindsight this is the type of out-of-the-box interpretation that tends to be most appreciated by my colleagues in Mojang Studios when it comes to Minecraft ideas, "Rubies aren't an ore or gem, they're something totally different".

## What went well?

I don't have a good memory of what the mod making process was like back in the day, but my grasp of Java 8 was good from my experience working with Android, and I remember the tutorial I followed "how to add a block" got me to the point of having a block of "Ruby", from there it was very easy to get JNI interfacing with my Ruby DLL and have a little "Hello World" activating, printing in the chat too. I think I had it run every tick? Didn't get around to setting it up to activate on redstone.

I also got Ruby snippets working in the chat, so anything written in chat would be executed as Ruby. I had ideas of a scripting console (Quake style drop-down!) so one could script stuff in the game with Ruby.

## What didn't go well?

I identified a big flaw in what I was doing: I'd need to ship compiled binaries for my mod, and I only really had the means to debug a Windows DLL at the time, so immediately Mac and Linux fans would be missing out on my genius Ruby block.

## Why didn't I finish it?

The week after I began development, Mojang announces the Command Block.

![The Command Block]({{ "/assets/ruby/JebAdventureModeControlBlockDev1.webp" | absolute_url }})

At the time it appeared to me that the Command Block covered a significant chunk of use cases I had in mind for my Ruby block, and the rest of the use cases the Command Block didn't cover was already achieved in ComputerCraft, so I didn't have anything that would be impressive enough on its own, at most I was replacing Lua with an even less popular language.

The Command Block stomping on my idea stuck with me long enough that I stood in line at the Command Block panel of Minecon 2015 amongst a load of farting kids to ask the panelists for their thoughts on what would have been different if Command Blocks ran a full scripting language like Lua.

I never asked the question because all the time was taken up by the kids in front of me asking "What's your favourite block" multiple times over.

## Learnings

I revisitted the Ruby powered Quake console part of this mod a number of times in later projects. From my experience of typing crude Ruby into Minecraft chat, I had realised that Ruby was oddly awesome for game commands.

A lot of the syntax felt like traditional game console commands, rather than typing out a program.

I wrote [an article](http://altimit.systems/articles/mruby%20Android) about my revelations regarding Ruby syntax for game commands, and reading that back (it's cringe, please don't read it) makes me still think Ruby would have been a great choice for game scripting.

This is one example from that page for teleporting the player if they are an admin:

```ruby
Position.set x: 100, y: 110, z: 40 if Player.access == :admin
```

If I were to imagine how this would have looked if I finished my Ruby block mod it would probably be:

```ruby
Player.teleport x: 100, y: 110, z: 40 if Player.team == "admin"
```

That's just one idea. You could probably get the syntax into something looking really cool and easy to read for a non-programmer.

## Would I revisit making this mod?

Probably not. I'm not even convinced Minecraft needs Command Blocks, let alone a full programming language in a block. ComputerCraft has that covered.

As for a proper scripting environment, I don't think Ruby is the right choice: Lua would make more sense because of its prevelance, but the game has such a huge educational power that it would be a missed opportunity to not go for a language that provides a transferable skill.
