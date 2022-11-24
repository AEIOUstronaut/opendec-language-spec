# OpenDec Language Specification
**Version v1.0.0**

The OpenDec language definition provides a language to ease the development of
text-to-speech music using text-to-speech engines based on DecTalk. The language
specification recognizes all basic DecTalk commands and features and provides
extended functionality for ease of use.

The original DecTalk progrem is a text-to-speech engine the reads user-provided
text. DecTalk supports features such as alterante pronounciation rules
(e.g. read, tear, lead), directly pronouncing phonemes, and converting phone
numbers to their equivalent dial tone sequences.

The OpenDec compiler, which consumes source code that follows this OpenDec
language specification, is a wrapper for the text-to-speech engine. The compiler
is designed to pass only base commands and phonemes to the text-to-speech
engine. This means that regular phrases (e.g. "hello") will not be processed,
but the equivalent phonemes ("hx eh l ow") will be processed. For this reason,
any base DecTalk commands that do not impact phoneme processing or
pronounciation will not be passed to the text-to-speech engine.

## Language Specification Summary
This language specification covers the following topics:

* Base Types
    - Derivative Types
* Base Components
* Base Commands
* Extended Commands
    - `[:bpm]` Command
    - `[:import]` Command
* Extended Functionality
    - Comments
    - Looping
    - `sound` Definitions
    - `phrase` Definitions
    - `voice` Definitions
* Order of Operations
* Appendices
    - Appendix A: English Phonemes
    - Appendix B: Phoneme Pitches
    - Appendix C: Voice Characteristics


# Base Types
Base types are a helpful abstraction used to help outline valid inputs and
syntaxes. Note that the `whitespace` type does not have any functionality other
than to spearate the other four base types. There are five base types:

| Type          | Definition                                                |
|---------------|-----------------------------------------------------------|
| `int`         | Unsigned integer                                          |
| `float`       | Unsigned floating point number                            |
| `string`      | An alphanumeric string with no special characters         |
| `special`     | One of the following characters: `:,{}[]<>`               |
| `whitespace`  | A whitespace character used to separate other base types  |

## Derivative Types
Some other types can be derived from the existing base types. Although they
cannot be differentiated from their base types in the compiler, they are useful
in the language specification to narrow down use-cases. There are two derivative
types:

| Type      | Base Type | Definition                                                        |
|-----------|-----------|-------------------------------------------------------------------|
| `keyword` | `string`  | A restricted set of strings, used for command options             |
| `var`     | `string`  | A limited string definition, used for user-defined object names   |

A `keyword` is specific to the command in which it is used. A valid `keyword` in
one command may be invalid for a different command.

A `var` is used to name user-defined objects such as a `sound`, `phrase`, or
`voice`. A `var` is a string with a limited definition that follows the regular
expression below:
```
([a-zA-Z])|([_a-zA-Z][_a-zA-Z0-9]+)
```


# Base Components
There are two base components that create an OpenDec script: `commands` and
`phonemes`

A `command` is a control sequence that modifies the text-to-speech engine state.
A command can modify how the engine will process phonemes, define reusable
sequences of components, and more. The syntax for a `command` is:
```
[:COMMAND ARGUMENT_1 ... ARGUMENT_N] { (PHONEME|COMMAND)* }
```

A `phoneme` is a component that represents the sound output of the
text-to-speech engine. This includes silence, symbols that modify stress to a
sound, and user-defined objects. There are four ways to represent a phoneme,
each one giving different options to define the sound's pitch and length. The
possible syntaxes are:
```
PHONEME
PHONEME<LENGTH>
PHONEME<,PITCH>
PHONEME<LENGTH,PITCH>
```


# Base Commands
`command`s are one of the base components understood by OpenDec. Base commands
can be directly passed to the text-to-speech engine (with some exceptions) to
modify its behaviour. This includes things such as modifying the current
speaking voice and volume. Note that some commands will have limited or no
effect on the compiled output (see introduction for more details). A summary of
the supported base commands can be found below.

