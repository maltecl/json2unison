Service for generating a bunch of [unison](https://www.unison-lang.org/) code from a json sample.

Hosted on [unison cloud](https://maltecl.unison-services.cloud/s/json2unison/).


Example
```json
{
  "kind": "doc",
  "content": [
    {
      "kind": "heading",
      "attrs": {
        "level": 1
      },
      "content": [
        {
          "kind": "text",
          "text": "headline"
        }
      ]
    },
    {
      "kind": "paragraph"
    },
    {
      "kind": "paragraph",
      "content": [
        {
          "kind": "text",
          "text": "Some extra text here, all padding though."
        }
      ]
    }
  ]
}
```
will generate
```elm 
unique type Type0  = {
  level: Nat
}

unique type Type1  = {
  kind: Text,
  text: Text
}

unique type Type2  = {
  attrs: Optional Type0,
  content: Optional [Type1],
  kind: Text
}

unique type Type3  = {
  kind: Text,
  content: [Type2]
}

Type0.decoder : '{Decoder} Type0
Type0.decoder = do
  level = object.at! "level" Decoder.nat
  Type0 level

Type1.decoder : '{Decoder} Type1
Type1.decoder = do
  kind = object.at! "kind" Decoder.text
  text = object.at! "text" Decoder.text
  Type1 kind text

Type2.decoder : '{Decoder} Type2
Type2.decoder = do
  attrs = Decoder.optional! (do object.at! "attrs" Type0.decoder)
  content = Decoder.optional! (do object.at! "content" (Decoder.array Type1.decoder))
  kind = object.at! "kind" Decoder.text
  Type2 attrs content kind

Type3.decoder : '{Decoder} Type3
Type3.decoder = do
  kind = object.at! "kind" Decoder.text
  content = object.at! "content" (Decoder.array Type2.decoder)
  Type3 kind content

Type0.encoder : Type0 -> Json
Type0.encoder = cases
  Type0 level -> Json.object [
    ("level", (Json.Number.Unparsed << Nat.toText) level)
  ]

Type1.encoder : Type1 -> Json
Type1.encoder = cases
  Type1 kind text -> Json.object [
    ("kind", Json.text kind),
    ("text", Json.text text)
  ]

Type2.encoder : Type2 -> Json
Type2.encoder = cases
  Type2 attrs content kind -> Json.object [
    ("attrs", (Optional.getOrElse Json.null << Optional.map (Type0.encoder)) attrs),
    ("content", (Optional.getOrElse Json.null << Optional.map ((Json.array << List.map (Type1.encoder)))) content),
    ("kind", Json.text kind)
  ]

Type3.encoder : Type3 -> Json
Type3.encoder = cases
  Type3 kind content -> Json.object [
    ("kind", Json.text kind),
    ("content", (Json.array << List.map (Type2.encoder)) content)
  ]
```


