---
title: Special Types
weight: 10
---

{{< toc >}}

## Children Var

Children Var is passed as `@children` to all components (available inside of a &lt;define-component&gt; block).
It contains all the *evaluated* children of the component.  You must pass a valid *ChildrenVar* object to the bind
expression of `<h-children>`.  Because most of the getters return ChildrenVar objects they can be chained.

Examples:
```
// access children by slot.  
// ".noslot" will find children not tagged with a slot
// ".byslot" will access children tagged with the given slot name
<h-children bind="@children.noslot"></h-children>
<h-children bind="@children.noslot.tags"></h-children>
<h-children bind="@children.byslot['head']"></h-children>

// only children which are 'tags' (filters out text/comments)
<h-children bind="@children.tags"></h-children>

// children by index, or first child
<h-children bind="@children.byindex[0]"></h-children>
<h-children bind="@children.first"></h-children>

// children by tag name
// for libraries use bycomp['[lib-name]:[comp-name]']
<h-children bind="@children.bytag['a']"></h-children>
<h-children bind="@children.bycomp['hibiki/bulma:option']"></h-children>

```

| Property | Description |
|----------|-------------|
| size | number of children |
| noslot | returns children not tagged with a slot name (includes text/whitespace/comment nodes) |
| tags | filters children to only include children which are tags (excludes text and whitespace) |
| first | returns first child |
| byindex | returns an array of childrenvar objects, each with one child (by index) |
| bytag | returns an object mapping tag names to childrenvars |
| bycomp | returns an object mapping '[lib-name]:[comp-name]' to children vars (useful for libraries to find their own components |
| node | returns a NodeVar object of the first child |
| nodes | returns an array of NodeVar objects |
| filter | advanced.  takes a lambda, calls it for each child (@node=NodeVar) and returns a new childrenvar object of the matching nodes |
| all | all the children (noop), deprecated (for historical purposes) |


## Hibiki Blob

Hibiki Blob can represent images, videos, text, etc.  They can be displayed by the appropriate tag (img, video, etc.) or downloaded by
the user.  They are stored as base64 dataurl values.

| Property | Description |
|----------|-------------|
| mimetype | the mimetype of the blob, e.g. 'text/csv', 'image/jpeg' |
| bloblen | size of blob (bytes) |
| name | name of blob - not all blobs have names.  usually a name comes from a file upload input and will contain the uploaded file name |
| base64 | base64 string representation of the blob |
| text | (v0.3.5) if the blob has mimetype of 'text/*' will return the text content of the blob |