| Command   | Alt   | Description                                           |
|-----------|-------|-------------------------------------------------------|
| comma     | cp    | Comma pause length                                    |
| dv        |       | Change voice characteristics of current voice         |
| error     |       | Adjust text-to-speech engine error handling           |
| mode      |       | *No effect*                                           |
| name      | n     | Change speaking voice                                 |
| period    | pp    | Period pause length                                   |
| phoneme   |       | *No effect*                                           |
| pitch     |       | *No effect*                                           |
| play      |       | Play a .WAV file                                      |
| pronounce |       | *No effect*                                           |
| punct     |       | Change text-to-speech engine punctuation processing   |
| rate      |       | *No effect*                                           |
| say       |       | Change text-to-speech engine phoneme processing       |
| skip      |       | Tell text-to-speech engine to ignore some rules       |
| tone      |       | Play a sine wave of given frequency and length        |
| volume    |       | Set output volume                                     |

## Comma Pause - `[:comma] | [:cp]`
Change the pause length of a comma (`,`) when processed by the text-to-speech
engine. The pause length will be in beats if the BPM has been modified,
otherwise it will be in milliseconds. The default pause length is 160ms. To
reset the comma pause to the default value, `LENGTH` must be `0`.
```
[:comma LENGTH]
[:cp LENGTH]

Parameters:
    LENGTH (int|float)  Pause length of a comma in milliseconds or beats
```

## Design Voice - `[:dv]`
Adjust the characteristics of the current voice used by the text-to-speech
engine. More details about each voice characteristic can be found in Appendix C.
```
[:dv OPTION VALUE]

Options:
    ap|as|b4|b5|bf|br|f4|f5|hr|hs|la|lx|nf|pr|qu|ri|sm|sr|sx

Parameters:
    VALUE (int)     The value for the given voice characteristic
```

## Error Handling - `[:error]`
Specify how errors should be handled by the text-to-speech engine. The error
handling mode remains in effect until another error handling mode is specified.
The options are:
* `ignore` - ignore all errors
* `speak` - have the text-to-speech engine speak what the error is (default)
* `tone` - play a tone on error
```
[:error OPTION]

Options:
    ignore|speak|tone
```

## Text Processing Mode - `[:mode]`
**NOTE:** This command will not affect the compiled results as this command does
not impact phoneme processing.

Specify how text is processed following this command. This text processing mode
remains in effect until this command is called again. Multiple modes can be
enabled at once. The text processing modes are:
* `math` - change interpretation of symbols (e.g. "-" -> "minus")
* `europe` - use European cardinal pronunciation (e.g. 1.000,00)
* `spell` - spell all words
* `name` - pronounce all upper case words as proper nouns
* `citation` - pronounce short sentences, single words without vowel reduction
* `latin` - valid but not supported
* `table` - table speaking mode

There are three keywords that change the text processing mode's state:
* `on` - enable the specified text processing mode
* `off` - disable the specified text processing mode
* `set` - enable the specified text processing mode and disable all others
```
[:mode OPTION MODE]

Options:
    math|europe|spell|name|citation|latin|table

Parameters:
    MODE (keyword)     on|off|set
```

## Name - `[:name] | [:n]`
Change the current speaking voice to one of the default voices or one of the
user-defined voices. The default voice is `Paul`.
```
[:name NAME]

Parameters:
    NAME (string)   Name of the voice settings to use (e.g. Paul)
```

Alternatively, there is a short-hand command that can be used to set the current
speaking voice, but this is only applicable to the default voices. The `OPTION`
corresponds to the first letter of the voice name to use. For example, call
`[:nb]` for `Betty`, `[:nd]` for `Dennis`, `[:nf]` for `Frank`, etc...
```
[:nOPTION]

Options:
    OPTION  b|d|f|h|k|p|r|u|w
```

