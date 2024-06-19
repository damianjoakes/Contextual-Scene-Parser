* [The SuEDE Parser](#the-suede-parser)
  * [Rulesets](#rulesets)
    * [Writing Rulesets](#writing-rulesets)
  * [Streams](#streams)
    * [One Rulesheet, Multiple Streams](#one-rulesheet-multiple-streams-)
    * [Lexical Stream](#lexical-stream-)
    * [Linter Stream](#linter-stream-)
  * [The View, the Viewbinder, and the Binder](#the-view-the-viewbinder-and-the-binder)
    * [Views](#views)
    * [The Viewbinder](#the-viewbinder)
    * [The Binder](#the-binder)


# The SuEDE Parser

## Rulesets

### Writing Rulesets
Rulesets are written in JSON format.

## Streams

### One Rulesheet, Multiple Streams  
The parser should be able to use one single rulesheet from a
plugin or other source and create tokens for all associated 
streams, without significant rewrites and additions to the
rulesheet.

### Lexical Stream  
The lexical stream produces tokens that contain lexical data, 
such as what tokens are where and what they mean in their 
context. The renderer relies on the lexical stream to assign 
CSS classes to the output tokens from the input code, and to
get the user's current position in the file.

### Linter Stream  
The linter stream produces tokens that contain linting data,
such as what errors are where, and an analysis of the code. The 
linter relies on the linter stream, and uses these tokens to
perform code validation.



## The View, the Viewbinder, and the Binder

### Views

_(What is a subview? A subview is a small component of the view represented. Subviews
don't always contain explicit code from the source code, they only contain bits
and pieces needed to make up an entire view.)_

Views are typically associated with rvalues and code blocks. A view can be thought 
of as a _"window into an action"_. A view can be something as simple as an 
assignment operation, or as complex as it needs to be thereafter. Let's remove the 
binder from a view to see what's going on.

<div style="flex: 1 0 auto;display: flex;flex-direction: column;border-left: 3px solid;border-bottom: 1px dashed;padding: 8px 8px;">
    <sub>Figure 1</sub>
    <br />
    <p style="margin: auto;">[viewbinder] (condition)</p>
    <div id="conditional" 
    style="flex: 1 0 auto;display:flex;justify-content: space-around">
        <div style="flex: 0 1 auto;display: flex;flex-direction: column">
            <p style="margin: auto">[condition] (comparison) i < 0</p>
            <p style="margin: auto">↓</p>
            <p style="margin: auto;border-bottom: 1px dashed">(return) i</p>
        </div>
        <div style="flex: 0 1 auto;display: flex;flex-direction: column">
            <p style="margin: auto">[condition] (mirror) i >= 0</p>
            <p style="margin: auto">↓</p>
        <p style="margin: auto;border-bottom: 1px dashed">(return) (undefined)</p>
        </div>
    </div>
</div>

This contains a _"window"_ into a comparison operation and shows what value the 
view is going to send back to the viewbinder.

The parenthesized words are `contexts`. Views are made of few-to-many contexts
depending on the complexity of the data that is contained within them. The parser 
comes with no built-in contexts. It's the job of library authors to create rulesets 
for the parser to create contexts. 

The words in square brackets are `carriages`. A carriage describes where the 
described context is going to apply its outputs. This is almost always either the
parent of the context, or the viewbinder. 

The combination of contexts and carriages describes _how_ a set of data will be 
interpreted, and _where_ the result of that interpretation will be stored. This 
serves as the foundation of the parser's side of code analysis. 

Views and subviews do not inherit carriages, however, the parser will flag views with 
no children with a viewbinder carriage as "useless", since everything from method
calls to variable assignments to function calls should eventually contain a call to 
the viewbinder. If a view contains no calls to the viewbinder, it never manipulates 
any data and nothing is returned back from it, then the view has no purpose.

This behavior can be changed on a context-to-context basis. For example, even though
the view pictured above contains no carriage on the `(return)` context, the ruleset
used for this mystery language states to always send the `(return)` context to the 
top-level view's carriage. This means that the result of this subview will never 
have to be evaluated, every subview with a `(return)` context will always send  
its result to the top-level's carriage.

Views and subviews will, however, inherit contexts. Everything down to 
the "return" subview inherits the top level `(condition)` context. This means that
the last line is technically a view with a `(condition comparison return)` context.

### The Viewbinder

The viewbinder is typically associated with assignment operators or return 
statements. The viewbinder represents calls to move data into the binder.

The viewbinder is the intersection between the view and the binder that marks when
values are declared. In figure 1 above, since this is one view which contains one 
`[viewbinder]` carriage which all subviews point towards, the viewbinder for 
this example will have one point for this view which contains the returned value
from the view, through the `[viewbinder]` carriage at the top level. 

### The Binder

The binder is typically associated with assignment keywords and lvalues/symbols. 
Things like `const` and `let` in many languages, as well as variable names like 
`my_variable` or `x` are contained within the binder. 

The binder is given its values from views by the viewbinder. The binder contains 
the scope of current views loaded into the parser. All lvalues referenced before the
current view are automatically carried into the current view via the binder. The
binder does not use the viewbinder to carry these values done, it's done inherently,
the viewbinder is only used by the views to move results into the binder.

