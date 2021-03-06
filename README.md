# gdcl - GoldenDict command-line interface written in Ruby

gdcl is a command-line interface for searching [GoldenDict](https://github.com/goldendict/goldendict) dictionaries. A request for a command-line version is currently [the third most commented issue](https://github.com/goldendict/goldendict/issues/37) on the GoldenDict issue tracker. This script is a very rudimentary workaround to allow searching through groups of dictionaries until an official command-line interface is available.

As an example of a similar interface, [StarDict](http://code.google.com/p/stardict-3/) has [sdcv](http://sdcv.sourceforge.net/) (StarDict Console Version), but it can only handle dictionaries in the StarDict format. For users of GoldenDict who have large collections of dictionaries in other formats (e.g. DSL or BGL), converting and maintaining two parallel sets of dictionaries is not a practical solution.

This script answers a practical need: namely the ability to search through groups of dictionaries from the command-line over ssh. The script can be used to search dictionaries interactively, but also has a non-interactive mode which allows results from GoldenDict dictionaries to be piped to standard output or used as part of a toolchain.

Currently, gdcl does not require an installation of GoldenDict, as it simply searches through subdirectories of existing dictionaries in the GoldenDict folder (which can be configured) and could conceivably be used to search through any collection of dictionaries in DSL format. However, the eventual goal of the project is to read preferences from GoldenDict's config file, support the full range of formats that GoldenDict can use and, ideally, to use GoldenDict's pre-made index files for faster searching.


* [1.1 Installation from distro packages](#installation-from-distro-packages)
  * [1.1.1 User packaged](#user-packaged)
* [2.1 Usage](#usage)
  * [2.1.1 Summary](#summary)
  * [2.1.2 Setup and configuration](#setup-and-configuration)
    * [2.1.2.1 gdcl.rb](#gdclrb)
  * [2.1.3 Searching](#searching)
  * [2.1.4 Options](#options)
    * [2.1.4.1 Listing groups and dictionaries](#listing-groups-and-dictionaries)
    * [2.1.4.2 Ignoring and restricting dictionaries](#ignoring-and-restricting-dictionaries)
    * [2.1.4.3 Logging search history](#logging-search-history)
    * [2.1.4.4 Forvo audio pronunciations](#forvo-audio-pronunciations)
* [3.1 Supported formats](#supported-formats)
* [4.1 To do](#to-do)
* [5.1 License](#license)


## Installation

To install you can either download the project source and run the script directly, or use a package manager to install the appropriate files for your distro. See below for more details and also the section on [setup and configuration](#setup-and-configuration) for how to customize your installation once you've downloaded the source files.


### Installation from Distro Packages
#### User Packaged
* ![logo](http://www.monitorix.org/imgs/archlinux.png "arch logo")Arch Linux: in the [AUR](https://aur.archlinux.org/packages/gdcl) and [Firef0x's Arch Linux Repository](http://firef0x.github.io/archrepo.html).

## Usage
### Summary

Interactive search:

    ruby gdcl.rb

Non-interactive search:

    ruby gdcl.rb [group] [keyword]

A group name can also be specified without a keyword, i.e.:

    ruby gdcl.rb [group]

This can be useful because it allows you to bind an alias to invoke the program and lookup words in specific groups. For example, you could bind `gf` to look up words in a subfolder of French dictionaryies called `fr`, and `gr` to search in Russian dictionaries (subfolder `ru`):

    alias gf='ruby /path/to/gdcl.rb fr'
    alias gr='ruby /path/to/gdcl.rb ru'

This can be a good way of maintaining a large collection of dictionaries while still being able to search quickly and with minimal interaction.

See below for further configuration and usage details.

### Setup and configuration
#### gdcl.rb
The main script for searching through dictionaries is called **gdcl.rb**.

There are a number of configuration options available in the `config.yml` file. By default, this file should be installed in the standard config folder under the user's home directory (i.e., in the folder `~/.config/gdcl`). If gdcl can't find the file `config.yml` in that folder, it will look for it in $XDG_CONFIG_DIRS (i.e., `/etc/xdg/gdcl`), and failing that, the script folder (i.e., the same directory as the script executable). The `~/.config/gdcl` folder and default `config.yml` file will be created if they do not already exist when you first run gdcl.

The options available in config.yml are commented and should be self-explanatory. They are listed below for reference:

* `dict_dir`: _Dictionary folder_ (The location of your GoldenDict dictionaries folder; can be any folder, but set to `~/.goldendict/dic` by default)
* `group`: _Group name_ (A subfolder of the directory specified in `dict_dir` above, containing a group of dictionaries to be searched together; default blank -- if you specify a value here, gdcl will never ask interactively for a group name, and will use the specified group by default)
* `kword`: _Keyword to search for_ (Use this to specify a keyword in the script; if not specified here, gdcl will search for a term provided either interactively or on the command line)
* `interactive_search`: _Interactive search_ (Set to false for non-interactive search, e.g. to pipe or redirect the search results; defaults to false if a group and keyword are specified as command-line parameters)
* `header_footer`: _Header and footer information_ (Set to false to turn off header and footer information, i.e.: dictionary name and number of hits for search term)
* `pager_off`: _Don't prompt to open results in pager_ (Set to true to turn off the pager prompt for all searches)
* `case_off`: _Case insensitive search_ (Set this to true if you want all searches to ignore character case)
* `history`: _Log search history_ (Uncomment this line if you want to log a record of your searches to a text file)
* `logfile`: _Alternate logfile location_ (A directory where you want to store the history logfile (default is a file called history.txt in the gdcl config folder, i.e.: ~/.config/gdcl))
* `temp_dir`: _Temporary working directory_ (The directory where gdcl will store files)
* `search_term`: _Search pattern_ (Specify a pattern to search for; default is headwords starting with _keyword_, but strict matches or any other regex are also supported)
* `del_dict`: _Excluded dictionaries_ (Optionally exlude the specified dictionaries from search results)
* `markup`: _DSL Markup Options_ (Defaults to removing dsl dictionary markup in results; to display markup, comment out this line and uncomment the line `markup = ""`)
* `markup_replace`: _DSL Markup Replacement String_ (Change this if you want to replace dsl markup with some other string)

See also the Options section for more details on how to specify some of these as command-line options.

Note: You don't need to set up or configure gdcl if you just have one or more dictionaries in a single folder that you want to search. You can specify the folder to look in with the `-d` option, or just navigate to the location of the dictionary folder in your terminal and execute gdcl with `-d` using `.` (a single period) to represent the current directory:

    ruby /location/of/gdcl.rb -d .

When prompted to enter a group name, just use `.` again.

### Searching

By default, invoking gdcl with the command `ruby gdcl.rb` will search interactively. Command prompts will ask you to specify a group of dictionaries to search in out of a list of available groups, and then a keyword to look for. Results will be displayed immediately to standard output.

In interactive mode, after the search results have finished displaying, there is an option to view the results in a paging program (by default `less`). This is helpful if there are many results or if results exceed the terminal buffer size.

Alternatively, you can use non-interactive mode to search and pipe results to a file or other programs. gdcl will default to interactive mode if a group and keyword are specified as command-line parameters:

    ruby gdcl.rb [group] [keyword]

For example, if you want to search for the term _aardvark_ in the _en_ dictionary group, you can use:

    ruby gdcl.rb en aardvark

As always, it is a good practice to quote or escape search strings, and this is mandatory for terms that contain e.g. spaces:

    ruby gdcl.rb en "monkey wrench"

To pipe dictionary search results to a file:

    ruby gdcl.rb en "monkey wrench" > output.txt

Regular expressions are supported in search terms. Let's say you are looking for the word "test" in a collection of dictionaries. By default gdcl searches for headwords in the dictionary that _begin_ with the search string, but this might give too many results ("testament", "testimony", "testing" etc). To find only words that _strictly match_ the word "test", you could use `test$`.

As another example, searching for `arm.....o` or `arm.*o$` will both find the word "armadillo".

### Options

Most default options can be configured in the user's config.yml file (see [here](#gdclrb) for more details on setting up the config.yml file).

There are also a number of settings that can be specified on the fly as command-line options. Use `ruby gdcl.rb -h` to print a list of all available command-line options. Currently, gdcl supports the following options:

* `-c GROUP`, `--names [GROUP]` (_List all dictionaries in specified group by canonical name_)
* `-C`, `--case-off` (_Enable case insensitive search_)
* `-d DIRECTORY`, `--dict-directory DIRECTORY` (_Directory in which to look for dictionaries_)
* `-g`, `--groups` (_Print a list of all available dictionary groups_)
* `-h`, `--help` (_Print help message_)
* `-H`, `--history` (_Record search term history in a log file_)
* `-i FILENAMES`, `--ignore FILENAMES` (_List of dictionaries to ignore while searching_)
* `-l GROUP`, `--list GROUP` (_List all dictionaries in specified group by filename_)
* `-L`, `--logfile DIRECTORY` (_Directory in which to store search log_)
* `-m`, `--markup` (_Don't strip DSL markup from output_)
* `-n`, `--no-headers` (_Remove headers and footers from results output_)
* `-p`, `--pager-off` (_Don't prompt to open results in pager_)
* `-r`, `--restrict FILENAMES` (_Restrict search to FILENAMES_)

Most of these can be combined, e.g.: `ruby gdcl.rb -nm -d /path/to/dictionaries` to search in `/path/to/dictionaries` and print out results with no headers or footers and without stripping DSL markup.

Some options provide information that can be supplied to other options. For example, you can use `-g` to get a list of available groups, and then print out a list of all dictionaries in one of those groups using the `-l` option. The results of `-l` can, in turn, be used to specify a list of dictionaries to ignore with the `-i` option.

An in-depth look at usage of some of these options is below.

#### Listing groups and dictionaries

The `-g` and `-l` options list all the groups and dictionaries within a group, respectively, that gdcl knows about. The group names are essentially the names of subfolders in your GoldenDict dictionaries folder (or whichever folder you have specified in `config.yml`).

The dictionary names provided by `-l GROUP` are in fact the raw filenames (minus `*.dsl.dz` extension) of each of the dictionaries in `GROUP`. This is useful for accessing other command-line options that take dictionary names in this format, such as [restrict and ignore](ignoring-and-restricting-dictionaries).

If you are more interested in the canonical name (i.e., the name specified in the first line of a DSL file) of dictionaries in a given group, you should use the `-c` option instead of `-l`.

For example, let's say you have a Swedish dictionary contained in a file called `myswedishdictfile.dsl.dz`, located in a subfolder `sv`, and the first line of the DSL file looks like this:

    #NAME "Fancy Swedish Dictionary - Min extraordinärt svenska ordbok"

Calling `ruby gdcl.rb -l sv` will give the following output:

    myswedishdictfile

Whereas calling `ruby gdcl.rb -c sv` will output the full name:

    Fancy Swedish Dictionary - Min extraordinärt svenska ordbok

To get a list of dictionary filenames mapped to canonical dictionary names, you can use the `-c` and `-l` options together. Following the example above, you could use the following command to list filenames and canonical names of dictionaries in the `sv` folder:

    ruby gdcl.rb -c -l sv

And the resulting output would be:

    myswedishdictfile	Fancy Swedish Dictionary - Min extraordinärt svenska ordbok

(Note, if you are piping this output to another file, the two fields are separated by a tab space)

#### Ignoring and restricting dictionaries

You can ignore certain dictionaries in a group with `-i` or, conversely, restrict your search to a subset of dictionaries in a certain group with `-r`. For example, to ignore a dictionary called `jedict.dsl.dz` in the `jp` group, use:

    ruby gdcl.rb -i jedict jp

The dictionary you specified will be excluded from your search, and results from all other dictionaries in the group will be shown instead.

If you want to _only_ show results from `jedict.dsl.dz`, you can use the following command:

    ruby gdcl.rb -r jedict jp

This can be really useful if you have groups with a large number of dictionaries (particularly collections of example sentences or encyclopedias), and you only want to do a quick lookup of a single term.

Both of these options can take comma separated lists of dictionaries you want to ignore or restrict your search to, for example:

    ruby gdcl.rb -r oed,longman,webster en
    ruby gdcl.rb -i wikipedia,encyclopaedia_britannica,good_writing_guide en

All dictionaries in the list will be included in the `--ignore` or `--restrict` parameters if they exist.

**Tip:** When using `-r`, you can supply a partial filename to search only in the dictionary (or dictionaries) that match the given string. For example, if you have a dictionary called `supercooldict.dsl.dz`, you could search only in that dictionary by entering:

    ruby gdcl.rb -r super

On the other hand, if you have a _collection_ of dictionaries in group `es` called e.g., `collins_spanish-english.dsl.dz`, `collins_spanish-french.dsl.dz`, `collins_spanish-verbs.dsl.dz`, `collins_french-spanish.dsl.dz`, etc., you could restrict your search to only those dictionaries containing `collins` in the title by using the command:

    ruby gdcl.rb -r collins es

#### Logging search history

If logging is enabled (it's off by default), gdcl will save a record of all search terms to a history file, located by default in the gdcl configuration directory (`~/.config/gdcl/history.txt`). This can be useful for, e.g., studying or learning new vocabulary.

There a couple of ways to enable logging of search terms, and you can also specify an alternate directory to save the history file.

If you want gdcl to always record your search terms, you should enable logging in the gdcl config file. Look in `config.yml` for a line containing `# :history: true` and uncomment it to turn logging on for all searches. If you only want to gdcl to record search terms selectively, you can use the `-H` option on the command-line turn logging on for a specific search or set of searches (if you use `-H`, logging will remain in effect until you exit the program).

To specify an alternate directory, uncomment the `# :logfile:` line in `config.yml` and specify a directory of your choice. You can also specify a different directory using `-L` and the directory name when running gdcl. This could be useful for, e.g., recording new vocabulary from different sources in different history files.

For example, let's say you were reading a [Swedish crime novel](https://en.wikipedia.org/wiki/Scandinavian_noir) and you wanted to lookup vocabulary as you read. You could open up an instance of gdcl and send a record of your search terms to a separate file with the following command:

    ruby gdcl.rb -H -L novel_vocab.txt

Later you are watching an interesting [Swedish documentary series](http://www.tv4play.se/tv/tags/Dokument%C3%A4rer) so you open up gdcl to record vocabulary in a new file:

    ruby gdcl.rb -H -L documentary_vocab.txt

Finally, you have to finish your Chinese homework, and you want to look up new words as you go along:

    ruby gdcl.rb -H -L homework_vocab.txt

Note that the search history file only contains a record of your _search terms_, not the actual search results themselves. If you want to save the full text of search results you should use [non-interactive mode](#summary) and pipe the results to a file, e.g.:

    ruby gdcl.rb mygroup "my search term" > search_results.txt

#### Forvo audio pronunciations
Looking up and playing back audio pronunciations from [Forvo](http://forvo.com/) is supported by gdcl using the `-f` option and mplayer. This requires registering for a [Forvo API key](http://api.forvo.com/), which is free for non-commercial educational use.

Once you have a key, you just need to copy it into your gdcl config file in your user home directory (i.e. `~/.config/gdcl/config.yml`) under the section "forvo key". Uncomment the line `# :forvo_key: ""` and add your key between the quotation marks `""`. Now you can look up pronunciations by running `gdcl.rb` with the `-f` option.

The basic format for a Forvo audio lookup is:

    ruby gdcl.rb -f [lang_code] [word_to_be_pronounced]

You'll need to supply the 2-letter [ISO 639 language code](http://www.forvo.com/languages-codes/) and a word or phrase to pronounce. You can find a full list of the supported codes [here](http://www.forvo.com/languages-codes/).

For example, if you wanted to find the pronounciation of the word "сегодня" in Russian, you would enter:

    ruby gdcl.rb -f ru сегодня

The last argument should probably be in quotes to avoid problems -- this also allows for pronunciation of phrases and other terms with spaces:

    ruby gdcl.rb -f sv "Johannes Robert Rydberg"

The script will immediately begin playback of all the available pronunciations it found.

Gdcl only supports playback of pronunciation audio. For more full-featured access to the Forvo API, including listing track info and saving pronunciation audio, check out [forvo-cl](https://github.com/dohliam/forvo-cl).

## Supported formats

gdcl currently supports compressed dictionary files in [ABBYY Lingvo .dsl dictionary format](http://lingvo.helpmax.net/en/troubleshooting/dsl-compiler/your-first-dsl-dictionary/) (i.e., files ending in the extension `.dsl.dz`) as well as online pronunciation audio files from Forvo.com (see [the section above](#forvo-audio-pronunciation) on using gdcl to look up pronunciations). Support for other formats and online dictionaries is planned for future releases.

## To do

Features that need to be implemented:
* Read search and dictionary preferences from GoldenDict config file
* Search using GoldenDict's existing index files
* ~~Dictzip support (i.e. search dictionaries in place rather than needing to unzip them to tmp folder)~~
* bgl, dict and other formats support
* Online dictionaries support (Wikipedia, Wiktionary etc)


## License

MIT -- see LICENSE file for details.