## Period Pause - `[:period] | [:pp]`
Change the pause length of a period (`.`) when processed by the text-to-speech
engine. The pause length will be in beats if the BPM has been modified,
otherwise it will be in milliseconds. The default pause length is 640ms. To
reset the comma pause to the default value, `LENGTH` must be `0`.
```
[:period LENGTH]
[:pp LENGTH]

Parameters:
    LENGTH (int|float)  Pause length of a period in milliseconds or beats
```

## Phoneme - `[:phoneme]`
**NOTE:** This command will not affect the compiled results as the compiled text
always has `[:phoneme arpabet on]` set, and to disable it would cause undefined
behaviour in the output sound file.

Modify phoneme interpretation settings for the text-to-speech engine.
```
[:phoneme OPTION on|off]

Options:
    arpabet
```

## Pitch - `[:pitch]`
**NOTE:** This command will not affect the compiled results as this command does
not impact phoneme processing.

Modified the pitch (frequency) different between upper case and lower case
letters. The default pitch difference is 35Hz.
```
[:pitch FREQUENCY]

Parameters:
    FREQUENCY (int)     Frequency difference in hertz
```

## Play WAV File - `[:play]`
Play a wave (.WAV) file.
```
[:play FILE]

Parameters:
    FILE (string)   Path to WAV file to open and play
```

## Pronounce - `[:pronounce]`
**NOTE:** This command will not affect the compiled results as this command does
not impact phoneme processing.

Determines how to pronounce the word immediately following this command. For
example, `[:pronounce OPTION] <word1> <word2>` will use the alternate
pronunciation for `<word1>` but not `<word2>`. The options are:
* `alternate` - pronounce the less common homograph (e.g. bass - the fish)
* `primary` - pronounce the more common homograph (e.g. bass - the instrument)
* `name` - pronounce as a name (e.g. John)
* `noun` - pronounce the noun version of the homograph (e.g. refuse - trash)
* `adjective` - pronounce the adjective version of the homograph (e.g. content - satisfied)
* `verb` - pronounce the verb version of the homograph (e.g. refuse - to decline)
```
[:pronounce OPTION]

Options:
    alternate|primary|name|noun|adjective|verb
```

## Punctuation Rules - `[:punct]`
Determine how the text-to-speech engine will process punctuation. The options
are:
* `none` - punctuation and other symbols are not processed
* `some` - clause boundary punctuation is not processed
* `all` - all punctuation and symbols are processed
* `pass` - all special punctuation processing is turned off
```
[:punct OPTION]

Options:
    none|some|all|pass
```

## Rate of Speech - `[:rate]`
**NOTE:** This command will not affect phonemes that have their length
specified.

Set the rate of speech for the text-to-speech engine. The default rate is 200
words per minute (WPM). The minimum rate is 75 WPM, the maximum rate is 600 WPM.
```
[:rate WPM]

Parameters:
    WPM (int)   Rate of speech in words per minute
```

## Name - `[:say]`
Specify how much data should be processed before the text-to-speech engine
begins speaking. The default option is `clause`. The options are:
* `clause` - speak on the end of a clause
* `word` - speak on the end of a word
* `letter` - speak on the end of a letter
* `filtered` - speak on the end of a letter, ignoring control characters
* `line` - speak on the end of a line
```
[:say OPTION]

Options:
    clause|word|letter|filtered|line
```

## Skip - `[:skip]`
Skip a selected part of preprocessing for the text-to-speech engine until the
next skip command is called. The options are:
* `punct` - turn off all punctuation rules
* `rule` - turn off all rules (e.g. useful when processing phone numbers)
* `all` - skip all preprocessing
* `off` - returns processing to default state
* `cpg` - skip code page translation
* `none` - do not skip anything
```
[:skip OPTION]

Options:
    punct|rule|all|off|cpg|none

Parameters:
    None.
```

