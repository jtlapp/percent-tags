
# modtags model

The `modtags` model defines the information that modtags captures, independently of the syntax used to represent the information.

## Summary

This is a metamodel.



## Distinctive Objectives

* Provide a data model for marking up text by tagging
* Allow text to represent structures of arbitrary complexity
* Provide a well-defined metamodel for code annotation

TBD

To support downstream applications in which the users are not programmers, and to minimize confusion learning or interpreting the syntax, the information model:

* Should not distinguish between a list containing a single item and the item itself.
* Should not distinguish among empty lists, the absence of a value, nulls, and undefined.
* Should not support confusingly similar lexical types, such as both numbers and strings that contain only numbers.

## Clarifying Non-Objectives

* Does not define text formatting
* Does not strive to be able to represent arbitrary ontologies
* Does not impose constraints on the syntax of the text
* Does not strive to represent arbitrary data serializations

## Content and Metadata

The model separately represents information as content and metadata. Content is the primary information of the structure. Metadata is information that further characterizes the content. Within this model, all data is fundamentally text.

For example, the text of a document would be its content and primary information, while attributes such as the title, author, and font to use for rendering would all be metadata. Moreover, the values of these attributes would be expressed as text.

It is possible for a structure to have no content, indicating that it has no primary information. When this structure has metadata, the metadata is all equally primary. A credit card application would be an example of a structure having metadata but no content.

It is also possible for a structure to have neither metadata nor content, indicating an absence of data. This structure explicitly asserts that no data is available.

Metadata may itself be structured, itself having either content or metadata or both. For example, a credit card application may have a metadata structure for an address, itself consisting of metadata for street, city, state/province, postal code, and country.

## Elements and Tags

In the `modtags` model, the structures that hold content and metadata are called *elements*, and *tags* are the means for attaching metadata to elements.

An 'element' is the fundamental unit of information. It only directly represents content. Elements may be composed of other elements, allowing for structured content. For example, a document may be an element consisting of passages, tables, lists, and callout boxes, each itself represented by an element.

A 'tag' is an association of an element with a name and a value. The name is a short, readable string, and the value is another element.

Tags are useful for assigning properties to an element. Consider a document having a property given by the tag name "company" or "admin contact". The value of the "company" tag would be an element providing the text of the company name, while the value of the "admin contact" tag would be an element that itself has property tags for each of name, email, and phone number.

Tags are also useful for assigning application-specific types to an element. Consider an element that represents a quote or one that represents a citation. In the former case the content of the element might be quoted text, and in the latter case it might be an element tagged with "book", "author", and "publication" tags. In both cases the tagged element provides the required content, so the type tag need only provide the type name. No content is required -- or appears -- in the tag value.

Together, elements and tags can organize textual content into a software-readable structure that is labelled for human consumption.

## Element Types

Elements vary in the structure of their content. Each has an inherent type that identifies its structure. Accordingly, any given element is a *text element*, a *block*, a *list*, or an *empty element*.

### Text Elements

A 'text element' is an element representing unstructured text. Its content is a 'text string', which is an ordered series of one or more Unicode characters.

The text element is the fundamental unit of information of the model. The model represents all primitive types as text, so character strings, numbers, booleans, symbols, dates, etc., are all represented as text elements.

Notice that a text string cannot be empty. In this model, all occurrences of the absence of data are equivalently represented as an empty element.

### Lists

A 'list' is an element representing a collection of independently-intelligible units of information. Its content is an ordered series of two or more elements, each of which is called a 'member' of the list.

Examples of lists include a checklist of to-do items, a series of log entries, a series of structured records, and the columns at which to place tab stops.

An application may generally assume that it can intelligibly present any member of a list independently from the list and from other members of the list. Even so, the order in which members appear in a list may also communicate information.

Notice that a list cannot be empty. In this model, an empty set is equivalent to the absense of data, which is represented as an empty element.

Also notice that a list cannot have just one element. This model does not represent collections independently of the elements composing the collections.

### Blocks

A 'block' is an element representing structured expository information. Its content is an ordered series of two or more elements, each of which is called a 'constituent' of the block. The information conveyed by each constituent after the first generally depends on preceding constituents of the block.

For example, whereas a text element represents text all in one format, a block could represent a concatenation of text in varying formats. In another example, a block could represent a series of visual elements that each builds on prior elements to communicate a point.

An application should normally present blocks as a whole and only isolate block constituents with reference to the context of the constituent within its block. 

Block constituents are subject to the following two constraints:

