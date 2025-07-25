# Translating Cataclysm: DDA

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
*Contents*

- [Translators](#translators)
  - [Getting Started](#getting-started)
  - [Glossary](#glossary)
  - [Grammatical gender](#grammatical-gender)
  - [Tips](#tips)
- [Developers](#developers)
  - [Translation Functions](#translation-functions)
    - [`_()`](#_)
    - [`pgettext()`](#pgettext)
    - [`n_gettext()`](#n_gettext)
  - [`translation`](#translation)
  - [Static string variables](#static-string-variables)
  - [Recommendations](#recommendations)
- [Maintainers](#maintainers)
  - [Automated updates](#automated-updates)
  - [Manual updates](#manual-updates)
  - [Language stats](#language-stats)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Translators

The official location for translating Cataclysm: DDA is the
[Transifex translations project][1].

Some of the currently supported languages are:

* Arabic
* Bulgarian
* Chinese (Simplified)
* Chinese (Traditional)
* Dutch
* French
* German
* Italian (Italy)
* Japanese
* Korean
* Polish
* Portuguese (Brazil)
* Russian
* Serbian
* Spanish (Argentina)
* Spanish (Spain)
* Turkish

Don't see your language in the list above? You can add it into the project at
Transifex!

If you have any questions or comments about translation, feel free to post in
the [Translations Team Discussion][2] subforum.

### Getting Started

To begin translating, head over the [translation project][1] and click on the
"Help Translate Cataclysm: DDA" button.
This should take you to a page where you can either create a free account on
Transifex, or login using GitHub, Google+ or LinkedIn.

![Start translating](img/translating-start.png)

After you've created your account, return to the [translation project][1] and
click on the "Join team" button.
This will open a window where you can choose the language you are interested on
translating, so pick one and click the "Join" button.

![Join project](img/translating-join.png)

After this, the most straightforward thing to do is to reload the page,
which should redirect you to the translation project's dashboard.
Here, you can click the "Languages" link on the sidebar to see the list of
supported languages and the current progress of the translation effort.

Note that you can request for the inclusion of additional languages,
if the one you are interested in is not available on the list.

![Language list](img/translating-list.png)

From this list, you can click on the language of your choice, and then click on
the "Translate" to get started right away. Otherwise, you can click on any
other language and click on the "Join team" button, if you are interested in
translating for that language as well.

After clicking on the "Translate" button, you will be taken to the web editor.
To begin, you need to choose a resource to translate. Most of the in-game text
is contained in the `master-cataclysm-dda` resource, so click on it to start.

![Choose a resource](img/translating-resource.png)

At this point, the editor should show you the list of text available for
translation, now you only need to click on the string you want to translate and
type your translation on the translation area on the right side of the screen.
Click on the "Save" button when you are satisfied with your translation.

![Web editor](img/translating-editor.png)

See [Transifex's documentation][3] for more information.

### Glossary

This glossary is intended to help explain some CDDA-specific terms and their
etymology in order to help translations.

* **Exodii**: The Exodii are a bunch of humans from another dimension.  When
  the Blob invaded their world, they managed to acquire enough technology to
  open portals of their own, and now they portal to worlds that have been
  attacked by the Blob and try to rescue survivors.  Exodii is a horrible
  mangling of "Exodus" - the word literally means leaving or going out and has
  connotations of forced emigration and refugees.  The Exodii are the people of
  an Exodus.  While Exodus is a Latin word and "ii" to indicate a plural is a
  Latin thing, but this isn't actually the [correct Latin
  plural](https://www.latin-is-simple.com/en/vocabulary/noun/9294/) for this
  word.

### Grammatical gender

For NPC dialogue (and potentially other strings) some languages may wish to
have alternate translations depending on the gender of the conversation
participants.  This two pieces of initial configuration.

1. The dialogue must have the relevant genders listed in the json file defining
   it.  See [the NPC docs](./JSON/NPCs.md).
2. Each language must specify the genders it wishes to use via the translation
   of `grammatical gender list`.  This should be a space-separated list of
   genders used in this language for such translations.  Don't add genders here
   until you're sure you will need them, because it will make more work for
   you.  If you need different genders than are currently supported you must
   add them to the `all_genders` lists in `lang/extract_json_strings.py` and
   `src/translations.cpp`.

Having done this, the relevant dialogue lines will appear multiple times for
translation, with different genders specified in the message context.  For
example, a context of `npc:m` would indicate that the NPC participant in the
conversation is male.

Because of technical limitations, all supported genders will appear as
contexts, but you only need to provide translations for the genders listed in
`grammatical gender list` for your language.

Other parts of the game have various ad hoc solutions to grammatical gender, so
don't be surprised to see other contexts appearing for other strings.

### Tips

There are issues specific to Cataclysm: DDA which translators should be aware of.
These include the use of terms like `%s` and `%3$d` (leave them as they are),
and the use of tags like `<name>`, which shouldn't be translated.

Information about these and any other issues specific to individual languages,
can be found in Cataclysm: DDA's [language notes folder][4].

General notes for all translators are in `README_all_translators.txt`,
and notes specific to a language may be stored as `<lang_id>.txt`,
for example `de.txt` for German.

Cataclysm: DDA has more than 50000 translatable strings, including all mods shipped
with the game but don't be discouraged. The more translators there are, the easier it
becomes 😄.

## Developers

Cataclysm: DDA uses a modified version of [GNU gettext][5] to display translated texts.

Using `gettext` requires two actions:

* Marking strings that should be translated in the source code.
* Calling translation functions at run time.

Marking translatable string allows for their automatic extraction.
This process generates a file that maps the original string (usually in English)
as it appears in the source code to the translated string.
These mappings are used at run time by the translation functions.

Note that only extracted strings can get translated, since the original string
is acting as the identifier used to request the translation.
If a translation function can't find the translation, it returns the original
string.

### Translation Functions

In order to mark a string for translation and to obtain its translation at
runtime, you should use one of the following functions and classes.

String *literals* that are used in any of these functions are automatically
extracted. Non-literal strings are still translated at run time, but they won't
get extracted.

#### `_()`

This function is appropriate for use on simple strings, for example:

```cpp
const char *translated = _( "text marked for translation" )
```

It also works directly:

```cpp
add_msg( _( "You drop the %s." ), the_item_name );
```

Strings from the JSON files are extracted by the `lang/extract_json_strings.py`
script, and can be translated at run time using `_()`. If translation context
is desired for a JSON string, `class translation` can be used instead, which is
documented below.

#### `pgettext()`

This function is useful when the original string's meaning is ambiguous in
isolation. For example, the word "blue", which can mean either a color or an
emotion.

In addition to the translatable string, `pgettext` receives a context which is
provided to the translators, but is not part of the translated string itself.
This function's first parameter is the context, the second is the string to be
translated:

```cpp
const char *translated = pgettext("The color", "blue")
```

#### `n_gettext()`

Some languages have complex rules for plural forms. `n_gettext` can be used to
translate these plurals correctly. Its first parameter is the untranslated
string in singular form, the second parameter is the untranslated string in
plural form and the third one is used to determine which one of the first two
should be used at run time:

```cpp
const char *translated = n_gettext("one zombie", "many zombies", num_of_zombies)
```

### `translation`

There are times when you want to store a string for translation, maybe with
translation context; Sometimes you may also want to store a string that needs no
translation or has plural forms. `class translation` in `translations.h|cpp`
offers these functionalities in a single wrapper:

```cpp
const translation text = to_translation( "Context", "Text" );
```

```cpp
const translation text = to_translation( "Text without context" );
```

```cpp
const translation text = pl_translation( "Singular", "Plural" );
```

```cpp
const translation text = pl_translation( "Context", "Singular", "Plural" );
```

```cpp
const translation text = no_translation( "This string will not be translated" );
```

The string can then be translated/retrieved with the following code

```cpp
const std::string translated = text.translated();
```

```cpp
// this translates the plural form of the text corresponding to the number 2
const std::string translated = text.translated( 2 );
```

`class translation` can also be read from JSON. The method `translation::deserialize()`
handles deserialization from a `JsonIn` object, so translations can be read from
JSON using the appropriate JSON functions. The JSON syntax is as follows:

```jsonc
"name": "bar"
```

```jsonc
"name": { "ctxt": "foo", "str": "bar", "str_pl": "baz" }
```

or

```jsonc
"name": { "ctxt": "foo", "str_sp": "foo" }
```

In the above code, `"ctxt"` and `"str_pl"` are both optional, whereas `"str_sp"`
is equivalent to specifying `"str"` and `"str_pl"` with the same string. Additionally,
`"str_pl"` and `"str_sp"` will only be read if the translation object is constructed using
`plural_tag` or `pl_translation()`, or converted using `make_plural()`. Here's
an example:

```cpp
translation name{ translation::plural_tag() };
jsobj.read( "name", name );
```

If neither `"str_pl"` nor `"str_sp"` is specified, the plural form defaults to
the singular form + "s". However, `"str_pl"` may still be needed if the unit
test cannot determine whether the correct plural form can be formed by simply
appending "s".

You can also add comments for translators by writing it like below (the order
of the entries does not matter):

```jsonc
"name": {
    "//~": "as in 'foobar'",
    "str": "bar"
}
```

Do note that the JSON syntax is only supported if a JSON value is read using
`translation`. If you want new json values to use this format, refer to
`translations.h|cpp` and read the strings with `translation`. Afterwards
you also need to update `extract_json_strings.py` and run `lang/update_pot.sh`
to ensure that the strings are correctly extracted for translation, and run the
unit test to fix text styling issues reported by the `translation` class.

If a string doesn't need to be translated, you can write `"NO_I18N"` in the
`"//~"` comment, and this string will not be available to translators (see [here](/doc/JSON/JSON_INFO.md#translatable-strings)):

```jsonc
"name": {
    "//~": "NO_I18N",
    "str": "Fake Monster-Only Spell"
}
```

### Static string variables

Translation functions should not be called when initializing a static variable.
For global static variables, calling these functions does nothing because the
translation system is not yet initialized. For local static variables, the
translation will only happen once and switching language in-game will not work
properly. Consider using translation objects (`to_translation()` or `pl_translation()`)
to mark the string for extraction and call `translation::translated()` on the
fly to ensure the string is properly translated each time.

Note if a string becomes translated in-game after you add a translation function
call to the initialization of a global static variable, it usually means a
translation call is already made when the string is used, and your newly added
translation call happens to mark the string for extraction. In this case, using
a translation object is also recommended to avoid calling the translation
function twice.

### Recommendations

In Cataclysm: DDA, some classes, like `itype` and `mtype`, provide a wrapper
for the translation functions, called `nname`.

When an empty string is marked for translation, it is always translated into
debug information, rather than an empty string.
On most cases, strings can be considered to be never empty, and thus always
safe to mark for translation, however, when handling a string that can be empty
and *needs* to remain empty after translation, the string should be checked for
emptiness and only passed to a translation function when is non-empty.

Error and debug messages must not be marked for translation.
When they appear, the player is expected to report them *exactly* as they are
printed by the game.

See the [gettext manual][6] for more information.

## Maintainers

### Automated updates

Under normal circumstances the translation files are updated automatically by a
weekly GitHub workflow called `pull-translations`.

### Manual updates

If for some reason you wish to update the translation files by hand, several steps need to be done in the correct order to correctly merge and maintain them.

There are scripts available for these, so usually the process will be as follows:

1. Download the translations in `.po` format.
2. Put them in `lang/incoming/`, ensuring they are named consistently with the files in `lang/po/`.
3. Run `lang/update_pot.sh` to update `lang/po/cataclysm-dda.pot`.
4. Run `lang/merge_po.sh` to update `lang/po/*.po`. (This is only used to test translations locally as the project now uses Transifex for translation)

    This will also merge the translations from `lang/incoming/`.

These steps should be enough to keep the translation files up-to-date.

To compile the .po files into `.mo` files for use, run `lang/compile_mo.sh`. It will create a directory in `lang/mo/` for each language found.

Also note that both `lang/merge_po.sh` and `lang/compile_mo.sh` accept arguments specifying which languages to merge or compile. So to compile only the translation for, say, Traditional Chinese (zh_TW), one would run `lang/compile_mo.sh zh_TW`.

After compiling the appropriate .mo file, if your system is using that language, the translations will be automatically used when you run Cataclysm.

If your system locale is different from the one you want to test, the easiest way to do so is to find out your locale identifier, compile the translation you want to test, then rename the directory in `lang/mo/` to your locale identifier.

So for example if your local language is New Zealand English (en_NZ), and you want to test the Russian (ru) translation, the steps would be `lang/compile_mo.sh ru`, `mv lang/mo/ru lang/mo/en_NZ`, `./cataclysm`.

You can also change the language in game options if both are installed.

### Language stats

We also store some statistics for how complete each translation is, in
`src/lang_stats.inc`.  This can be used for example to show end users some
information about the translations.

These stats are also normally updated by the GitHub workflow, but can be
updated by hand via `lang/update_stats.sh`.

[1]: https://www.transifex.com/cataclysm-dda-translators/cataclysm-dda/
[2]: https://discourse.cataclysmdda.org/c/game-talk/translations-team-discussion
[3]: https://docs.transifex.com/
[4]: ../lang/notes
[5]: https://www.gnu.org/software/gettext/
[6]: https://www.gnu.org/software/gettext/manual/index.html