## Tone - `[:tone]`
Play a simple sine-wave tone of a given frequency and length. The length will
be in beats if the BPM has been modified, otherwise it will be in milliseconds.
```
[:tone FREQUENCY LENGTH]

Parameters:
    FREQUENCY (int)     Frequency of the tone to play in hertz
    LENGTH (int|float)  Length of the tone in beats or milliseconds
```

## Volume - `[:volume]`
Modify the voice volume of the text-to-speech engine from 0 to 100. Stereo audio
is supported, meaning prepending `l` to the option will modify the left channel,
and prepending `r` to the option will modify the right channel. The options
(without left/right channel specification) are:
* `up` - increase volume by the given amount
* `down` - decrease volume by the given amount
* `set` - set volume to the given value
```
[:volume OPTION VALUE]

Options:
    up|lup|rup|down|ldown|rdown|set|lset|rset

Parameters:
    VALUE (int)     Volume adjustment
```

Alternatively, to set left and right audio channels to different volumes at
once, use the following command:
```
[:volume sset LCHANNEL RCHANNEL]

Parameters:
    LCHANNEL (int)  Volume for the left audio channel
    RCHANNEL (int)  Volume for the right audio channel
```


# Extended Commands
Extended commands provide functionality that might not be supported by the
text-to-speech engine. It is up to the OpenDec compiler to process these
commands in such a way to to enable the feature through commands recognized by
the text-to-speech engine. A summary of the supported extended commands can be
found below.

`command`s are one of the base components understood by OpenDec. Base commands
can be directly passed to the text-to-speech engine (with some exceptions) to
modify its behaviour. This includes things such as modifying the current
speaking voice and volume. Note that some commands will have limited or no
effect on the compiled output (see introduction for more details). A summary of
the supported base commands can be found below.

| Command   | Description                               |
|-----------|-------------------------------------------|
| bpm       | Use beats-per-minute for phoneme length   |
| import    | Import another OpenDec script             |

## Beats Per Minute - `[:bpm]`
Configure the OpenDec compiler to interpret lengths and timings in quarter notes
at the specified tempo (in beats per minute) instead of the default
milliseconds. For example, a quarter not will have a length of `1`, a half note
`2`, and a whole note `4`. Floating point numbers can be used for the length, so
an eighth note will have a length of `0.5` and a sixteenth note `0.25`.
```
[:bpm TEMPO]

Parameters:
    TEMPO (int)     Tempo at which to convert timings
```

## Import Source Code - `[:import]`
Import another script so that users can re-use user-define object or a sequence
of commands and phonemes. Import files can be absolute or relative paths.
The compiler will search for relative paths in the current working directory as
well as any specified include directories.
```
[:import PATH]

Parameters:
    PATH (str)  Path to OpenDec script to import
```


# Extended Functionality
There is additional functionality build into the OpenDec language specification
that has no equivalent in some text-to-speech engines. These are different to
the extended commands above as either the syntax does not match that of a basic
command or does not directly modify the text-to-speech engine's behaviour.

One handy abstraction is `command` context. This adds an extension to `command`
syntax by providing the command with context. This context is used for
functionality such as looping and user-defined objects. Context is any component
or extended functionality between a pair of braces (i.e. `{}`).

The following additional features are supported in this version of the OpenDec
language specification:
* Comments
* Repeat a sequence of components using `loop`
* Define small named, reusable phoneme sequences with a `sound` definition
* Define named, reusable component sequences with a `phrase` definition
* Create new named voices with a `voice` definition

## Comments
C-style inline and block comments can be used to prevent the OpenDec compiler
from processing anything within the command. This is helpful for debugging and
documenting OpenDec scripts. Here is an example for OpenDec comments:
```
// This is an inline comment.
// Anything after the "//" characters on a lien is ignored.

/* This is a block comment */
/* Anything between the "slash-stars" is ignored */

/*
Unlike inline comments, block comments
can work over multiple lines
*/
```

## Loop Component Sequences with `loop` Sections
Any sequence of components or user-define object, including other loops, can be
repeated several times. The syntax for looping is:
```
[:loop COUNT] { CONTEXT }

Parameters:
    COUNT (int)     Number of times to repeat the command context
```

