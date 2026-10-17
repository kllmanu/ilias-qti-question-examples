# QTI 1.2.1 for ILIAS 11

This guide explains how to import QTI questions into an ILIAS 11 question pool. The example file [beispielfragen-qti12.xml](beispielfragen-qti12.xml) was tested with ILIAS 11.4.

## File format

- ILIAS can import this QTI XML file directly. This example does not need a ZIP file or an IMS manifest.
- The root element is `questestinterop`. Each question is one `item`.
- Every `item` ID must be unique. ILIAS reads the source NIC from IDs in this form: `il_<NIC>_qst_<number>`. If this pattern is missing, importers that process RTE or media content may fail because the NIC is missing.
- In the question pool, choose **Import**, upload the XML file, select the questions in the preview, and confirm.

## ILIAS metadata

ILIAS-specific values go inside `itemmetadata/qtimetadata/qtimetadatafield`. ILIAS reads `fieldlabel` and `fieldentry` with a SAX parser. Keep these elements together, with no formatting whitespace between them:

```xml
<qtimetadatafield><fieldlabel>QUESTIONTYPE</fieldlabel><fieldentry>assSingleChoice</fieldentry></qtimetadatafield>
```

Do not add a newline or indentation between `</fieldlabel>` and `<fieldentry>`, or between `</fieldentry>` and `</qtimetadatafield>`. The parser can treat this whitespace as part of the metadata value. Then it may not recognize `QUESTIONTYPE` or fields such as `answer_type`. Do not let an XML formatter split these elements.

Important fields:

- `ILIAS_VERSION`: the ILIAS version, for example `11.4 2026-09-03`.
- `QUESTIONTYPE`: the exact ILIAS question type from the table below.
- `AUTHOR`: set this field if the imported user should not appear as the author.
- Other fields depend on the question type. For example, Kprim uses `answer_type`, and file-upload questions use `points`.

## Question type IDs

The first column shows the labels used in the German ILIAS interface.

| ILIAS label                                       | `QUESTIONTYPE`          |
| ------------------------------------------------- | ----------------------- |
| Single Choice                                     | `assSingleChoice`       |
| Multiple Choice                                   | `assMultipleChoice`     |
| Kprim Choice                                      | `assKprimChoice`        |
| Fehler/Worte markieren (mark wrong words)         | `assErrorText`          |
| Hotspot/Imagemap                                  | `assImagemapQuestion`   |
| Lueckentextfrage (cloze)                          | `assClozeTest`          |
| Numerische Antwort (numeric answer)               | `assNumeric`            |
| Formelfrage (formula question)                    | `assFormulaQuestion`    |
| Begriffe benennen (identify terms)                | `assTextSubset`         |
| Anordnungsfrage: vertikal (vertical ordering)     | `assOrderingQuestion`   |
| Anordnungsfrage: horizontal (horizontal ordering) | `assOrderingHorizontal` |
| Zuordnungsfrage (matching)                        | `assMatchingQuestion`   |
| Freitext eingeben (free text)                     | `assTextQuestion`       |
| Datei hochladen (file upload)                     | `assFileUpload`         |
| Long Menu                                         | `assLongMenu`           |

## Answers and scoring

- Choice questions use `response_lid`, `render_choice`, and unique answer IDs. Correct answers are scored in `resprocessing`.
- Kprim uses `response_lid` with `rcardinality="Multiple"`. Use numeric answer IDs and a `decvar` with `varname="SCORE"` and `maxvalue`. The metadata fields `answer_type` and `option_label_setting` are also required. Example values are `singleLine` and `right_wrong`.
- Matching questions use one `response_grp` with `ident="MQ"` and one shared `render_choice`. Term and definition IDs must be numbers. Definitions use `match_max="1"` and `match_group`. A `varsubset` pair is written as `termID,definitionID`.
- Cloze gaps must be separate elements between separate `material` elements inside `flow`. Do not put a response inside a `material` element. Use the same gap ID in `response_str` and its scoring rule, for example `gap_0`.
- Free-text questions need `resprocessing scoremodel="HumanRater"` and a `decvar` with a positive `maxvalue`. Without a positive maximum score, ILIAS marks the imported question as incomplete.
- Numeric questions use `response_num` with `numtype="Decimal"`. Use `vargte` and `varlte` to define the accepted range.
- ILIAS vertical ordering uses the response ID `OQT`. Scoring rules also need position values.
- Embed hotspot images as `matimage` with `embedded="base64"`. The hotspot coordinates must fit the image size.
- Formula and error-marking questions use ILIAS-specific PHP-serialized metadata. Do not reformat these values by hand. When possible, copy them from an ILIAS export.
- Long Menu questions need a placeholder such as `[Longmenu 1]` in the menu text, plus matching `gapTypes` metadata and answer IDs.

## Check before import

1. Check that the XML is well formed and that every `item` ID is unique.
2. Check that every item has a supported `QUESTIONTYPE` value and, if needed, an `AUTHOR` field.
3. Make sure metadata fields have no formatting whitespace between the label and value elements.
4. Check the ILIAS preview for all expected question types. After import, check the type, score, and **Die Frage ist unvollstaendig!** warning in the question list.

The question titles in this example use the prefixes `01` to `15`. This keeps them in the same order as the question type list in ILIAS.
