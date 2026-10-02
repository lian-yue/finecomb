# A.31 Configuration and serialization languages

Applies to: JSON, YAML, TOML, INI and similar configuration inputs, including embedded documents. Select the actual format, schema, parser and options; file extensions alone do not establish semantics. Loading and precedence remain in [32](dimensions/32-configuration-and-defaults.md).

| Checkpoint | What counts as a problem |
| --- | --- |
| Duplicate and equivalent keys | Validation and use disagree on duplicate keys or keys that collide after case folding, coercion, dotted-key expansion or normalization |
| Scalar resolution | A YAML schema or parser revision changes a string into a boolean, number, null or timestamp, altering the meaning of a configuration field |
| Document and token boundaries | The loader accepts only the first document or value while another component consumes trailing content; malformed input is accepted as a partial configuration |
| YAML aliases and tags | Aliases create shared mutable objects or excessive expansion; application tags invoke constructors or resolve resources outside the intended schema |
| Numeric and text round trips | Large JSON numbers lose precision; a writer changes signed zero, non-finite values, escapes or Unicode data that an identity or signature depends on |
| TOML tables and keys | Dotted keys, inline tables and table arrays are interpreted with different nesting or redefinition rules by the validator and consumer |
| INI and implementation extensions | Interpolation, comment syntax, key case, repeated sections or bare values differ between the chosen parsers; one implementation's extension is assumed portable |
| Embedded expressions | A value crosses from configuration data into a template, shell, policy or build expression without applying that language's checks |

Semantic references: [JSON](https://www.rfc-editor.org/rfc/rfc8259), [YAML](https://yaml.org/spec/1.2.2/), [TOML](https://toml.io/en/v1.0.0). Decoder security also uses [4.3](specialties/4.3-decoding-untrusted-external-data.md); INI behavior must be established from the specific implementation.
