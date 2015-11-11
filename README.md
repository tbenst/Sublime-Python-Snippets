# Sublime-Python-Snippets
Contains a replacement for the def snippet in Python that supports type hinting &amp; automatically creates a Docstring with the argument list pre-populated. Supports PEP257. Written for Sublime Text 3.

# Example

To try, type def[tab][tab] and type some variables. They will be automatically populated under Args

```python
    def function(x, m: str="Supports \" quotes", y=['and', ["nested", 2], "arrays"],
                 z: str='newlines are fine', d: int) -> (None):
        """
        Heading.

        Description

        Args:
            x: 
            m: 
            y: 
            z: 
            d: 


        Returns:
            Description
        Tests:
            >>> 2+2
            4
        """
        pass
```

# Setup

1. install [PackageResourceViewer](https://github.com/skuroda/PackageResourceViewer) using [Package Control](https://packagecontrol.io/).
2. Using the Command Palette (cmd-shift-P on Mac), select PackageResourceViewer: Open Resource -> Python -> function.sublime-snippet.
3. Paste the text from the file in the repo & save. (To undo, simply delete ~/Library/Application Support/Sublime Text 3/Packages/Python/function.sublime-snippet, and the default is restored)
4. we need to disable auto-pairing in the scope of function parameters, as auto-pairing will disrupt the snippet. In "Settings - User" add the following:

        "auto_match_enabled": false,
        
5. To restore this functionality elsewhere, open your "Key Bindings - User" file and add the lines from  "Default (OSX).sublime-keymap" in this repo.
    
    I imagine other platforms are the same, but in case not, copy the code under  "// Auto-pair quotes" in "Key Bindings - Default", and replace all occurances of

        { "key": "setting.auto_match_enabled", "operator": "equal", "operand": true },
        
    with
    
        { "key": "selector", "operator": "not_equal", "operand": "meta.function.parameters.python", "match_all": true },

# The belly of the beast

Here's what the Regex looks like for substituting in the variable name in the Docstring. It was non-trivial to support Type Hinting, nested Lists, Tuples, Dicts, escaped quotes, etc.

    (?(DEFINE)(?'quote'(?:((?'q'["'])(?:(?!\k<q>|\\).)*(?:\\.(?:(?!\k<q>|\\).)*)*\k<q>))))
    
    (?(DEFINE)(?'delim'(?:,\s*)))
    
    (?(DEFINE)(?'paren'(?:(\((?>[^()]|(?&paren))*\)))))
    
    (?(DEFINE)(?'brac'(?:(\[(?>[^\[\]]|(?&brac))*\]))))
    
    (?(DEFINE)(?'curl'(?:(\{(?>[^\{\}]|(?&curl))*\}))))
    
    (?(DEFINE)(?'equal'(?:=\s?(?:(?P>quote)|(?>\w+)|(?P>brac)|(?P>curl)|(?P>paren))(?:(?P>delim)|\z))))
    
    (?'var'(?>\w+))((?>:\s\w+)(?:(?P>equal)|(?P>delim)|\z)|\s?(?P>equal)|(?P>delim)|\z)


The following function variable strings were used as tests as well (note not all are valid Python):

    x
    x = 4
    x: str
    x: str=5
    abc, y: str
    z: int, abc, h: int=4
    x: int, y, as: as, z: str="yay", abc
    x: int, y, as: as, z: str="yay", hah = [1, 2, 3, 4]
    z: int, abc = 4, h: int=4
    x: int, y, as: as, z: str="It 'even' supports \"escaped\" quotes", my_list = [1, 2, 3, 4], h: int
    x = [[1,2],[3,4]]
