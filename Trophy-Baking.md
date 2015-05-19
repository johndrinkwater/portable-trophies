# Trophy Baking

## What Is Trophy Baking?

In OpenTrophies we structures called [assertions](/Assertion.md), which are pieces of data that can be used to prove whether a not a person who says they got a trophy earned it (in the technical sense that it was issued to them).

Trophy Baking is the process of taking those assertions and embedding them into the image, so that when a user displays a trophy on a page, software that is OpenTrophies-aware can automatically extract that assertion data and perform the checks necessary to see if a person legitimately earned the trophy.

## Technical Details

### PNGs

#### Baking

An <a href="http://www.w3.org/TR/PNG/#11iTXt">`iTXt` chunk</a> should be inserted into the PNG with **keyword** `opentrophies`. The **text** can either be a signed trophy assertion or the raw JSON for the OpenTrophies assertion. Compression **MUST NOT** be used. At the moment, *language tag* and *translated keyword* have no semantics related to trophy baking.

An example of creating a chunk (assuming an iTXt constructor):

```js
var chunk = new iTXt({
  keyword: 'opentrophies',
  compression: 0,
  compressionMethod: 0,
  languageTag: '',
  translatedKeyword: '',
  text: signature || JSON.stringify(assertion)
})
```

An iTXt chunk with the keyword “opentrophies” **MUST NOT** appear in a PNG more than once. When baking a trophy that already contains OpenTrophies data, the implementor may choose whether to pass the user an error or overwrite the existing chunk.

#### Extracting

Parse the PNG datastream until the first <a href="http://www.w3.org/TR/PNG/#11iTXt">`iTXt` chunk</a> is found with the keyword `opentrophies`. The rest of the stream can be safely discarded. The text portion of the iTXt will either be the JSON representation of an OpenTrophies assertion or a signature.

#### Legacy PNGs

The pre-specified behavior of trophy baking worked differently. Instead of baking the whole assertion or signature into an `iTXt:opentrophies` chunk, the URL pointing to the hosted assertion was baked into a `tEXt:opentrophies` chunk. In order to get the full assertion, an additional HTTP request must be made after extracting the URL from the `tEXt` chunk.

### SVGs

#### Baking
First, Add an `xmlns:opentrophies` attribute to the `<svg>` tag with the value "http://opentrophies.org". Directly after the `<svg>` tag, add an `<opentrophies:assertion>` tag with a `verify` attribute. The value of `verify` should either be a signed OpenTrophies assertion **or** the URL from `verify.url` in the trophy assertion.

If a signature is being baked, no tag body is necessary and the tag should be self closing.

If an assertion is being baked, the JSON representation of the assertion should go into the body of the tag, wrapped in `<![CDATA[...]]>`.

An example of a well baked SVG with a hosted assertion:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<svg xmlns="http://www.w3.org/2000/svg"
     xmlns:opentrophies="http://opentrophies.org"
     viewBox="0 0 512 512">
  <opentrophies:assertion verify="https://example.org/assertion.json">
    <![CDATA[
       {
         "uid": "abcdef12345",
         "identity": {
           "recipient": "sha256$cbb08ce07dd7345341b9358abb810d29eb790fed",
           "type": "email",
           "hashed": true
         }
         "verify": {
           "type": "hosted",
           "url":"https://example.org/assertion.json"
         }
         "issuedOn": "2013-11-05",
         "trophy": "https://example.org/trophy.json"
       }
    ]]>
  </opentrophies:assertion>

  <...rest of document...>
</svg>
```

There **MUST** be only one `<opentrophies:assertion>` tag in an SVG. When baking a trophy that already contains OpenTrophies data, the implementor may choose whether to pass the user an error or overwrite the existing tag.

#### Extracting

Parse the SVG until you reach the first `<opentrophies:assertion>` tag. The rest of the SVG data can safely be discarded.

If the tag has no body, the `verify` attribute will contain the signature of the trophy. If there is a body, it will be the JSON representation of a trophy assertion.
