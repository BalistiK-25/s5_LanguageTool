# s5_LanguageTool
The whole LanguageTool is documented with Emmy Annotation. If you use Visual Studio Code, for example, you can download the lua-extension by sumneko which includes Emmy Annotation support.

## Table of contents
* [Why a language tool](#why-a-language-tool)
* [Including Supported Languages](#including-supported-languages)
* [Briefings and such](#briefings-and-such)

## Why a language tool

Free-play and campaign maps have been created by the developers in such a way that they can be offered in several languages. For this purpose, a briefing, for example, uses an XML file that is located in a folder with the respective language version of the game (when using the German language package, the XML file for this is located in the folder ("\extra2\shr\text\de\InGame").

Unfortunately, this possibility of loading strings from an XML file is only possible for preinstalled maps. Map creators cannot access an XML file from their map, let alone any other file except LUA files (which is a good thing). However, this makes it very time-consuming to offer one's own maps in several languages.
However, this tool is intended to make this easier, or to provide a framework that can be used:

* A window displays the language selection that this map supports. The game is also paused for this purpose so that the language selection does not inadvertently hinder the map's operations.
* Any functions that are to be multilingual can be offered as multilingual via the tool.
* Special characters that exist in many languages are automatically converted to UTF-8. (See [Including Supported Languages](#including-supported-languages))
* Briefings, messages and the like need little adaptation to support several languages at the same time.

The LanguageTool is quite large and covers a vast amount of functions for multilingual support. It very well may be that it still misses some functions. It also very well be that this LanguageTool might be a little bit of an **overkill** for small maps with **little amount of text**.

## Including Supported Languages

If you have copied the LanguageTool as a Comfort-Function into your mapscript, nothing will happen at first. You need to call the following function, in order to display the window (best as a call in the FMA). Note: As long as the window is displayed, the game is paused.
`LanguageTool.DisplayLanguageSelection(state, callback, parameters)`
* `state` (number) = optional; whether the window should be displayed or not. 0 = invisible, 1 = visible.
* `callback` (function) = optional; the function that is called after confirming the language selection.
* `parameters` (table) = optional; the parameters to be passed to the callback function when it is called.

However, this window will be pretty useless, as you cannot select any language.
In order for languages to become available for selection, they must be added first. The purpose of this is that the map developer himself should determine which languages he supports. Therefore, these must be added via the following function:

`LanguageTool.AddToLanguageSelection(id, name, title, charset)`
* `id` (string) = the deterministic ID of the language. (Recommended is the language abbreviation according to ISO 639-1)
* `name` (string) = the name of the language (preferably in the mother tongue, e.g. Deutsch, English, Polski, ...)
* `title` (string) = the title displayed at the top window when the language is "hovered over".
* `charset` (table) = (optional) the charset represents the special characters of the respective language. The charset is used to convert these special characters in the texts into UTF-8. Since, for example, Settler has problems displaying special characters in briefings, these would have to be replaced in the briefings by UTF-8 encoding, e.g. by an special function. This is automatically replaced by this tool, depending on the given charset.

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

## Briefings and such

