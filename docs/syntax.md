
# CherryTags Syntax

The CherryTags Syntax is a syntax for associating human-centric metadata with text.

## Summary

## Objectives and Non-Objectives

Objectives:

* to provide structured document metadata and generic compile-time directives
* metadata structure implicit in the syntax for most cases
* annotate code within comments (it's a code annotation language)
* metadata within formatted text
* parse parameters in a consistent way
* simple data structures providing English description or metadata
* Within the data annotations, able to provide formatted text
* No or minimal interference with markdown, HTML, handlebars, JavaDocs/JSDocs
* The meaning of a syntax in one context should suggest the meaning of the same syntax in other contexts.

Non-objectives:

* representing arbitrary data structures
* formatting text or marking up text with formatting
* not a syntax for comments; instead, a syntax for embedding metadata in comments and a syntax for the metadata structure

## Tagging Text

chunking ambient content

## Content Blocks

A 'content block' is a series of one or more lines of text. 

Each line tag occupies an entire line of text and possibly subsequent lines but doesn't contribute to the text of the content block. They 

Line tags in ambient sections do not necessarily attribute the entire ambient section. That depends on the enclosing syntax.


## Line Tags


The ambient content matches the `AmbientBlock` production rule and evaluates to either a text element or an empty element. 

```ebnf
AmbientSection ::= BL* AmbientBlock (NL BL+ AmbientBlock)* (NL BL* ST*)?;
AmbientBlock ::= AmbientTextBlock | TagBlock;
AmbientTextBlock ::= InitialTextLine (NL TrailingTextLine)*;
InitialTextLine ::= ST* ((ContentChar - '\' - '%') | '\' ST* ContentChar |
    '%' S | '%' (ContentChar - ReservedIdentifierChar)) (ContentChar | ST)*;
TrailingTextLine ::= ST* ContentChar (ContentChar | ST)*;
ReservedIdentifierChar ::= InitialIdentifierChar | '[' | [({'"`]

TagBlock ::= LineTag (NL LineTag)*;
LineTag ::= ST* '%' TagSpec (InlineDelim (InlineValues | InlineTaggedElem))? ST*;
TagSpec ::= Identifier ('.' Identifier)? | '.' Identifier;
Identifier ::= InitialIdentifierChar TrailingIdentifierChar*;
InlineDelim ::= ST+ (NL ST*)? | NL ST*

InlineValues ::= InlineValue MoreValues? ColonEndLineValue?;
MoreValues ::= (InlineDelim InlineValue)+ | (ST* ',' STN* InlineValue)+;

InlineTaggedElem ::= InlineTag (InlineDelim InlineTag)* ColonEndLineValue?;
InlineTag ::= '.' Identifier InlineValue?;
InlineValue ::= NonTextValue | QuotedValue | Symbol;

ColonEndLineValue ::= InlineDelim? ':' InlineDelim? EndLineValue;
EndLineValue ::= IF(NonTextValueIndicator, NonTextValue, EndLineText);
NonTextValueIndicator ::= '{' | '[' | '*' InitialIdentifierChar |
    Number ST* (NL | '}' | _EOS_);
EndLineText ::= ContentChar+ (ST+ ContentChar+)*;

NonTextValue ::= BracketedValue | Array | Reference | Number;
BracketedValue ::= '{' ValueSection? '}';
Array ::= '[' (InlineValue MoreValues?)? ']';
Symbol ::= Identifier ('.' Identifier)*;
Reference ::= '*' Symbol;
Number ::= Decimal | Hex;
Decimal ::= '-'? [0-9]+ ('.' [0-9]+)? ([Ee][-+]?[0-9]+)?;
Hex ::= '0x' [0-9A-Fa-f]+;

ValueSection ::= BL* ValueBlock (NL BL+ ValueBlock)* (NL BL* ST*)?;
ValueBlock ::= ValueTextBlock | TagBlock;
ValueTextBlock ::= InitialBalancedLine (NL TrailingBalancedLine)*
InitialBalancedLine ::= BalancedLine(BalancedLineChar - '%'| '%' S |
    '%' (ContentChar - ReservedIdentifierChar));
