# Narikiri Dungeon X
An attempt to create a patch for Narikiri Dungeon X (PSP).  
https://docs.google.com/spreadsheets/d/1S1RwcAVeOqBEnIhfU2z7ZVff2C3vqwJ1tvNcy_qHZ6w/edit#gid=0

![logo](https://raw.githubusercontent.com/pnvnd/Narikiri-Dungeon-X/main/assets_archives/images/GITHUB_cover.png)  

![NDR](https://raw.githubusercontent.com/pnvnd/Narikiri-Dungeon-X/main/assets_archives/images/ndr.png)  

## Hacker Note 1
Here is one interesting snip, might be useful for Narikiri Dungeon X...  
If you want to know the file names of the data you extract, check this:

![hash](https://raw.githubusercontent.com/pnvnd/Narikiri-Dungeon-X/main/assets_archives/images/hash.png)  

```
unsigned int hash(const char* string)
{
    unsigned int hash = 0;
    while(*string)
        hash = ((hash << 7) + hash) + (hash << 3) + *string++;
    return hash;
}
```

The files packed inside the `.bin` each have one its filenames...
...Except that they don't appear enywhere in the `.bin`, or any other file in the game.  

Some system files do have their names in the ELF (`all.dat`).
Those numbers are not random, they are the hash of the filename string, as per the above hash function.

For example, it's actually pretty trivial to "detach" this game, from Phantasia X, resulting in 2 independent ISO's.
> It's just another ELF inside the game (`top.prx`?)

Then alpha-out the Phantasia X logo from the menu...and voila!
The game's engine simply reloads the menu if you still try to access it.


## Hacker Note 2
1. Besides the executable, `all.dat` is the only file of the game
2. Indices are in the executable, offset + size + name hash.  So each entry should be 12-bytes, given each member 32-bits
3. Are the "sub-files" padded to start at an LBA? I think so, try searching for both (should take 2 minutes)
4. Just identify the padding in the big file to know where a subfile starts. Then search both offsets and sector in the exe. And you'll find the entry table.
5. Now the game doesn't store names anywhere, but rather generates them at runtime. Then hashes the generated names and looks it up in the index table to find offset and size. If you want to extract the right file names, you need to bruteforce the hashes from the index table with the function above (Hacker Note 1)

## Hacker Note 3
The game engine statically links something like `libc`, so it uses `sprintf` to generate in runtime file names.
For instance `sprintf('levels/%d/enemy/%s/model.dat", 7, "bat");`
Just a made up case. Then it would hash `"levels/5/enemy/bat/model.dat"`
So theoretically if you know all the folders you can brute force faster
Some system files, like the font, have file names directly stored in the execuable
Those are easy to get, just run strings on the exe. Hash all, and take the ones existing in the entry table
There is also the fact that the hash is a partially reversible function
Meaning that a small change in the source string, causes a small change in the resulting hash
With all this... the hashes could be all recovered

## Hacker Note 4
Other aspects of the game are somewhat easy to hack, like the font
It's just a set of `gim` files. You can't find them using the GE debugger though...
Just find the font name in the exe, hash it and get the font from the big file with its offset and size
Decode the gim and that's it

## Hacker Note 5
Files with `MSCF` in the header are Microsoft CAB Files and can be extracted in Windows with `EXPAND file.cab -F` and packed with `MAKECAB`.  For example, Chat files have .bin extension but can be extracted like a CAB file.  Furthermore, after extract chat file, the resulting .dat file is a PAK3 file that can be extracted with pakcomposer.

## Hacker Note 6
Re-pack scenario/script CAB files with the following:  
`makecab /D CompressionType=LZX /D CompressionMemory=15 /D ReservePerCabinetSize=8 ar.dat cab.cab`  

While not required, update CAB identity to `4392`.  Do this by hex editing CAB file at `0x20` and `0x21` from `00 00` to `28 11`.

## Hacker Note 6
looks like tss header starts with TSS (of course)  
there's some kind of start `0x04` and end `0x14`, probably to the bytecode script  
pointer to the text block `0XOC`  
maybe size of the text block `0x18`  
then you just have to parse the bytecode script looking for stuff, there might be differences across games if they changed it between games  
from the notes here, `0x0100A304` is a pointer table (?)  
`0x40002004` is a "name array", whatever that means  
`0x00008202` is a direct string  
`0x10000C04` is some kind of skit thing  
`0x40000C04` is a different type of skit thing  
`0x00002004` is some other kind of pointer table  
you could reverse engineer the entire bytecode to be able to compile your own event scripts  
but imo it's too much work, just look for commands related to text  
and leave the byte script alone, edit only pointers and text  

## Links
- https://talesofnaridanx.weebly.com/downloads.html
- https://www.youtube.com/user/Crevox/videos
- http://www.mediafire.com/file/g83m35bh2ju746s/top_patch_v012.rar/file

## Credits
- Thanks to `Kajitani-Eizan` for some hacker notes
- Thanks to `Sky` for donating resources to get started
- Thanks to `crevox` for permission to use their menu patch
- Thanks to `kevassa` for permission to use their English translation script
