#Linter Manager
=============

Linter manager is a scratch plugin created to facilitate the task to check errors
in your projects. Linters are tools created to check files against a set of rules
designed to avoid a variety of common errors and particular cases, and this plugin
will help to bring that utility to scratch.

##Features
--------

* Easy Configuration
* Full integration with scratch
* Support for multiple programming languages

##Usage
-----

* First enable the plugin.
* Make sure you have installed the appropiate linter in your system.
* Check your configuration and make sure that the command call is correct that
 the regex is also correct.
* The proper linter is chossen acording to the current programming language
(Syntax Highlight) selected. Ie: left click Syntax Highlight -> x Javascript will
run the linter for Javascript
* To run the check to your just click in the linter icon (A Document with the
 check mark)
* In the botom part of the window, there is a section with all the errors listed
and the position (line and colum). With the left click you can copy the error
 message.

##Configuration
-------------

Inside the configuration dialog you will see the list of linter profiles created.
Here you can:

* Add a new profile, for a new language.
* Edit a profile.
* Delete a profile
* Restore Defaults. This will reset all profiles to the Default settings and will
 erase all your current Configuration.

###Profile
-------

Every profile consist of a unique language (Syntax), a command to call the linter
, and a regular expression (regex for short) to evaluate the linter output. The
 regex is used to give the feedback in the editor.

* Language:

    Click in the button and select the appropriate language syntax. There can be
    only one profile for language.

    i.e. html, php, javascript.

* Command.

    This is the command that will call the linter. You need the name (or the
    fullpath if the linter is installed outside of the directories of executable
    search path.) and the arguments.

    i.e.

        tidy -errors -quiet -utf8

        /opt/bin/csslint --format=compact

* Regex

The regex engine is provided by the glib library, and the syntax rules are
described by the Gnome Developer Documentation you can check
[here](https://developer.gnome.org/glib/stable/glib-regex-syntax.html)


The regex is the part of the magik used to integrate the linter to scratch.
First scratch call the linter application (it gives the file path) and it
 recieves a text output from the program.

Consider this output for html-tidy

        line 104 column 9 - Warning: inserting implicit &lt;body&gt;
        line 104 column 9 - Warning: missing &lt;/span&gt; before &lt;link&gt;
        line 106 column 9 - Warning: discarding unexpected &lt;/span&gt;
        line 107 column 9 - Warning: missing &lt;/span&gt; before &lt;link&gt;
        line 109 column 9 - Warning: discarding unexpected &lt;/span&gt;
        line 104 column 9 - Info: &lt;body&gt; previously mentioned


You can see that there's a lot of information. The line number, the color number
the kind of "Warning level" (here Error, Warning and Info) and the actual error
message (ie: missing &lt;/span&gt; before &lt;link&gt;)


The regex used for evaluation looks like this:
```
line (?<line>\d+) column (?<col>\d+) - (?:(?<err>Error)|(?<war>Warning)|(?<inf>Info)): (?<message>.+)
```
* the first word **line** is known as a literal, for the regrex any character
 simply means "find that character" (except for special cases) in this case it
 means find a string that contains an "l" followed by an "i" followed by a "n"
 followed by an "e". The same is for empty spaces. "   " (three spaces) will find
 three spaces.
* The parenthesys "(" & ")" are for grouping, everything inside is called a sub
 expresion.
*  ?&lt; &gt; are used to create a named sub-expression. In this example there
 are 6. line col err war inf message. They are used internally for the Linter
 Manager.
* All the reversed slash "\" means that the next character has a special meaning,
 in this case the "\d" is translated as one expresion and it means one digit
 a number from 0 to 9.
* The plus sign "+" and the asterisk "\*" have a special meaning, the first mean
 one or more so "\d+" one or more digit (ie: 0, 1000, 7000) in the case of an
 asterisk "\d\*" it will mean zero or more.
 For example "\d\d\*" and "\d+" are equivalent expresions. Other example "lines\*"
 will match line or lines or linessss
* The "?:" means that this sub expresion is optional.
* The vertical bar or pipe "|" is an or. The regex evaluates the first
 sub-expresion if succeds then skips the rest, if not it will try one by one
 until finds one or fails.
* Finally ".+" (dot plus) the dot is a special character it means "any character"
 the regres search line for line, so the expression will find all the characters
 until the end of the line.

Remember the regex needs to have: two named sub-expressions "?&lt;line&gt;
 "?&lt;col&gt; that return decimal numbers.

Three named expresion that return the warning status "?&lt;err&gt;" "?&lt;war&gt;"
 "?&lt;info&gt;" should return the expresion used for the error, warning or info.
 But they are optional.

And of course the "?&lt;message&gt;" that returns the message.