Here is an example of how the loop command works:
```
// The following two lines are equivalent.
[:loop 5] { aa }    // Using the looping feature
aa aa aa aa aa      // Not using the looping feature
```

## Small Phoneme Group Reuse with `sound` Definitions
A `sound` is a named sequence of phonemes that can be reused throughout an
OpenDec script. A `sound` is defined by its name and context. The name is a
`var` and cannot be one of the defined phonemes or share the name of another
user-define object. The context must have at least one vowel phoneme and can
optionally have any number of leading and trailing consonant phonemes. See
Appendix A for a list of consonant and vowel phonemes. The syntax for a `sound`
is:
```
[:sound NAME] { [consonant]* vowel [consonant]* }

Parameters:
    NAME (var)  Name of the user-defined sound
```

Once defined, a `sound` can be referenced throughout the script by calling its
name, and can be invoked like any other phoneme. The length of the `sound` is
evenly split between the vowel phonemes. The pitch of the `sound` is applied
to the vowel phonemes.
```
[:sound blah] { b l ax }

// Once defined, the sound can be called like any other phoneme.
blah                // No parameters
blah<LENGTH>        // Set sound length only
blah<,PITCH>        // Set sound pitch only
blah<LENGTH,PITCH>  // Set sound length and pitch

// The following lines are equivalent.
[:sound blah] { b l ax }
blah<500,10>            // Invoking the sound with length and pitch
b<15> l<15> ax<470,10>  // Equivalent sequence of phonemes

// The following lines are equivalent.
[:sound ooee] { uw iy }
ooee<500,10>            // Invoking the sound with length and pitch
uw<250,10> iy<250,10>   // Equivalent sequence of phonemes
```

## Component Group Reuse with `phrase` Definitions
A `phrase` is a named sequence of components that can be reused throughout an
OpenDec script. A `phrase` is defined by its name and context. The name is a
`var` and cannot be one of the defined phonemes or share the name of another
user-define object. The context can contain any number of components - that is
any base component (i.e. base command or phoneme) and any extended functionality
(e.g. a `sound`, another `phrase`, a `loop`). The syntax for a `phrase` is:
```
[:phrase NAME] { component* }

Parameters:
    NAME (var)  Name of the user-defined phrase
```

Once defined, a `phrase` can be referenced throughout the script by calling its
name, and can be invoked like any other phoneme. However, when invoked, a phrase
will ignore the length and pitch values.
```
[:phrase example] { [:bpm 60] aa<,13> [:bpm 0] }

// The following lines are all equivalent.
example                     // Invoking the phrase
example<100>                // Invoking the phrase - length ignored
example<,10>                // Invoking the phrase - pitch ignored
example<100,10>             // Invoking the phrase - length and pitch ignored
[:bpm 60] aa<,13> [:bpm 0]  // Manually invoking the components
```

## Define New Voices with `voice` Definitions
The text-to-speech engine comes with several default voices. They provide
functionality to define a custom voice, but those settings will overwrite the
default voice `Wendy`. This is limited as we lose the settings for `Wendy` and
we can only save one custom voice at a time.

The OpenDec `voice` definition allows users to create a named custom voice
without overwriting the settings for `Wendy`. Each new `voice` definition is
based on default voice `Paul` - that is, if a voice characteristic is not
modified, then it will default to the voice characteristic value for `Paul`.
For example, if a new voice definition does not explicitly set `sx`, then the
new definition will default to `sx = 1`.

The syntax for a `voice` definition is:
```
[:voice NAME] { option1 value1, ..., optionN valueN }

Parameters:
    NAME (var)  Name of the user-define voice
```

Once a voice is defined, it can be invoked with the `[:name]` command.
```
[:voice custom] { sx 0, hs 110 }

aa<1000,10>     // Default voice is Paul
[:name custom]  // Change the voice
aa<1000,10>     // Using custom voice
```


