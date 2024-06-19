# The Contextual Scene Parser

The Contextual Scene Parser (CSP) is a code parsing tool currently under development for use
in the SuEDE IDE.   

## Goals
- Language agnosticism
  - No differentiation between languages, the parser will parse all languages 
    the same, based on the grammar constraints passed into the parser.
- Simple integration using JSON.
  - Grammar constraints are defined through the use of JSON files.
- 