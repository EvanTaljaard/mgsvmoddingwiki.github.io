---
title: Hash Wrangling
permalink: /Hash_Wrangling/
---

Wrangling Hashes, notes and workflow

## Hashing?

Hashing is used a lot to generate smaller numeric representations of
strings to allow more efficient comparison/lookup.

The fox engine uses different hashing functions for different purposes.

It mostly uses CityHash with seeds.

The main ones so far are StrCode32 - used in lua, langIds, and in GZ as
part of the file name hashing?? PathFileNameCode64 - Used in file name
hashing, most commonly seen in output from GzsTool/QarTool. (The name is
my(tinmantex) choosing, since I don't know actual the function name)
PathFileNameCode32 - some variant of the above, with a uint32 return to
fit in luas number representation, used in a limited fashion in lua.

In most cases we (modders) are only presented with the hashes
themselves, and working with just hashes is not ideal in terms of
identifying something in a human way so we want to recover the original
strings.

Assuming we know/have a running version of the hash function used we do
this by gathering the hashes we want to recover, a list of strings we
think might be the original values and comparing by converting the
string using the same hash function and then comparing to the hash.

## Main goals

Creating a definitive set of hashes for a given set of data. Creating a
dictionary of original that matches just those hashes.

While there are several dictionary projects for different tools, the
processes and workflows haven't been documented so have presented
difficulty as different members have contributed over time, some have
stopped contributing, and the data sets have increased from patches.

So another preferred goal would be to provide enough information of
workflows, tools and input data be repeatable, and hopefully achieve the
goals quicker if later research or patches increase the set of data.

### Up in the air

Hash lists and dictionaries should probably be limited to the specific
game (Tpp/GZ/MGO/Whatever) for performance and to mitigate hash
collision. This is more an issue for StrCode32 (with it's uint32 hash it
will cause more collisions that a uint64 output hash), and also where
strings aren't likely to overlap with TPP. In the case of
archives/filenames it's possible many assets will be brought over from
GZ and MGO so it might be better to have combined dictionary? The
current dictionary also has PT strings in it which would probably be
better to shift to it's own dictionary.

Tools standardization, ideally tools that use hashes should document the
hash function used, have an option to output hashes from processed files
and allow to specify which string dictionary to use.

## Gathering hashes

This will be different for each data type and whatever tools may be
being used to view the data. In some cases we can (barring any additions
from patches) get a definitive list of all hashes for the set of data.
If an up to date repository of the hashes can be found this step can be
skipped and you can go on to Creating a test dictionary.

###
langId hashes:

Repository: <https://github.com/TinManTex/mgsv-lookup-strings>

Gathering: Using tinmantex fork of LangTool
<https://github.com/TinManTex/FoxEngine.TranslationTool/> with the -d
switch will append all hashes of an lng/2 file to langIdHashes.txt in
the working folder. Workflow: Make sure all lng files are accessible:
Extract all qar .dats Extract all fpk/ds.

Delete any existing langIdHashes.txt

Run LangTool on each lng/2 with the -d switch

Use some process to cull the list to unique hashes and sort. TODO: post
uniquify

FileName hashes: Repository: The
<https://github.com/emoose/MGSV-QAR-Dictionary-Project> has reference
hashes, however they are currently not updated to 1.0.11.0, and there's
no information of emooses workflow to create them.

Gathering: Ideally GzsTool should have an option to output hashes as it
processes archives. As that is not the case one method is to recover
from the archive metadata created by GzsTool or QAR Tool/FPK Tool. If
it's easier than adding hash output to GzsTool a tool should be created
to automate the following workflow step.

As GzsTool does not output the file name hashes on it's entries if it
finds a string match from it's dictionary you will have to run GzsTool
on all files with an empty qar_dictionary.

QAR Tool will output hashes to it's archive.inf files regardless.

All target archive metadata files should be processed to gather the
hashes. GzsTool uses xml so could be recovered by reading that, but it
would probably be faster to just targeted string scrape 'Key=' QAR tool
inf files, for each line, if find '|', hash is string before '|'

## Creating a test dictionary

Can either be a kitchen sink approach of whatever strings you can find
or generate, or something more targeted from any currently known strings
for that set of data. This falls into string scraping, and brute-force
or dictionary-attack methods.

langIds: There are many original/unhashed langIds referenced in the lua
scripts.

fileNames: There are file name references in the lua scripts and fox
files.

<https://github.com/TinManTex/mgsv-lookup-strings> project has a
collection of strings from scraping various MGSV data and manual
additions.

<https://github.com/cstBipBop/MGSV-dictionary-generator> has some
dictionary generation approaches.

## Testing hashes vs dictionaries

<https://github.com/TinManTex/HashWrangler> has been created for this
purpose.

Input \<hashes.txt\> - hashes to test \<strings.txt\> -
dictionary/strings to test

Output <hashes>_matchedHashes.txt - hashes that matched a string
<hashes>_unmatchedHashes.txt - hashes that didn't match any strings

<strings>_matchedStrings.txt - strings that matched a hash, can be
considered the validated dictionary for the given input hashes.
<strings>_matchedStrings.txt - strings that didn't match any hash

_matchedHashes and _matchedStrings are output in paired order, so for
a given line in one file it will match in the other.

<strings>_HashStringMatches.txt - hash string pairs, useful as a manual
human lookup or identifying the file names for files that were extracted
prior to the name being found.

<strings>_HashStringsCollisions.txt - hash strings pairs, input strings
that resolved to the same input hash. If you can clearly determine which
string is correct then add it to the validated dictionary and remove it
from input strings for this set of hashes. If you can't decide which
string is correct it's probably better to just remove them all from the
input strings for the hash set.

Ok lets run some data through this.

Setup: Grab <https://github.com/TinManTex/mgsv-lookup-strings> Beyond a
collection of scraped strings useful for creating test dictionaries it
has hashes and existing dictionaries for tools.

I'll be focusing on langIds for LangTool

LangTool\\Hashes\\ - has hashes output from tinmantex LangTool fork
using the -ExportHashes option. LangTool\\Dictionaries\\ - has the
current lang dictionary project lang_dictionary

Other scattered output from HashWrangler which will get replaced if you
follow the process.

For example I will be testing
LangTool\\Hashes\\langIdHashesTpp-1.0.11.0.txt - As of writing the
current set of Tpp langId hashes.
LangTool\\Dictionaries\\MGSV-Lang-Dictionary-Project-2017-08-10 - Which
had some strings added that don't apply to langIds as well as some
string collisions.

Command line for HashWrangler is HashWrangler <hashes file path>
<strings file path> -HashFunction <hash function type>

LangId uses StrCode32 which is HashWranglers default so we don't really
need to add the option -HashFunction StrCode32

Console output:

`Using HashFunction StrCode32`
` ReadDictionary D:\Projects\MGS\HashWrangling\LangTool\Dictionaries\MGSV-Lang-Dictionary-Project-2017-08-10\lang_dictionary.txt`
` hash collision for 532403241: announce_find_plant | flowstation_sp_seperator_tank`
` hash collision for 1424049156: cassette_001 | option_h_vr`
` hash collision for 1702400502: mb_sortie_mode_fob | pause_info_point_none`
` ReadInputHashes D:\Projects\MGS\HashWrangling\LangTool\Hashes\langIdHashesTpp-1.0.11.0.txt`
` Finding strings for hashes`
` Stats:`
` unmatchedHashes    [ 5413/15950]`
` matchedHashes      [10534/15950]`
` collsionHashes     [    3/15950]`
` unmatchedStrings   [ 1119/11659]`
` matchedStrings     [10534/11659]`
` Writing out files`
` done`

As you can see the dictionary did pretty well, matching a lot of
strings, but we have a few collisions. Opening
\\HashWrangling\\LangTool\\Dictionaries\\MGSV-Lang-Dictionary-Project-2017-08-10\\lang_dictionary_HashStringsCollisions.txt
shows:

`1424049156 cassette_001||option_h_vr`
` 1702400502 mb_sortie_mode_fob||pause_info_point_none`
` 532403241 announce_find_plant||flowstation_sp_seperator_tank`

The langId hash, then strings that matched to the hash separated by ||
One string for each hash needs to be chosen for the dictionary, but
which is correct? First we'll run LangTool with our new
langIdHashesTpp-1.0.11.0_matchedStrings.txt as this will have removed
the colliding strings. With the tinmantex fork we can specify the
dictionary with the -Dictionary <strings path> option.

After that’s completed a find-in-files using a text editor for the
hashes with collisions. The first langId hash 1424049156 fortunately
finds itself sandwiched between other found langIds

<Entry LangId="cassette_002" Color="1" Value="TRACK" />
` `<Entry Key="1424049156" Color="1" Value="CASSETTE" />
` `<Entry LangId="gameover_select_return_title" Color="1" Value="RETURN TO TITLE MENU" />

Which suggests cassette_001 is the correct langId and option_h_vr is
incorrect The following hashes are also pretty clear as to the correct
langId

<Entry LangId="mb_sortie_mode_sideops" Color="1" Value="SIDE OPS" />
` `<Entry Key="1702400502" Color="1" Value="FOB MISSIONS" />
` `<Entry LangId="mb_fob_staff_res" Color="1" Value="Assigned Staff/%sPlaced Containers" />
` `
` `<Entry LangId="announce_find_processed_res" Color="5" Value="Processed Material Spotted [%s] x %d" />
` `<Entry Key="532403241" Color="5" Value="Medicinal Plant Spotted [%s] x %d" />
` `<Entry LangId="announce_extract_keyitem" Color="5" Value="Key Item [%s]" />

For extra confirmation a search of the collision strings in the lua
files could be done: announce_find_plant appears, and in relation to
langId usage, flowstation_sp_seperator_tank does not appear in the
lua files. The other colliding strings do not appear, so no confirmation
either way for those.

Now to finalize a validated dictionary for LangTool I copy off
lang_dictionary_matchedStrings.txt to
\\HashWrangling\\LangTool\\Dictionaries\\lang_dictionary.txt, since I
aim for langtools default dictionary to be targeted to tpp. And manually
copy the correct strings - cassette_001, mb_sortie_mode_fob,
announce_find_plant to it.

Another run of HashWrangler, this time with lang_dictionary_validate
gives:

`Using HashFunction StrCode32`
` ReadDictionary D:\Projects\MGS\HashWrangling\LangTool\Dictionaries\lang_dictionary.txt`
` ReadInputHashes D:\Projects\MGS\HashWrangling\LangTool\Hashes\langIdHashesTpp-1.0.11.0.txt`
` Finding strings for hashes`
` Stats:`
` unmatchedHashes    [ 5413/15950]`
` matchedHashes      [10537/15950]`
` collsionHashes     [    0/15950]`
` unmatchedStrings   [    0/10537]`
` matchedStrings     [10537/10537]`
` Writing out files`
` All done`

Great, 0 collisions, 0 unmatched strings, we still have 5413 unmatched
hashes to find, but that's for future attempts.

For now I copy off lang_dictionary_validate_matchedStrings.txt (as
this will have sorted the three resolved collision strings I added) to
lang_dictionary.txt to act as the default dictionary for LangTool.

Future runs can be done by pointing the HashWranglers dictionary arg to
a folder instead of a specific file, and copying the current validated
dictionary there along with dictionaries of new strings to test.
[Category:Guides](/Category:Guides "wikilink")