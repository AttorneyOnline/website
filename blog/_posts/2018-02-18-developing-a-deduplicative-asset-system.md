---
layout: post
title: "Developing a deduplicative asset system"
author: "longbyte1"
---

We have probably accomplished more in the past three days than I have been able to accomplish in the past month.

After an extended discussion, I decided to suspend work on Animated Chatroom to make way for an Electron-based Attorney Online 3 client, which would cut multiple layers of abstraction in that were present in Animated Chatroom and make it much easier to add features with less code, including WebM, Opus, and the network protocol.

While the others work on general client/server code, I’ve been tasked with developing a tool to convert AO-family assets to the AC/AO3 format, as well as managing the design of the new asset system.

Converting char.ini files to the info.json format is surprisingly tricky, even while closely following the specification as written by OmniTroid. Sound effects must be extracted from the base installation, and case sensitivity must be watched (some creators took the liberty to use capital letters in directories and shout sounds). What’s more, some obscure INI typos are detected by the converter, even though they may not have produced any error at all by any client.

(However, I have not yet addressed other extensions of the format that were addressed in AO2, especially animation timing. As it turns out, Blink and other web engines “round down” the GIF animation delays in order to maintain sync with the display’s refresh rate – that is, a .02 (1/50) second delay gets rounded down to 1/60 to achieve a smooth 60 fps animation, a “feature” which I do not believe Qt has added support for.)

At this point, it would be favorable to attempt conversion of GIFs to VP8 (WebM), which should greatly reduce the size of sprites while incurring a minor compression loss (chroma subsampling?), as well as converting uncompressed WAV to 48k or 64k Opus (which resamples the audio to 48 kHz, but not as if it wasn’t already mono 22.5 kHz to compensate for the size of WAV files). I will leave this format conversion for another day, though, since it incurs great risk.

After placing all of the asset files into the content.tar (alongside info.json), a checksum must be generated, which serves as the asset’s unique identifier. This unique identifier is then requested by clients during the automatic download process. The benefit to this design is that clashing is essentially impossible – that’s one problem down.

But we haven’t even gotten to the meat of it! To truly deduplicate asset data, a parent hierarchy is in place to allow one asset to be the derivative of another, only needing to package new data instead of both new and old data. In this way, characters can be updated seamlessly, even a trivial fix, without having to download the entire package anew. This is at the core of the system.

Indeed, this also means that assets imported from AO1 should also strive for deduplication. The only way to accomplish this is by establishing the AO Standard Base meta-asset, which would contain the generic sound effects and interjections shared by all AO1 characters. I would then incorporate the manifest of the standard base into the import script, allowing child assets to be generated without necessarily requiring a base folder.

The implications of this system are important to consider: first, it requires a degree of cooperation with asset makers, who need to understand the existence of a hierarchy, and, instead of fighting the system by attempting to create assets with no connection to their parents, simply embrace the decentralized nature of the system (without reinventing peer-to-peer file sharing). Second, the parent hierarchy makes it seem like a graph theory problem. Should child assets be only allowed one parent – or could there be a system where multiple parents are allowed? Third, this imposes monumental requirements on the side of the asset server to verify that two assets have not been duplicated with different hashes, and to perhaps visualize the relationships of assets.

As for decentralization, every asset server is a repository – essentially a glorified web server. Game servers suggest a list of asset servers by order of priority – ideally “official” repositories first (having the best bandwidth) and personal repositories last (having the worst bandwidth).

For those who cannot host full-fledged servers, but can use file hosting services such as Dropbox, I have also designed a “mini-repo” in which server owners need only host two links: an automatically-generated index of assets, and a large zip file containing all of the mini-repo’s assets. New assets (including updates) can be placed in a new repository. This design is not final yet, but I am sure it’ll prove to be convenient.

To be frank, I have never seen such a complicated system for assets in any other game, even after trying to simplify it as much as I could. (And if you got this far, what did you expect, it’s an engineering post.) But I’m willing to try this out – there are some clear advantages:

    Assets are easy to download.
    Assets are easy to modify without needing to tamper them.
    Changes made to an asset do not affect the original asset.
    There are many places where assets can be downloaded.
    Assets cannot conflict with one another.

For now, I will continue to hack away at the conversion script until it becomes reliable enough to make full conversions of around 400 characters I have buried somewhere in my hard drive.
