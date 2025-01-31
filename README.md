# s5LanguageTool
The whole LanguageTool is documented with Emmy Annotation. If you use Visual Studio Code, for example, you can download the lua-extension by sumneko which includes Emmy Annotation support.

The LanguageTool is quite large too (also because of this Emmy annotation) and covers a vast amount of functions for multilingual support (It very well may be that it still misses some functions!). It also very well be that this LanguageTool might be a little bit of an **overkill** for small maps with **little amount of text**.

Furthermore, the calls of StartCutscene or AddTribute may only work if you have the correct Comfort-Functions in your script. Otherwise nothing will happen.

## Table of contents
* [Why a language tool](#why-a-language-tool)
* [Including Languages](#including-languages)
* [General Pattern](#general-Pattern)
* [Briefings and Cutscenes](#briefings-and-cutscenes)
* [Check for missing keys](#check-for-missing-keys)
* [Adding your own multilingual function](#adding-your-own-multilingual-function)

## Why a language tool

Freeplay- and campaignmaps have been created by the developers in such a way that they can be offered in several languages. For this purpose, a briefing uses an XML file that is located in a folder with the respective language version of the game (when using the German language package, the XML file for this is located in the folder "\extra2\shr\text\de\InGame").

Unfortunately, this possibility of loading strings from an XML file is restricted to preinstalled maps. Map creators cannot access an XML file from their map, let alone any other file except LUA files (which is a good thing). However, this makes it very time-consuming to offer one's own maps in several languages. The common approach is to upload one's map in different languages multiple times.
This tool is intended to make this easier and avoid uploading maps in multiple languages over and over again. It provide a basis that can be used, with the most common functions already covered:

* A window displays the language selection, with which a user can choose between a set of languages. The game is also paused for this purpose, so that the language selection does not interfere with the map's timing.
* Any functions that are to be multilingual can be offered as multilingual via the tool, either with the already implemented ones or custom made ones.
* Special characters that exist in many languages are "automatically" converted to UTF-8. (See [Including Languages](#including-languages))
* Briefings, Messages and the like need little adaptation to support several languages at the same time.

## Including Languages

If you have copied the LanguageTool as a Comfort-Function into your mapscript (or include the lua file), you can call the following function, in order to display the language-selection-window (best as a call in the FMA). As long as the window is displayed, the game is paused:
`LanguageTool.DisplayLanguageSelection(state, callback, parameters)`
* `state` (number) = optional, but must be set if a callback function is passed; whether the window should be displayed or not. 0 = invisible, 1 = visible.
* `callback` (function) = optional; the function that is called after confirming the language selection.
* `parameters` (table) = optional; the parameters to be passed to the callback function when it is called.

This window will be pretty useless, as you cannot select any language as none is set for selection yet.
In order for languages to become available for selection, they must be added first. Languages can be added by the following function:

`LanguageTool.AddToLanguageSelection(id, name, title, charset)`
* `id` (string) = the deterministic ID of the language. (Recommended is the language abbreviation according to ISO 639-1)
* `name` (string) = the name of the language (preferably in the mother tongue, e.g. Deutsch, English, Polski, Français, ...)
* `title` (string) = the title displayed at the top window when the language is "hovered over".
* `charset` (table) = (optional) the charset represents the special characters of the respective language. The charset is used to encode these special characters in  texts to UTF-8 (most commonly that is). Since, Settlers 5 has problems displaying special characters in briefings, these would have to be replaced in the briefings by UTF-8 encoding, e.g. by a special function. For the german version there has been a solution with the "Umlaute"-Functin that does exactly this. The given charset is therefore nothing else, but it works for all languages, as long as the charset is given.

For example, this is how you add German to the language selection: 
```
LanguageTool.AddToLanguageSelection("de", "Deutsch", "Drücke ENTER um in deutsch zu Spielen", {
    {"ä", "\195\164"},
    {"ö", "\195\182"},
    {"ü", "\195\188"},
    {"ß", "\195\159"},
    {"Ä", "\195\132"},
    {"Ö", "\195\150"},
    {"Ü", "\195\156"}
})
```
The special characters are specified with their UTF-8 code, which allows them to be replaced in texts. This may also work as to replace any "ó" with an "o", as there is no representation for that letter in settlers.

It is important to **first** add the languages to the LanguageTool **before** displaying the selector.

Since version 2.5 the LanguageTool now comes with predefined LanguageSets: German, English, French and Polish. They can be used in the following manner:
```
LanguageTool.AddToLanguageSelection(
    LanguageTool.GERMAN_PRESET.id, 
    LanguageTool.GERMAN_PRESET.text, 
    LanguageTool.GERMAN_PRESET.title, 
    LanguageTool.GERMAN_PRESET.charset
)
```

## General Pattern

If the LanguageTool has been successfully initialised and several languages have been added, the Briefings, Messages, Cutscenes, etc... must be adapted next. Basically, all multilingual text outputs are built on the same pattern: Each function that origninally receives a string, that should output in multiple languages, can either receive a string that is valid for all languages or a table (with a certain pattern). Let's take the regular "Message"-Function as an example. The LanguageTool decides as follows:

* If the value passed is a *string*, it will be used for **all** languages (No replacement of the special characters is exectued!)
* If the value passed is a *table*, a *key* is searched that *corresponds to the selected language's id*.
* If the value passed is a *table* and *no key can be found* that corresponds to the selected language, the *key shared* is searched for. If this is found, it is will be used.
* If the value passed is a *table* and *no key and no shared* can be found, an *error text is returned as a string*.
* As from version 2.2 the *prefix* key can now be set, allowing to pass a value that will be placed in front of every returned string. The prefix key will automatically add an empty string (prefix.." ") at the end of it.

You can look at the shared key as a "default option" in a switch-statement in other programming languages.  
As an example, it is assumed that the LanguageTool has been initialised with languages with the ids "de", "en", "pl" and "fr". In the following example, a simple text output (Message) is used to show how the LanguageTool selects the proper language-string.

```
-- Every selected language will display "Example Text"
Message("Example Text") 

-- Every selected language, except with the id "de" and "en", will display "Example text for shared".
-- If the language with the id "de" is selected, "Beispieltext für de" will be displayed.
-- If the language with the id "en" is selected, "Example text for en" will be displayed.
Message({
    shared = "Example text for shared",
    de = "Beispieltext für de",
    en = "Example text for en"
})

-- If the language with the id "de" is selected, "Beispieltext für de" will be displayed.
-- If the language with the id "en" is selected, "Example text for en" will be displayed.
-- If the language with the id "pl" is selected, "Przykładowy tekst dla pl" will be displayed.
-- Since no key is set for the id "fr", if "fr" would be the selected language an error message would be displayed saying "LanguageTool: No key was found for the selected language with the key "fr" "
Message({
    en = "Example text for en",
    de = "Beispieltext für de",
    pl = "Przykładowy tekst dla pl"
})

-- As from version 2.2, the prefix-key allows to pass a string that will prepend every returned string, no matter if a key was found.
-- In this case, every string will start with "@color:0,255,0", allowing every returned language string
-- to appear green.
-- If the language with the id "de" is selected, "Beispieltext für de" will be displayed.
-- If the language with the id "en" is selected, "Example text for en" will be displayed.
-- If the language with the id "pl" is selected, "Przykładowy tekst dla pl" will be displayed.
-- If the language with the id "fr" is selected, "Exemple de texte pour fr" will be displayed.
Message({
    prefix = "@color:0,255,0"
    en = "Example text for en",
    de = "Beispieltext für de",
    pl = "Przykładowy tekst dla pl",
    fr = "Exemple de texte pour fr"
})
```

For these effects to take place it is **utterly neccessary** to either call `LanguageTool.Init()` or `LanguageTool.DisplayLanguageSelection`.
Otherwise you will run (sooner or later) into code errors.

## Briefings and Cutscenes

Since briefings (cutscenes and such) are a more complex topic, they are presented below in an additional example. The cutscene's text and title work the same way as the example of the briefings text and title does. Suppose we have the following briefing:
```
local briefing = {}

table.insert(briefing, {
    title       = "Totally viable title",
    text        = "Totally viable text",
    position    = GetPosition("position")
})

StartBriefing(briefing)
```
In this example, all selected languages would display title and text in the same way, so nothing would have changed from the regular StartBriefing. Suppose we want to display a different text for the language "de", "en" and "pl". We would solve this as follows:
```
local briefing = {}

table.insert(briefing, {
    title       = "Totally viable title",
    text        = {
        de = "Vollkommen brauchbarer Text",
        en = "Totally viable text",
        pl = "Tekst całkowicie wykonalny"
    },
    position    = GetPosition("position")
})

StartBriefing(briefing)
```
In this case, the languages with the id "de", "en" and "pl" would now display different texts. For the language with the id "fr" we would get an error message as a  text. However, all languages would still display the same title: "Totally viable title". For the next step, we therefore want to give the languages "de" and "en" the same title and "pl" a different one:
```
local briefing = {}

table.insert(briefing, {
    title       = {
        shared = "Totally viable title"
        pl = "Całkowicie realny tytuł" 
    },
    text        = {
        de = "Vollkommen brauchbarer Text",
        en = "Totally viable text",
        pl = "Tekst całkowicie wykonalny"
    },
    position    = GetPosition("position")
})

StartBriefing(briefing)
```
Now we would have the same title for "de" and "en" (and also "fr") and our own title for "pl".  
For "de", "en" and "pl" we would therefore have a different text and for "fr" an error message as text.

It is also very important to **initialise the LanguageTool before adding tables as title and text**. Otherwise you will quickly run into an error that can only be solved by Alt+F4 the game, since the regular StartBriefing can't handle tables for their text and title. The Briefing page would just get stuck at this point.

If we want to create multiple-choice briefings we must replace the firstText and secondText of the mc-table with the same principle:
```
    ...
mc   = {
    title = "Title for everyone",
    text = "Text for everyone",
    firstText = {
        de   = "Erster Text für de",
        en   = "First text for en",
        pl   = "Pierwszy tekst do pl"
    },
    secondText = {
        de  = "Zweiter Text für de",
        en  = "Second text for en",
        pl  = "Drugi tekst do pl"
    },
    firstSelected  = 2,
    secondSelected = 4,
},
    ...
```

The same goes for quests.
The concept stays the same across all functions that may take text that should be multilingual. Here is the full list of all functions implemented in the LanguageTool currently:

* Message
* StartBriefing
* StartCutscene (requires comfort-function)
* SetPlayerName
* CreateNPC
* LanguageTool.AddQuest -> A comfort-function to add quests outside of briefing pages.
* AddTribute (requires comfort-function)

## Check for missing keys

As in version 2.3, a new functionionality was added: `LanguageTool.EnableLanguageCheck(_flagStrings)` (disabled by default).  
With this function it is now possible to assist in creating multilingual Briefings, Messages and such. By calling this function, the `GetString` function passively checks, if all languages have a valid key representation within the passed table. If it is not the case, a warning is appended to the string returned by the function, which looks like this: "LanguageTool: No key/(s) was/were found for the id/(s): ..."

It is important to mention that this **does not work** as expected, if the **given value is a string**, like the title in this case:
```
{
    title = "Test Title",
    text = {
        de = "Test for de",
        en = "Test for en"
    }
}
```
Since a string as value is a valid key itself, a warning wouldn't make much sense here. It would, however, return that for the languages with the id "pl" and "fr" no key is set.

As in version 2.4, an additional parmater can be passed to the `LanguageTool.EnableLanguageCheck(true)` function. This enables a flag, which marks every string to the `GetString` function, that the passed value is valid because it is a string. This helps to identify false positives, as it may not be clear if a string is left accidentally or on purpose. Like in the example above, if the flag is set to true, the title would now be appended with the string "LanguageTool: The valid key is a string.", to quickly spot any accidentally left out strings.

## Adding your own multilingual function

If there is a need to write a separate function to display multilingual text, if you may want to have your own function multilingual or a function that is not covered by the LanguageTool yet, the function `LanguageTool:GetString(_table, _returnInput)` can be used.
This is the main function to find the correct keys from a table (as seen throughout the examples above). Important to note is, if no matching key is found a error string is returned by default. If you set a flag called `_returnInput` (a boolean) to true, the passed input will be returned instead of an error message:
```
-- Your own function that does something with the text that should be multilingual
function Test(_text)
    ...
end

-- The adapted function of the LanguageTool
function LanguageTool.Test(_table)
    _table = LanguageTool:GetString(_table) -- will either return the correct string for the selected language's key or an error message as string.
    _table = LanguageTool:GetString(_table, true) -- will either return the correct string for the selected language's key or _table (the input)
    
    Test(_table)
end
```