# Order of Operations.
The following order of operations must be respected by the OpenDec compiler.
Understanding the order of operations may be helpful when debugging your OpenDec
script. The order of operations is:

1) Comments
2) Whitespace
3) Commands
4) Phonemes, Phrases, and Sounds


# Appendix A: Phoneme Definitions
Phonemes are the base sound that is generated and processed by the underlying
text-to-speech engine. OpenDec supports 17 vowel and 24 consonant phonemes for
American English (the default language).

## Supported Consonant Phonemes
| Phoneme   | Example   |   | Phoneme   | Example       |
|-----------|-----------|---|-----------|---------------|
| b         | **b**in   |   | nx        | si**ng**      |
| ch        | **ch**in  |   | p         | **p**in       |
| d         | **d**ebt  |   | r         | **r**ed       |
| dh        | **th**is  |   | rx        | o**r**ation   |
| dx        | ri**d**er |   | s         | **s**it       |
| f         | **f**in   |   | sh        | **sh**in      |
| g         | **g**ive  |   | t         | **t**est      |
| hx        | **h**ead  |   | th        | **th**in      |
| jh        | **g**in   |   | tx        | La**t**in     |
| k         | **c**at   |   | v         | **v**est      |
| l         | **l**et   |   | w         | **w**est      |
| lx        | be**ll**  |   | yx        | **y**et       |
| m         | **m**et   |   | z         | **z**oo       |
| n         | **n**et   |   | zh        | mea**s**ure   |

## Supported Vowel Phonemes
| Phoneme   | Example    |   | Phoneme   | Example      |
|-----------|------------|---|-----------|--------------|
| aa        | f**a**ther |   | ih        | b**i**t      |
| ae        | b**a**t    |   | ir        | b**eer**     |
| ah        | b**u**t    |   | iy        | b**ea**t     |
| ao        | b**ou**ght |   | or        | b**ore**     |
| aw        | b**ou**t   |   | ow        | b**oa**t     |
| ax        | **a**bout  |   | oy        | b**oy**      |
| ay        | b**i**te   |   | rr        | b**ir**d     |
| eh        | b**e**t    |   | uh        | b**oo**k     |
| el        | bott**le** |   | ur        | b**oor**     |
| en        | butt**on** |   | uw        | l**u**te     |
| er        | b**ear**   |   | yu        | c**u**te     |
| ey        | b**a**ke   |   |           |              |

## Non-Pronounced Phonemes
Some phonemes are not pronounced. Some phonemes can either represent silence,
and others can apply a specific type of stress to the preceding vowel phoneme
(because the text-to-speech engines cannot stress consonant phonemes).

