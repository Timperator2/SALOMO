# SALOMO

SALOMO is an annotation tool that reduces human workload in multi-label tasks by using a committee of LLMs to pre-select labels.

<img width="1920" height="918" alt="SALOMO screen" src="https://github.com/user-attachments/assets/82effa49-4997-4e42-aa62-d3b131e3d139" />

## Quick Start

- Download this repository, or just use `SALOMO.html`.
- Open `SALOMO.html` in a modern browser such as Chrome, Firefox, Edge, or Safari.
- Load a taxonomy-and-config JSON file.
- Load your dataset JSONL file in UTF-8.
- Resolve dissent labels, edit the union as needed, and save your decisions.
- Export the results as JSONL when you are done.

## Input Formats

### Dataset JSONL

Each line must be a JSON object with these required fields:

- `sentence`: string
- `union`: array of strings
- `dissent`: array of strings

Example:

```json
{"sentence":"This is an example sentence.","union":["Type 1"],"dissent":["Type 2","Type 4"]}
{"sentence":"This is another example sentence!","union":["Type 1","Type 2"],"dissent":[]}
{"sentence":"One more example sentence?","union":[],"dissent":["Type 1","Type 3"]}
```

### Taxonomy and Config JSON

The taxonomy-and-config file defines:

- taxonomy groups and labels
- optional sentence-level fields
- gamification settings
- task instructions
- translation defaults

If you do not want a slider or a free-text field, simply omit `annotationOptions.slider` or `annotationOptions.text` from the file.

Example:

```json
{
  "name": "Taxonomy and Config Template",
  "version": "1.2",
  "description": "A template for loading taxonomy definitions and interface configuration into SALOMO.",
  "taskDescription": "Read the sentence carefully and decide which taxonomy labels belong in the final union. Resolve all dissent labels, use the optional sentence-level fields if they are enabled, and save only when your review is complete.",
  "translation": {
    "autoTranslate": true
  },
  "groupOrder": ["Group A", "Group B", "Group C"],
  "groups": [
    {"name": "Group A", "color": "#eab308"},
    {"name": "Group B", "color": "#3b82f6"},
    {"name": "Group C", "color": "#6b7280"}
  ],
  "types": {
    "Type 1": {"group": "Group A", "definition": "This is the definition of Type 1.", "example": "This is an example for Type 1."},
    "Type 2": {"group": "Group B", "definition": "This is the definition of Type 2.", "example": "This is an example for Type 2."},
    "Type 3": {"group": "Group B", "definition": "This is the definition of Type 3.", "example": "This is an example for Type 3."},
    "Type 4": {"group": "Group C", "definition": "This is the definition of Type 4.", "example": "This is an example for Type 4."}
  },
  "annotationOptions": {
    "slider": {
      "key": "numeric_example_value",
      "name": "Numeric example value",
      "required": false,
      "min": 0,
      "max": 1,
      "step": 0.01,
      "default": 0.5,
      "leftLabel": "left",
      "rightLabel": "right"
    },
    "text": {
      "key": "short_reasoning",
      "name": "Short reasoning note",
      "required": false,
      "description": "Add a short note about what influenced your annotation decision for this sentence.",
      "placeholder": "Write a brief note...",
      "maxChars": 180
    }
  },
  "gamification": {
    "timeBasedXp": {
      "min": 10,
      "max": 25,
      "baseSeconds": 25,
      "unionSeconds": 5,
      "dissentSeconds": 10
    },
    "fieldXp": {
      "slider": 8,
      "text": 15
    },
    "streakBonuses": [
      {"streak": 3, "multiplier": 1.1},
      {"streak": 5, "multiplier": 1.25},
      {"streak": 10, "multiplier": 1.5}
    ],
    "levels": [
      {"xp": 0, "title": "Beginner Annotator", "icon": "🌱", "desc": "Welcome! Start annotating sentences to level up."},
      {"xp": 60, "title": "Type Spotter", "icon": "👁️", "desc": "You are spotting patterns with confidence."}
    ]
  }
}
```

## Notes on Configuration

- `annotationOptions.slider` is shown only if `name`, `leftLabel`, `rightLabel`, `min`, and `max` are present and valid.
- `annotationOptions.text` is shown only if `name`, `description`, and `maxChars` are present and valid.
- Set `required: true` on either field to make it mandatory before a sentence can be saved.
- If `required` is omitted or `false`, the field is visible but optional.
- `taskDescription` is optional. If it is present, SALOMO shows a `Task` button.
- `translation.autoTranslate` is optional. If it is set to `true` or `false`, SALOMO uses that value as the default hover-translation state.
- Hover translation uses NLLB directly.
- The translation target selector includes the following NLLB-supported targets: English, Mandarin Chinese (Simplified), Hindi, Spanish, French, Arabic, Bengali, Portuguese, Russian, Indonesian, and German.
- If `gamification` is omitted, SALOMO falls back to built-in defaults.
- XP for the optional slider and free-text field is configured in `gamification.fieldXp`, not in `annotationOptions`.
- Slider XP is granted only if the slider was actively used.
- Text XP is granted only if the free-text field contains content.

## How to Annotate

### Task Information

- Click `Task` to open the task description provided by the taxonomy-and-config file.

### Dissent Resolution

- For each dissent label, choose whether to keep it in the union or remove it.
- Use `Keep ALL` or `Remove ALL` for bulk actions.

### Union Editing

- Click `Modify union` to add or remove labels directly.
- Start typing to get autocomplete suggestions from the loaded taxonomy.
- You can add labels that are not present in the taxonomy. They will still be colorized with fallback colors and exported.

### Navigation

- `Prev`, `Next`, `Jump`, and `Next unresolved` help you move quickly.
- Use the search panel to search sentences by keyword.
- Use the filter to show only items whose union contains a selected type.
- Hover over a type to view its definition and example from the taxonomy.

### Translation

- SALOMO includes an optional hover-translation mode for annotators who may not speak every sentence language fluently.
- Translation uses Meta's NLLB model through the available Hugging Face Spaces endpoints.
- If you do not want translation enabled by default, set `translation.autoTranslate` to `false` in the taxonomy-and-config file.

### Gamification

- SALOMO includes an optional gamification system to make annotation more engaging.
- Users earn XP while annotating and can progress through configured levels.
- Time spent on a sentence contributes XP up to a configurable maximum based on the number of union and dissent labels.
- Extra XP for slider usage and free-text input is configured in `gamification.fieldXp`.

## Export Format

The exported JSONL preserves the original inputs and adds annotation metadata:

- `sentence`: string
- `union`: array of strings
- `dissent`: array of strings
- `union_original`: array of strings
- `resolution`: object mapping each dissent label to a boolean
- `resolved`: boolean
- `resolved_by`: string, if set
- `resolved_at`: ISO timestamp, if saved
- `added_nonoriginal`: labels added to the final union that were not in `union_original`
- `removed_nondissented`: labels removed from the final union that were in `union_original` and not part of `dissent`
- `numeric value`: number for the optional slider field
- `free text`: string for the optional free-text field

These fields make it possible to audit where the final annotation diverged from the original model pre-selection.

## Keyboard and Tips

- Press `Enter` in the add-type field to add the current suggestion.
- Press `Shift+Delete` to remove the current sentence from the dataset after confirmation.

## Citation

If you use SALOMO in research, please cite our LREC paper once the citation information is available.