TrailingBalancedLine ::= BalancedLine(BalancedLineChar)
BalancedLineChar ::= ContentChar - '{' - '}' - '\' - '`'
BalancedLine(c) ::= (NestedBrackets NonBracketText(c)?)+ |
    NonBracketText(c) (NestedBrackets NonBracketText(c)?)*;
NonBracketText(c) ::= (NonBracketChar(c) BacktickEscape?)+ |
    BacktickEscape (NonBracketChar(c) BacktickEscape?)*;
NonBracketChar(c)::= c | EscapedChar | ST;
NestedBrackets ::= '{' NestedContent? '}';
NestedContent ::= NL? NestedTextBlock (NL BL+ ValueBlock)* (NL BL* ST*)? |
    (NL BL+ ValueBlock)+ (NL BL* ST*)?;
NestedTextBlock ::= TrailingBalancedLine (NL TrailingBalancedLine)*;

QuotedValue ::= SQuotedValue | DQuotedValue | BacktickEscape;
SQuotedValue ::= "'" SQuotedText? "'";
DQuotedValue ::= '"' DQuotedText? '"';
BacktickEscape ::= (n * '`') BacktickText(n)? '`' {n >= 1};

// TBD: should backticks support escaped chars?

SQuotedText ::= (ContentChar - "'" - '\' | EscapedChar | ST)+;
DQuotedText ::= (ContentChar - '"' - '\' | EscapedChar | ST)+;
BacktickText(m) ::= BacktickAtStart(m) | BacktickMayEnd(m)+;
BacktickAtStart(m) ::= (i * '`') BacktickMayEnd(m)* {1 <= i < m};
BacktickMayEnd(m) ::= (ContentChar - '`' | NST) (i * '`')? {1 <= i < m};

ContentChar ::= UnicodeChar - ASCIIControlChar - S;
UnicodeChar ::= ? any Unicode character ?
ASCIIControlChar ::= [\u0000-\u001F] | [\u007F]
EscapedChar ::= '\' ContentChar;

InitialIdentifierChar ::= '$' | '_' | UnicodeIDStart;
TrailingIdentifierChar ::= '$' | '_' | UnicodeIDContinue | UnicodeZWJ | UnicodeZWNJ;
UnicodeIDStart ::= ? any Unicode character with property 'ID_Start' ?;
UnicodeIDContinue ::= ? any Unicode character with property 'ID_Continue' ?;
UnicodeZWJ ::= \u200C;
UnicodeZWNJ ::= \u200D;

