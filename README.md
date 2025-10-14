# SALOMO

An annotation tool that reduces human workload in multi-label tasks by using a committee of LLMs to pre-select labels.

# Quick Start

- Download this repository (or just the index.html file).

- Open index.html in a modern browser (Chrome, Firefox, Edge, Safari).

- Load a taxonomy JSON (file or URL).

- Load your dataset JSONL file (UTF-8).

- Resolve dissent labels, edit the union as needed, and Save.

- Export results (.jsonl) when done.

# Input Formats

## Dataset JSONL (one JSON object per line)

Required fields per JSON:

- sentence: string

- union: array of strings (labels suggested by any model)

- dissent: array of strings (labels suggested by only one model)

Example:
```json
{"sentence": "This an example sentence.", "union": ["Type 1"], "dissent": ["Type 2","Type 4"]}
{"sentence": "This another example sentence!", "union": ["Type 1","Type 2"], "dissent": []}
{"sentence": "One more example sentence?", "union": [], "dissent": ["Type 1", "Type 3"]}
```

## Taxonomy JSON

Defines groups (with colors) and types (with group, definition, example). Type colors are optional; group colors are optional (fallback colors are generated).

Example:
```json
{
  "name": "Taxonomy Template",
  "version": "1.0",
  "description": "A template for loading a taxonomy into SALOMO",
  "groupOrder": ["Group A","Group B","Group C"],
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
  }
}
```

# How to annotate

## Dissent resolution:

- For each dissent label, choose Keep in union or Remove.

- Use Keep ALL / Remove ALL for bulk decisions.
  
## Union editing:

- Click “Modify union” to add/remove labels directly.

- Start typing to get autocomplete suggestions from the taxonomy.

- You can add labels not present in the taxonomy; they will still be colorized (fallback) and exported.

## Navigation:

- Prev/Next, Jump, and Next unresolved help you move quickly.
  
- Search sentences by keyword (left panel).
  
- Filter items by a union type. Clear the filter anytime.

# Export format

Exported JSONL preserves inputs and adds metadata:

    sentence: string
    union: array of strings (final union after your decisions and edits)
    dissent: array of strings (as loaded)
    union_original: array of strings (as loaded or derived)
    resolution: object, dissent label -> boolean (true=kept, false=removed)
    resolved: boolean
    resolved_by: string (if set in the UI)
    resolved_at: ISO timestamp
    added_nonoriginal: labels added to final union that were not in union_original
    removed_nondissented: labels removed from final union that were in union_original and not part of dissent

These fields let you audit where you overrode the original model pre-selection (e.g., adding a new label not suggested by either model, or removing a label suggested by both).

# Keyboard and tips

- Enter in “Add a type” field: add the highlighted suggestion.

- Shift+Delete: remove current sentence from dataset (confirm prompt).

# Citation

If you use SALOMO in research, please cite our LREC paper (to be added)