| Phoneme   | Purpose                                       |
|-----------|-----------------------------------------------|
| ,         | Silence (whatever the comma pause value is)   |
| .         | Silence (whatever the period pause value is)  |
| _         | Silence (length specified by user)            |
| '         | Primary Stress                                |
| `         | Secondary Stress                              |
| "         | Emphatic Stress                               |


# Appendix B: Phoneme Pitch
| Pitch | Note  | Frequency (Hz)    |   | Pitch | Note  | Frequency (Hz)    |
|------:|-------|:------------------|---|------:|-------|:------------------|
|     1 |    C2 |                65 |   |    20 |    G  |               196 |
|     2 |    C# |                69 |   |    21 |    G# |               208 |
|     3 |    D  |                73 |   |    22 |    A  |               220 |
|     4 |    D# |                77 |   |    23 |    A# |               233 |
|     5 |    E  |                82 |   |    24 |    B  |               247 |
|     6 |    F  |                87 |   |    25 |    C4 |               261 |
|     7 |    F# |                92 |   |    26 |    C# |               277 |
|     8 |    G  |                98 |   |    27 |    D  |               293 |
|     9 |    G# |               103 |   |    28 |    D# |               311 |
|    10 |    A  |               110 |   |    29 |    E  |               329 |
|    11 |    A# |               116 |   |    30 |    F  |               348 |
|    12 |    B  |               123 |   |    31 |    F# |               370 |
|    13 |    C3 |               130 |   |    32 |    G  |               392 |
|    14 |    C# |               138 |   |    33 |    G# |               415 |
|    15 |    D  |               146 |   |    34 |    A  |               440 |
|    16 |    D# |               155 |   |    35 |    A# |               466 |
|    17 |    E  |               164 |   |    36 |    B  |               494 |
|    18 |    F  |               174 |   |    37 |    C5 |               523 |
|    19 |    F# |               185 |   |       |       |                   |


# Appendix C: Voice Characteristics
All voices have several characteristics that can be adjusted. The table below
describes all of the supported characteristics, their minimum and maximum
values, and the unit of measurement they use (for reference).

## Vocal Tract Parameters
| Parameter | Min   | Max   | Unit  | Function                      |
|-----------|------:|------:|------:|-------------------------------|
| sx        | 0     | 1     | N/A   | Sex (0:feminine, 1:masculine) |
| hs        | 65    | 145   | %     | Head size                     |
| f4        | 2000  | 4650  | Hz    | Fourth formant frequency      |
| f5        | 2500  | 4950  | Hz    | Fifth formant frequency       |
| b4        | 100   | 2048  | Hz    | Fourth formant bandwidth      |
| b5        | 100   | 2048  | Hz    | Fifth formant bandwidth       |

### `sx` - Sex
Different sexes can have difference voice characteristics such as head size,
pharynx length, larynx mass, breathiness, etc. The `sx` characteristic allows
the text-to-speech engine to use different values for a combination of different
voice characteristics, simplified into one value.

### `hs` - Head Size
A different head size will produce different voices. A larger head will produce
lower notes, and resonate at lower notes as well. The opposite happens with
smaller head sizes.

### `f4`, `f5`, `b4`, `b5` - Higher Formants
Child voices have 3 formant resonances, female voices have 4, and male voices
have 5. Formants produce peaks in the spectrum that become more prominent as the
bandwidth gets smaller. To make a resonance disappear, set the frequency to
2500Hz and the bandwidth to 2048Hz.

Some recommended values for these settings:
* `f5` must be at least 300 Hz higher than `f4`
* `f4` must be at least 3250Hz if `sx` is male
* `f5` must be at least 3700Hz if `sx` is female
* If `hs` is not 100, the values must be multiplied by (hs/100)

## Voicing Sound Source Parameters
| Parameter | Min   | Max   | Unit  | Function                                              |
|-----------|------:|------:|------:|-------------------------------------------------------|
| br        | 0     | 72    | dB    | Breathiness                                           |
| lx        | 0     | 100   | %     | Lax breathiness                                       |
| sm        | 0     | 100   | %     | Smoothness (high frequency attenuation)               |
| ri        | 0     | 100   | %     | Richness                                              |
| nf        | 0     | 100   | N/A   | Number of fixed samplings of glottal pulse open phase |
| la        | 0     | 100   | %     | Laryngealization                                      |

## Intonation Parameters
| Parameter | Min   | Max   | Unit  | Function      |
|-----------|------:|------:|------:|---------------|
| bf        | 0     | 40    | Hz    | Baseline fail |
| hr        | 2     | 100   | Hz    | Hat rise      |
| sr        | 1     | 100   | Hz    | Stress rise   |
| as        | 0     | 100   | %     | Assertiveness |
| qu        | 0     | 100   | %     | Quickness     |
| ap        | 50    | 350   | Hz    | Average pitch |
| pr        | 0     | 250   | %     | Pitch range   |

## Gain Adjustment Parameters
| Parameter | Min   | Max   | Unit  | Function                              |
|-----------|------:|------:|------:|---------------------------------------|
| lo        | 0     | 86    | dB    | Loudness of the voice                 |
| gv        | 0     | 86    | dB    | Gain of voicing source                |
| gh        | 0     | 86    | dB    | Gain of aspiration source             |
| gf        | 0     | 86    | dB    | Gain of frication source              |
| g1        | 0     | 86    | dB    | Gain of cascade formant resonator 1   |
| g2        | 0     | 86    | dB    | Gain of cascade formant resonator 2   |
| g3        | 0     | 86    | dB    | Gain of cascade formant resonator 3   |
| g4        | 0     | 86    | dB    | Gain of cascade formant resonator 4   |

## Default Parameters For Pre-Defined Voices
| Parameter | Paul  | Harry | Frank | Dennis    | Betty | Ursula    | Wendy | Rita  | Kit   |
|-----------|------:|------:|------:|----------:|------:|----------:|------:|------:|------:|
| **sx**    | 1     | 1     | 1     | 1         | 0     | 0         | 0     | 0     | 0     |
| **hs**    | 100   | 115   | 90    | 105       | 100   | 95        | 100   | 95    | 80    |
| **f4**    | 3300  | 3300  | 3650  | 3200      | 4450  | 4500      | 4500  | 4000  | 2500  |
| **f5**    | 3650  | 3850  | 4200  | 3600      | 2500  | 2500      | 2500  | 2500  | 2500  |
| **b4**    | 260   | 200   | 280   | 240       | 260   | 230       | 400   | 250   | 2048  |
| **b5**    | 330   | 240   | 300   | 280       | 2048  | 2048      | 2048  | 2048  | 2048  |
|           |       |       |       |           |       |           |       |       |       |
| **br**    | 0     | 0     | 50    | 38        | 0     | 0         | 55    | 46    | 47    |
| **lx**    | 0     | 0     | 50    | 70        | 80    | 50        | 80    | 0     | 75    |
| **sm**    | 3     | 12    | 46    | 100       | 4     | 60        | 100   | 24    | 5     |
| **ri**    | 70    | 86    | 40    | 0         | 40    | 100       | 0     | 20    | 40    |
| **nf**    | 0     | 10    | 0     | 10        | 0     | 10        | 10    | 0     | 0     |
| **la**    | 0     | 0     | 5     | 0         | 0     | 0         | 0     | 4     | 0     |
|           |       |       |       |           |       |           |       |       |       |
| **bf**    | 18    | 9     | 9     | 9         | 0     | 8         | 0     | 0     | 0     |
| **hr**    | 18    | 20    | 20    | 20        | 14    | 20        | 20    | 20    | 20    |
| **sr**    | 32    | 30    | 22    | 22        | 20    | 32        | 22    | 32    | 22    |
| **as**    | 100   | 100   | 65    | 100       | 35    | 100       | 50    | 65    | 65    |
| **qu**    | 40    | 10    | 0     | 50        | 55    | 30        | 10    | 30    | 50    |
| **ap**    | 122   | 89    | 155   | 110       | 208   | 240       | 200   | 106   | 306   |
| **pr**    | 100   | 80    | 90    | 135       | 140   | 135       | 175   | 80    | 210   |
|           |       |       |       |           |       |           |       |       |       |
| **lo**    | 86    | 81    | 86    | 84        | 81    | 80        | 83    | 83    | 73    |
| **gv**    | 65    | 65    | 65    | 65        | 65    | 65        | 53    | 65    | 65    |
| **gh**    | 70    | 70    | 70    | 70        | 70    | 70        | 70    | 70    | 70    |
| **gf**    | 70    | 70    | 70    | 70        | 72    | 72        | 72    | 72    | 72    |
| **g1**    | 68    | 73    | 63    | 75        | 69    | 69        | 69    | 69    | 69    |
| **g2**    | 60    | 60    | 58    | 60        | 67    | 66        | 62    | 72    | 69    |
| **g3**    | 49    | 52    | 56    | 52        | 50    | 51        | 53    | 48    | 53    |
| **g4**    | 65    | 63    | 67    | 62        | 57    | 59        | 55    | 54    | 50    |
