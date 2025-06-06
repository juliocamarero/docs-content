---
applies_to:
  stack: ga
  serverless: ga
mapped_pages:
  - https://www.elastic.co/guide/en/kibana/current/numeral.html
---

# Numeral formatting [numeral]

Numeral formatting in {{kib}} is done through a pattern-based syntax. These patterns express common number formats in a concise way, similar to date formatting. While these patterns are originally based on Numeral.js, they are now maintained by {{kib}}.

Numeral formatting patterns are used in multiple places in {{kib}}, including:

* [Advanced settings](kibana://reference/advanced-settings.md)
* [Data view formatters](find-and-organize/data-views.md#field-formatters-numeric)
* [**TSVB**](visualize/legacy-editors/tsvb.md)
* [**Canvas**](visualize/canvas.md)

The simplest pattern format is `0`, and the default {{kib}} pattern is `0,0.[000]`. The numeral pattern syntax expresses:

Number of decimal places
:   The `.` character turns on the option to show decimal places using a locale-specific decimal separator, most often `.` or `,`. To add trailing zeroes such as `5.00`, use a pattern like `0.00`. To have optional zeroes, use the `[]` characters.

Thousands separator
:   The thousands separator `,` turns on the option to group thousands using a locale-specific separator. The separator is most often `,` or `.`, and sometimes ` `.

Accounting notation
:   Putting parentheses around your format like `(0.00)` will use accounting notation to show negative numbers.

The display of these patterns is affected by the [advanced setting](kibana://reference/advanced-settings.md#kibana-general-settings) `format:number:defaultLocale`. The default locale is `en`, but some examples will specify that they are using an alternate locale.

Most basic examples:

| Input | Pattern | Locale | Output |
| --- | --- | --- | --- |
| 10000.23 | 0,0 | en (English) | 10,000 |
| 10000.23 | 0.0 | en (English) | 10000.2 |
| 10000.23 | 0,0.0 | fr (French) | 10 000,2 |
| 10000.23 | 0,0.000 | fr (French) | 10 000,230 |
| 10000.23 | 0,0[.]0 | en (English) | 10,000.2 |
| 10000.23 | 0.00[0] | en (English) | 10,000.23 |
| -10000.23 | (0) | en (English) | (10000) |


## Percentages [_percentages]

By adding the `%` symbol to any of the previous patterns, the value is multiplied by 100 and the `%` symbol is added in the place indicated.

The default percentage formatter in {{kib}} is `0,0.[000]%`, which shows up to three decimal places.

| Input | Pattern | Locale | Output |
| --- | --- | --- | --- |
| 0.43 | 0,0.[000]% | en (English) | 43.00% |
| 0.43 | 0,0.[000]% | fr (French) | 43,00% |
| 1 | 0% | en (English) | 100% |
| -0.43 | 0 % | en (English) | -43 % |


## Bytes and Bits [_bytes_and_bits]

The Bytes and Bits formatters will shorten the input by adding a suffix like `GB` or `TB`. Bytes and Bits formatters include the following suffixes:

`b`
:   Bytes with binary values and suffixes. 1024 = `1KB`

`bb`
:   Bytes with binary values and binary suffixes. 1024 = `1KiB`

`bd`
:   Bytes with decimal values and suffixes. 1000 = `1kB`

`bitb`
:   Bits with binary values and suffixes. 1024 = `1Kibit`

`bitd`
:   Bits with decimal values and suffixes. 1000 = `1kbit`

Suffixes are not localized with this formatter.

| Input | Pattern | Locale | Output |
| --- | --- | --- | --- |
| 2000 | 0.00b | en (English) | 1.95KB |
| 2000 | 0.00bb | en (English) | 1.95KiB |
| 2000 | 0.00bd | en (English) | 2.00kB |
| 3153654400000 | 0.00bd | en (English) | 3.15GB |
| 2000 | 0.00bitb | en (English) | 1.95Kibit |
| 2000 | 0.00bitd | en (English) | 2.00kbit |


## Currency [_currency]

Currency formatting is limited in {{kib}} due to the limitations of the pattern syntax. To enable currency formatting, use the symbol `$` in the pattern syntax. The number formatting locale will affect the result.

| Input | Pattern | Locale | Output |
| --- | --- | --- | --- |
| 1000.234 | $0,0.00 | en (English) | $1,000.23 |
| 1000.234 | $0,0.00 | fr (French) | €1 000,23 |
| 1000.234 | $0,0.00 | chs (Simplified Chinese) | ¥1,000.23 |


## Duration formatting [_duration_formatting]

Converts a value in seconds to display hours, minutes, and seconds.

| Input | Pattern | Output |
| --- | --- | --- |
| 25 | 00:00:00 | 0:00:25 |
| 25 | 00:00 | 0:00:25 |
| 238 | 00:00:00 | 0:03:58 |
| 63846 | 00:00:00 | 17:44:06 |
| -1 | 00:00:00 | -0:00:01 |


## Displaying abbreviated numbers [_displaying_abbreviated_numbers]

The `a` pattern will look for the shortest abbreviation for your number, and use a locale-specific display for it. The abbreviations `aK`, `aM`, `aB`, and `aT` can indicate that the number should be abbreviated to a specific order of magnitude.

| Input | Pattern | Locale | Output |
| --- | --- | --- | --- |
| 2000000000 | 0.00a | en (English) | 2.00b |
| 2000000000 | 0.00a | ja (Japanese) | 2.00十億 |
| -5444333222111 | 0,0 aK | en (English) | -5,444,333,222 k |
| -5444333222111 | 0,0 aM | en (English) | -5,444,333 m |
| -5444333222111 | 0,0 aB | en (English) | -5,444 b |
| -5444333222111 | 0,0 aT | en (English) | -5 t |


## Ordinal numbers [_ordinal_numbers]

The `o` pattern will display a locale-specific positional value like `1st` or `2nd`. This pattern has limited support for localization, especially in languages with multiple forms, such as German.

| Input | Pattern | Locale | Output |
| --- | --- | --- | --- |
| 3 | 0o | en (English) | 3rd |
| 34 | 0o | en (English) | 34th |
| 3 | 0o | es (Spanish) | 2er |
| 3 | 0o | ru (Russian) | 3. |


## Complete number pattern reference [_complete_number_pattern_reference]

These number formats, combined with the previously described patterns, produce the complete set of options for numeral formatting. The output here is all for the `en` locale.

| Input | Pattern | Output |
| --- | --- | --- |
| 10000 | 0,0.0000 | 10,000.0000 |
| 10000.23 | 0,0 | 10,000 |
| -10000 | 0,0.0 | -10,000.0 |
| 10000.1234 | 0.000 | 10000.123 |
| 10000 | 0[.]00 | 10000 |
| 10000.1 | 0[.]00 | 10000.10 |
| 10000.123 | 0[.]00 | 10000.12 |
| 10000.456 | 0[.]00 | 10000.46 |
| 10000.001 | 0[.]00 | 10000 |
| 10000.45 | 0[.]00[0] | 10000.45 |
| 10000.456 | 0[.]00[0] | 10000.456 |
| -10000 | (0,0.0000) | (10,000.0000) |
| -12300 | +0,0.0000 | -12,300.0000 |
| 1230 | +0,0 | +1,230 |
| 100.78 | 0 | 101 |
| 100.28 | 0 | 100 |
| 1.932 | 0.0 | 1.9 |
| 1.9687 | 0 | 2 |
| 1.9687 | 0.0 | 2.0 |
| -0.23 | .00 | -.23 |
| -0.23 | (.00) | (.23) |
| 0.23 | 0.00000 | 0.23000 |
| 0.67 | 0.0[0000] | 0.67 |
| 1.005 | 0.00 | 1.01 |
| 1e35 | 000 | 1e+35 |
| -1e35 | 000 | -1e+35 |
| 1e-27 | 000 | 1e-27 |
| -1e-27 | 000 | -1e-27 |