- Consecutive text element constituents cannot both be untagged.
- Block and empty element constituents must each have at least one tag.

Notice that blocks cannot be empty. This eliminates ambiguities that would exist among empty blocks, empty lists, and empty text strings, representing them all instead as empty elements.

Also notice that a block cannot have just one element. This eliminates ambiguities that would exist among single-element blocks, single-element lists, and isolated single elements.

Finally, notice that informationally-equivalent blocks are identically structured, ignoring synonymies that an application might impose. For example, two adjacent untagged text elements would be informationally equivalent to a single text element having a concatenation of their text strings. Similar isomorphies would exist for adjacent undifferentiated blocks and empty elements, but there are also contexts in which these latter elements aren't meaningful within a block, so blocks require that they always be tagged and hence differentiated.

### Empty Elements

An 'empty element' is an element representing the absence of primary information. It has no content. An untagged occurrence indicates the complete absense of data. A tagged occurrence indicates a structured record having only properties and types, in any combination.

Examples of empty elements include the value of a tag that only indicates type, a placeholder within a list, a reference to an external resource to be rendered, a specification for a configuration consisting only of properties, and an account application consisting of multiple fields represented as properties.

Nothing inherently distinguishes a tagged empty element from an element containing content. Whether data occurs within a tag value or within content is a characteristic of the schema of tags employed to structure the text.

## Tags

*Tags* are the means for semantically labeling data in the `tagtied` model. A *tag* is formally an association of an element with a *tag type* and a *tag value*. The tag type is said to 'tag' the element, and the element is said to be 'tagged'. *Tag schemas* define the available tag types, their valid values, and their interpretations.

### Tag Types

The 'tag type' indicates the meaning of the tag and uniquely distinguishes it from all other tags. Each tag type has a *tag name* and a *namespace*.

The 'tag name' is a name that is descriptive or suggestive of the tag type, at least within the context of the namespace. It is a character string conforming to the `TagName` production:

```ebnf
TagName ::= VisibleChar+ (S VisibleChar+)*
VisibleChar ::= UnicodeChar - ControlChar - S
ControlChar ::= [#x00-#x1F] | #x7f
S ::= #x20
```

The 'namespace' is a scope of tag names. It is structurally analogous to a Universal Resource Locator (URL), minus the URL protocol or scheme. Each namespace has a domain name and is identified relative to this domain name via a series of tag names. Specifically, any given namespace is either a *domain* or a tag type.

A 'domain' is an Internet domain corresponding to a domain name. It may include subdomains.

A namespace that is a tag type itself has a namespace, which is either a domain or another tag type. As a consequence, namespaces may be organized into a hierarchy of namespaces, with a domain name at the root.

Tag names within the same namespace that differ only in letter case refer to the same tag type.

### Tag Schemas

Given that a tag name and namespace together uniquely identify a tag type among all possible tag types, it is possible to define tag types independently of their applications. These definitions are *tag schemas*. A 'tag schema' is a specification for which tag types are available, what their tag values may be, and how applications should interpret the tag types and tag values.



The `modtags` model allows any tag type to both tag an element and serve as a namespace for other tag types. The application is responsible for specifying both the schema of available tag types and their valid uses.




Given that each non-root namespace is also a tag type, an application may interpret the tagging of an element with a tag type also as a tagging of the element with the tag type of its namespace, recursively. In this case, the tag value of each namespace is implicitly the empty element. Consequently, each tag type namespace effectively assigns a type to the tagged element rather than a property.

For example, consider a text element that is tagged with a tag type named "bold" belonging to a tag type namespace named "emphasis", which itself belongs to a tag type namespace named "formatting". An application may interpret that text element as having beed tagged with each of the types "bold", "emphasis", and "formatting". Were the "bold" tag to have a value indicating the strength of the boldness, such as values "light" or "strong", the implicit "emphasis" and "formatting" tags would not also take this value; they would instead each have an empty element value, indicating no value.


cyclicity, shared elements, structured tag values


Tag URLs label content and may serve as type names or property names. Tag values are elements, which allows tags to have arbitrary structure. In short, tags generically perform the functions of type assignment, property assignment, and structured composition.

## Notes

* Named like modulus, model, module (maybe rad too)

%

- owl eyes
- bird eyes
- pong
- cufflinks
- potoo
- bubbles
- berries
- berry tags
- boggle eyes
- revolving door
- merry go round
- turnstile (taken in NPM)
- lifesaver
- mints
- stork
- pinball
- paddle (paddle tags)
- cherry tags
- modmods/moddocs