ST ::= S | Tab;
STN ::= S | Tab | NL;
NL ::= CR? LF;
BL ::= ST* ('\' ST*)? NL

Tab ::= \u0009;
CR ::= \u000D;
LF ::= \u000A;
S ::= \u0020;
_EOS_ ::= ? end of character stream ?;
```

## Modified EBNF

* Excludes commas. Spaces and operators delimit terms. Operator precedence, highest first: (`\`, `[]`), `()`, (`?`, `*`, `+`), `i * X`, ` ` (space), `-`, `|`, `::=`
* Intraline whitespace (ST, STN, NL) that is not the last token of a production rule only matches when the first required non-whitespace token to follow within the production rule also matches.
* The tokens are the literals and productions beginning with `_`.
* `IF(ifWouldMatchThis, thenMatchThis, elseMatchThis)` - reduces complexity of BNF
* `R(x)` - Generates a production rule with x expanded in R
* `i * X` - repeat given number of times, where i is local to production rule
* `{constraint}` - specifies a constraint
* `\p{C}` and `\p{L}` are defined at https://www.regular-expressions.info/unicode.html#category
* `[...]`
* `\uXXXX` unicode

----

# Play

## Accommodate spaces in tag names

%foo: .attr1 32 .attr2 44:: Body text
%typefail.error: .code 2000 .regex "some error"
%foo.(prop name): "yowsa"
%foo: .(prop name) 32
%foo: .(prop name) [text of prop name]
%foo: (.prop name) 32

%foo/prop name: "yowsa yowsa"
%foo/prop name: [yowsa yowsa]
%typefail/error: /code 2000 /regex "some error"
%foo/prop name:: yowsa yowsa
%typefail/error: /code 2000 /regex "some error": yowsa yowsa

%foo/prop name = "yowsa yowsa"
%foo/prop name = [yowsa yowsa]
%typefail/error = /code 2000 /regex "some error"
%foo/prop name: yowsa yowsa
%typefail/error = /code 2000 /regex "some error": yowsa yowsa

%foo/prop name: [yowsa yowsa]
%typefail/error: /code 2000 /regex "some error"
%foo/prop name:= yowsa yowsa
%typefail/error: /code 2000 /regex "some error" = yowsa yowsa

%foo/prop name: [yowsa yowsa]
%typefail/error: /code 2000 /regex "some error"
%typefail/error:| error message
%typefail/error | error message
%typefail/error: /code 2000 /regex "some error" | yowsa yowsa
%typefail/error: /(error code) 2000 /(regular expression) "some error"
%(w/slashes)/(also w/ slashes)

%(=abcdef)
%typefail/error: (=abcdef) /code 2000 /regex [some error] | yowsa yowsa
%(#abcdef)
%typefail/error: (#abcdef) /code 2000 /regex [some error] | yowsa yowsa
%typefail/error: /(error code) 2000 /(regular expression) [some error]
%(prop name w/ slashes): [prop value]
%typefail/error: 1000 2000 3000 | text string
%typefail/error: {1000, 2000, 3000}
%param: argName [array[]] | Single-line multi-word description

%spider %Latrodectus variolus %GB/Phylogeny A1/Marpissa obtusa 

%spider %Latrodectus-variolus %GB/Phylogeny-A1/Marpissa-obtusa
    -- ppl wanna type, but must escape actual dashes (discouraging use of dashes in tags)

%spider %Latrodectus_variolus %GB/Phylogeny_A1/Marpissa_obtusa
    -- clearer spaces/dashes, but _ less widely known, encouraging use of dashes in tags

%spider %(Latrodectus variolus) %GB/(Phylogeny A1)/(Marpissa obtusa)

%red-green_color
%red--green-color
%red\-green-color
%company_name: Megatron Inc.
%company name: {Megatron Inc.}
%company name: "Megatron Inc."
%company_name: | Megatron Inc.

%company-name: Megatron Inc.
%company name: {Megatron Inc.}
%company name: "Megatron Inc."
%company name: | Megatron Inc.

%expect-error /code 3001 /regex {error message}
%expect_error /code 3001 /regex {error message}
%expect_error /code 3001 /regex "error message"

// colon indicates line syntax, otherwise embeddable syntax

%foo %bar %baz
%foo(123 list) %bar[text] %baz[%poo %bah embedded text]
%fuz[/zy /zie embedded text]

%foo: 123
%foo: 123 list
%foo: 123 (1 2 3)
%bar: [words of text]
%baz: [%poo %bah embedded text]
%fuz: [/zy /zie embedded text]
%fuz: /zy /zie[hello]
%todo: /priority /by[29 Feb 2019] [Do this thing.]
%todo: /priority [Do this thing.]
%todo: /priority[high] [Do this thing.]
%foo: %poo %bah [poo-bah thing]
%company-name: [Megatron Inc.]
%expect-error: /code(3001) /regex[error message]
%expect-error[%/code(3001) %/regex[error message]]
%expect-error(/code(3001) /regex[error message])
%foo(/a1 /a2 [av] /b1 /b2 [bv])
%param: identifier [type] [Description] /attrForNextListItem [Next list item.]

// tweet and post syntax

(The only reason to allow tags within text is for convenience. The tags only characterize the unit as a whole. A person might want to embed a tag in the text to keep from having to redundantly type text, as when the tag name would duplicate some of the text. It might also be easier to throw the tag on the end of a line or at the beginning to keep the text compact and easier to read as a whole. In this latter case, the tags are not part of the text. If tags are to sometimes be part of the text and sometimes not, the two modes will need a syntactic difference, at added syntactic complexity. Perhaps the tags should strictly be separate from the text to restrict tags to being external (sometimes requiring redundant typing) and prevent user confusion over whether they render in the text. Maybe one way to reduce redundancy is with app support for highlighting text to turn it into a tag. But if there's app support maybe there isn't much need for syntax at all; syntax may make the input more convenient and allow for embedding of metadata where there isn't app support.)

%Latrodectus-mactans() Here is my photo.
%foo 32: My tweet goes here. (X - Produces a list, not a tagged tweet.)
%foo(32) My tweet goes here.
%foo% %bar% %foo-bar(foo bar value)
%foo %bar %foo-bar My tweet goes here.
%foo; %bar; %foo-bar; My tweet goes here.
%foo! %bar! %foo-bar! My tweet goes here.
%foo %bar %foo-bar| My tweet goes here.
%foo %bar(Bar's value) %foo-bar My tweet goes here.
%expect-error(/code 3001 /regex {error message})
%(tag w/ slashes)(/code 3001 /regex {error message})
%[tag w/ slashes](/code 3001 /regex {error message})
%[expect error](/[error code] 3001 /[regular expression] [error message])
%expect-error(/error-code 3001 /regular-expression [error message])
%expect-error(/code 3001 /regex [error message])
%expect-error(%code 3001 %regex [error message])

%error (%/code 3001 %/regex [error message])
%error (/code 3001 /regex [error message])
%error(/code 3001 /regex [error message])
%error /code 3001 /regex [error message]

%error: Description
%error(: Description)
%param(identifier {type}: Description)

%error [Description]
%error([Description])
%param(identifier [type] [Description])
%param(identifier {type} {Description})
%param(, identifier, type, Description)
%param(; identifier; type; Description)
%param(identifier | type | Description)

%param(; options; [object]; Description of options.)

%param( options | [object] | Description of options. )

%param(options | [object] | Description of options.)

%param( options [object] | Description of options. )

%param(options [object] | Description of options.)

%error(/code 3001 /regex [error message])

%param options {object} Description of options.
%param {options [object] [Description of options.]}

%param options [object]: Description of options. -- weird

%param options [object] | Description of options. -- weird

%param options [object] [Description of options.] -- better

%param{options [object] [Description of options.]}
%param {options [object] [Description of options.]}

%param(options [object] [Description of options.])
%param (options [object] [Description of options.])

%todo /priority /by [12 Feb 2019]: Thing to do.

%todo [%/priority %/by [12 Feb 2019]

Thing to do.]

%todo(| Thing to do.)
%todo(: Thing to do.)
%todo([Thing to do.])
%todo[ Thing to do. ]
%todo [Thing to do.] -- fine!
%todo {[Task A] [Task B] [Task C]}

// Require tag values to be bracketed

%todo (/priority /by[12 Feb 2019] Thing to do.) -- should merge these syntaxes
%todo [%/priority %/by[12 Feb 2019] Thing to do.]
%todo [/priority: Thing to do.]
%tabs (1 2 3 4)
%tabs [/as-spaces: (1 2 3 4)] -- ugh, now have to escape an initial text '('; also colon ambiguity
 -- but %(maybe w/ slashes) is used for bracketing tag names
%tabs /as-spaces: {1 2 3 4}

%todo ([Task A] [Task B] [Task C]) -- parameter list (to curly braces!)
%todo [/priority /by[12 Feb 2019] Thing to do.] -- markup w/ optional value

// more like code and @param -- better complements markdown (spaces as dashes in tag names)

%tabs [ 4, 8, 12, 16 ]
%param identifier {type}: Single-line multi-word description
%error /code 3000 /regex "error message"
%error /code 3000 /regex {error message}
%bio {First paragraph.

Second paragraph.}
%bio {[initial markdown link](http://somewhere.com)} -- error prone
%bio: \[initial markdown link\](http://somewhere.com)


// more like markdown -- confusable with markdown (spaces as dashes in tag names)
//  but more likely to be typed, [] better known, more readable, no need for quoted values

%tabs { 4, 8, 12, 16 }
%param identifier [type]: Single-line multi-word description
%error /code 3000 /regex "error message" -- more likely to be typed, no tag escaping
%error /code 3000 /regex [error message]
%bio [First paragraph.

Second paragraph.]
%bio: [[initial markdown link](http://somewhere.com)] -- error prone
%bio: \[initial markdown link\](http://somewhere.com)


// more like code and @param -- better complements markdown (spaces in tag name syntax)

%tabs: [ 4, 8, 12, 16 ]
%param: identifier {type} | Single-line multi-word description
%error: /code 3000 /regex {error message}
%bio: {First paragraph.

Second paragraph.}
%bio: {[initial markdown link](http://somewhere.com)}
%bio: | \[initial markdown link\](http://somewhere.com)


// more like markdown -- confusable with markdown (spaces in tag name syntax)

%tabs: { 4, 8, 12, 16 }
%param: identifier [type] | Single-line multi-word description
%error: /code 3000 /regex [error message]
%bio: [First paragraph.

Second paragraph.]
%bio: [[initial markdown link](http://somewhere.com)]
%bio | \[initial markdown link\](http://somewhere.com)


%typefail/error: /code 2000 /regex {some error} | yowsa yowsa
%typefail/error: /(error code) 2000 /(regular expression) {some error}
%(prop name w/ slashes): {prop value}
%typefail/error: 1000 2000 3000 | text string
%typefail/error: [1000, 2000, 3000]
%param: argName {array[]} | Single-line description

%(prop name) 32
%(typefail.error) .code 2000 .regex "some error"
%(foo.prop name)
%foo (.prop name) {text value of prop name}
%foo (.prop name) [text value of prop name]
%(prop name) {text value of prop name}
%(prop name) [text value of prop name]
%(prop name)[text value of prop name]

%[prop name] {text value of prop name}
%[prop name] [text value of prop name]
%[prop name] (text value of prop name)
%[prop name] .[attr 1] (a1 value)
%[prop name] .[attr 1] 1234
%[prop name] .[attr 1] (*ref) -- should still be a reference

%"prop name" ."attr 1" 1234

%namespace."prop name" ."attr 1" [text value]

%namespace.[prop name] .[attr 1] (text value)





## Prior Play

```
%foo attr1 #symbol attr2 "string"
%foo attr1 32 attr2 45 = Symbol
%foo attr1 "hello" attr2 "there" = 1234
%foo attr1 32 attr2 45: text
%foo attr1 = 1234 // but 1234 is the value of foo, not attr1!
$foo = Symbol
%foo = 1234
%foo: text
%foo: 1234 text
%foo "text"
%foo %attr1 32 %attr2 45: text
%image %file "drive:/somewhere/here" %caption "Say this below."
%image {%file "drive:/somewhere/here" %caption "Say this below."}

%foo: "text"
%foo: `text`
%foo: {text}
%foo: text
%foo: #symbol

%param [argName {type} "description"] // hash not needed in a list
%param [#argName {type} "description"]
%param [#argName {type} {description}]
%param [#argName {type}: description]
%param [argName {type}: description] // should no-hash default to string? (Yes, for consistency)
// ^ or perhaps above should be invalid for now

%param #argName {type} "description"
%param #argName {type} {description}
%param #argName {type}: description // only allow this unbracketed?

{{foo}} // how does this translate? -- inner brackets not in value position, therefore text

%foo 1 2 3
%foo [1 2 3]
%foo [[1 2 3]] - can't have a list of one element, so is this a list of 3 or pure text?
%foo {[1 2 3]}
%foo {{abc}}

%error code 2000 regex "invalid parameter"
%error .code 2000 .regex "invalid parameter"
%error code=2000 regex="invalid parameter"

%error %code 2000 %regex "invalid parameter"
%error %code=2000 %regex="invalid parameter"
%error {%code 2000 %regex "invalid parameter"}
%error {code 2000 regex "invalid parameter"}
%error 2000 {%regex "invalid parameter"}
%error `  exact match text `


%error code=2000 regex="invalid parameter" -- problem: precludes tags defaulting to empty element
%param 'argName' {type}: description -- problem: more work than should be required for a term/symbol
%param #argName {type}: description -- problem: ambiguous when to use/require symbols/strings FAV
%error code 2000 regex "invalid parameter" -- problem: requires diff syntax to express a list FAV
%error {code 2000 regex "invalid parameter"} -- problem: no longer have bracketed content FAV
 // when do I need bracketed content? when both tags and bracketed content? A: pretty nested structs
%param [argName {type} "description"] -- problem: shouldn't have to quote lengthy descriptions
%param [argName {type}: description] -- problem: this is unintuitive, awkward, but necessary
%error %code 2000 %regex "invalid parameter" -- problem: too hard to read; better as sibling props
%error(code 2000 regex "invalid parameter") -- problem: looks like a func call listing param values
%error {%code 2000 %regex "invalid parameter"} -- problem: too verbose and hard to read
%error {.code 2000 .regex "invalid parameter"} -- (implicit use of containing namespace)
%typefail.error {.code 2000 .regex "invalid parameter"} -- (implicit NS) problem: only delims text
%typefail.error .code 2000 .regex "invalid parameter" -- (implicit NS) problem: what would this 
 // mean were it using full tag names with '%'?

%error .code 2000 .regex "invalid parameter" -- these tags belong to the generic namespace
%typefail.error .code 2000 .regex "invalid parameter" -- these tags belong to the generic namespace
%param argName {type}: Description

%param [argName {type}]: Description -- problem: not clear that description is part of list



%validate #false
%validate: #false

%foo attr1 {%bar: text}
%foo attr1 {%bar: #symbol}
%foo attr1 {%bar: "#string"} // are the quotes part of the string? Yes, defaults to string
%foo attr1 {%bar: \#string}
%foo attr1 {%bar: {#string}}

%foo attr1 {%bar: text if not syntactically something else}
%foo attr1 {%bar: {always text no matter what}}


{} = [] = '' = "" = empty element
```

Questions:

* It would be nice to allow users to enter symbol names without the # in places where the # is not required to disambiguate; but doing so creates inconsistencies in the syntax that might cause the user to accidentally supply a string elsewhere when they meant a symbol. For example, in lists.
* A tag can take an unbracketed values list only if tag property names are syntactically different from all values in the list. In this case, it seems best to require both the bracketed and unbracketed list syntaxes to require the same member syntax. If so, symbol names and string must be disintguishable from tag names in both cases.
* Should we allow implicit, unbracketed lists?