
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

Non-objectives:

* representing arbitrary data structures
* formatting text or marking up text with formatting
* not a syntax for comments; instead, a syntax for embedding metadata in comments and a syntax for the metadata structure

## Tagging Text

chunking ambient content

## Content Blocks

A 'content block' is an ordered series of one or more lines of text. 

Each line tag occupies an entire line of text and possibly subsequent lines but doesn't contribute to the text of the content block. They 


## Line Tags


The ambient content matches the `AmbientBlock` production rule and evaluates to either a text element or an empty element. 

```ebnf
AmbientBlock ::= AmbientLine? (_NL AmbientLine?)*;
AmbientLine ::= IF(LineTagIndicator, LineTag, AnyTextLine);
LineTagIndicator ::= ST* '%' InitialIdentifierChar;
AnyTextLine ::= (ContentChar | ST)+;

LineTag ::= ST* '%' TagSpec (ST+ (InlineValues | InlineTaggedElem))? ST*;
TagSpec ::= Identifier ('.' Identifier)? | '.' Identifier;
Identifier ::= InitialIdentifierChar TrailingIdentifierChar*;

InlineValues ::= InlineValue MoreValues? ColonEndLineValue?;
MoreValues ::= (ST+ InlineValue)+ | (ST* ',' STN* InlineValue)+;

InlineTaggedElem ::= InlineTag (ST+ InlineTag)* ColonEndLineValue?;
InlineTag ::= '.' Identifier InlineValue?;
InlineValue ::= NonTextValue | QuotedValue | Symbol;

ColonEndLineValue ::= ST* ':' ST* EndLineValue;
EndLineValue ::= IF(NonTextValueIndicator, NonTextValue, EndLineText);
NonTextValueIndicator ::= '{' | '[' | '*' InitialIdentifierChar |
    Number ST* (_NL | '}' | _EOS);
EndLineText ::= ContentChar+ (ST+ ContentChar+)*;

NonTextValue ::= BracketedValue | Array | Reference | Number;
BracketedValue ::= '{' ValueBlock? '}';
Array ::= '[' (InlineValue MoreValues?)? ']';
Symbol ::= Identifier ('.' Identifier)*;
Reference ::= '*' Symbol;
Number ::= Decimal | Hex;
Decimal ::= '-'? [0-9]+ ('.' [0-9]+)? ([Ee][-+]?[0-9]+)?;
Hex ::= '0x' [0-9A-Fa-f]+;

ValueBlock ::= ValueLine (_NL ValueLine?)* | (_NL ValueLine?)+;
ValueLine ::= IF(LineTagIndicator, LineTag, BalancedLine);
BalancedLine ::= (NestedBrackets NonBracketText?)+ |
    NonBracketText (NestedBrackets NonBracketText?)*;
NestedBrackets ::= '{' NestedContent? '}';
NonBracketText ::= (NonBracketChar BacktickEscape?)+ |
    BacktickEscape (NonBracketChar BacktickEscape?)*;
NonBracketChar ::= ContentChar - '{' - '}' - '\' - '`' | EscapedChar | ST;
NestedContent ::= BalancedLine (_NL ValueLine?)* | (_NL ValueLine?)+;

QuotedValue ::= SQuotedValue | DQuotedValue | BacktickEscape;
SQuotedValue ::= "'" SQuotedText? "'";
DQuotedValue ::= '"' DQuotedText? '"';
BacktickEscape ::= (n * '`') BacktickText(n)? '`' {n >= 1};

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
STN ::= S | Tab | _NL;
_NL ::= '\'? CR? LF;

Tab ::= \u0009;
CR ::= \u000D;
LF ::= \u000A;
S ::= \u0020;
_EOS ::= ? end of character stream ?;
```

## Modified EBNF

* Excludes commas. Spaces and operators delimit terms. Operator precedence, highest first: (`\`, `[]`), `()`, (`?`, `*`, `+`), `i * X`, ` ` (space), `-`, `|`, `::=`
* Intraline whitespace (ST, STN) that is not the last token of a production rule only matches when the first required non-whitespace token to follow within the production rule also matches.
* The tokens are the literals and productions beginning with `_`.
* `IF(ifWouldMatchThis, thenMatchThis, elseMatchThis)` - reduces complexity of BNF
* `R(x)` - Generates a production rule with x expanded in R
* `i * X` - repeat given number of times, where i is local to production rule
* `{constraint}` - specifies a constraint
* `\p{C}` and `\p{L}` are defined at https://www.regular-expressions.info/unicode.html#category
* `[...]`
* `\uXXXX` unicode

----

Play:

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