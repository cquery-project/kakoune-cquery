# kakoune-cquery

kakoune-cquery is a [kakoune](kakoune.org) extension for [cquery](https://github.com/jacobdufault/cquery), a low-latency language server supporting multi-million line C++ code-bases, powered by libclang.

kakoune-cquery is based on [danr/libkak](https://github.com/danr/libkak), a python library for interacting with kakoune and lsp.

These are the lsp features that libkak currently supports

* diagnostics
* find definition/references
* signature help (not supported by cquery)
* completion
* hover messages
* renaming

Additionally, kakoune-cquery supports these cquery specific features:

* semantic highlighting

![](https://image.ibb.co/j1nPnn/image.png)

## Quickstart
Currently, you need to start cquery using a script like this one:
```sh
#!/bin/bash
cquery --init='{
  "cacheDirectory": "/tmp/cquery-cache-dir/",
  "cacheFormat": "msgpack",
  "completion": {
    "detailedLabel": true
  },
  "xref": {
    "container": true
  }
}'   
```

Then, in your kakrc
```kak
# Commands for language servers
decl str lsp_servers %{
    cpp:path/to/cquery-start
    # You can add any other language servers for other filetypes here
    python:pyls 
}

# Example keybindings
map -docstring %{Goto definition}     global user . ':lsp-goto-definition<ret>'
map -docstring %{Select references}   global user ? ':lsp-references<ret>'
map -docstring %{Rename}              global user r ':lsp-rename<ret>'
map -docstring %{Hover help}          global user h ':lsp-hover docsclient<ret>'
map -docstring %{Next diagnostic}     global user j ':lsp-diagnostics next cursor<ret>'
map -docstring %{Previous diagnostic} global user k ':lsp-diagnostics prev cursor<ret>'

# Manual completion and signature help when needed
map global insert <a-c> '<a-;>:eval -draft %(exec b; lsp-complete)<ret>'
map global insert <a-h> '<a-;>:lsp-signature-help<ret>'

# Hover and diagnostics on idle
hook -group lsp global NormalIdle .* %{
    lsp-diagnostics cursor
    lsp-hover cursor
}

# Aggressive diagnostics
hook -group lsp global InsertEnd .* lsp-sync

```

Then attach `cquery.py` to a running kak process. Say it has PID 4032 (you see
this in the lower right corner), then issue:

    python cquery.py 4032

If you want to start this from Kakoune add something like this to your kakrc:

```kak
def cquery-start %{
    %sh{
        ( python path/to/cquery-kakoune/cquery.py $kak_session
        ) > /dev/null 2>&1 < /dev/null &
    }
}
```

## Semantic Highlighting

To see semantic highlighting, you have to add the faces to your colorscheme. There are currently no defaults.

```kak
face cqueryTypes                 type
face cqueryEnums                 type
face cqueryTypeAliases           type
face cqueryTemplateParameters    variable
face cqueryFreeStandingFunctions function
face cqueryMemberFunctions       yellow,  default +i
face cqueryStaticMemberFunctions yellow,  default
face cqueryFreeStandingVariables variable
face cqueryGlobalVariables       variable
face cqueryStaticMemberVariables variable
face cqueryMemberVariables       blue,    default +i
face cqueryParameters            variable 
face cqueryEnumConstants         variable
face cqueryNamespaces            module
face cqueryMacros                red,     default
```
