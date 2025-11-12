# Rowspan Implementation Flow Diagram

## Data Flow Overview

```
Google Sheets (Merged Cells)
         ↓
    CSV Export
         ↓
  parseCSV Function
         ↓
allTable1SheetData Array
         ↓
populateComponentKnowledgeTable
         ↓
    [PASS 1: COLLECT]
         ↓
    topicRows Array
         ↓
    [PASS 2: GROUP]
         ↓
   lessonGroups Array
         ↓
    [PASS 3: RENDER]
         ↓
  HTML with Rowspan
```

## Detailed Algorithm Flow

### PASS 1: Data Collection
```
For each row in allTable1SheetData:
  ├─ Skip header row
  ├─ Check if row has enough columns (6+)
  ├─ Check if Topic matches current tab
  └─ If match:
      └─ Collect: {
          lesson: raw value (column B),
          learningIntention: escaped (column C),
          prereq: escaped (column D),
          componentKnowledge: escaped (column F)
        }

Result: topicRows = [row1, row2, row3, ...]
```

### PASS 2: Grouping Logic
```
Initialize: currentGroup = null
For each row in topicRows:
  ├─ If lesson is NOT empty:
  │   ├─ Save currentGroup to lessonGroups
  │   └─ Start new group:
  │       currentGroup = {
  │         lesson: escaped lesson value,
  │         rows: [current row]
  │       }
  └─ If lesson IS empty:
      └─ Add to current group:
          currentGroup.rows.push(current row)

Finally: Add last currentGroup to lessonGroups

Result: lessonGroups = [
  { lesson: "Lesson 1", rows: [row1, row2, row3] },
  { lesson: "Lesson 2", rows: [row4, row5] },
  { lesson: "Lesson 3", rows: [row6] }
]
```

### PASS 3: Rendering with Rowspan
```
For each group in lessonGroups:
  ├─ Calculate rowspan = group.rows.length
  └─ For each row in group.rows:
      ├─ Create <tr> element
      ├─ If first row (index 0):
      │   └─ HTML = <td rowspan="N">lesson</td>
      │            <td>LI</td>
      │            <td>CK</td>
      │            <td>checkbox</td>
      └─ If subsequent row (index > 0):
          └─ HTML = <!-- skip lesson cell -->
                   <td>LI</td>
                   <td>CK</td>
                   <td>checkbox</td>

Result: Properly formatted table with rowspan
```

## Example Data Transformation

### Input CSV Data
```
Topic              | Lesson    | LI      | Prereq | ... | CK
-------------------|-----------|---------|--------|-----|-------
Atomic Structure   | Lesson 1  | LI 1.1  | P1     | ... | CK 1.1
Atomic Structure   |           | LI 1.2  | P2     | ... | CK 1.2
Atomic Structure   |           | LI 1.3  | P3     | ... | CK 1.3
Atomic Structure   | Lesson 2  | LI 2.1  | P4     | ... | CK 2.1
Atomic Structure   |           | LI 2.2  | P5     | ... | CK 2.2
```

### After PASS 1 (topicRows)
```javascript
[
  { lesson: "Lesson 1", learningIntention: "LI 1.1", ... },
  { lesson: "",         learningIntention: "LI 1.2", ... },
  { lesson: "",         learningIntention: "LI 1.3", ... },
  { lesson: "Lesson 2", learningIntention: "LI 2.1", ... },
  { lesson: "",         learningIntention: "LI 2.2", ... }
]
```

### After PASS 2 (lessonGroups)
```javascript
[
  {
    lesson: "Lesson 1",
    rows: [
      { lesson: "Lesson 1", learningIntention: "LI 1.1", ... },
      { lesson: "",         learningIntention: "LI 1.2", ... },
      { lesson: "",         learningIntention: "LI 1.3", ... }
    ]
  },
  {
    lesson: "Lesson 2",
    rows: [
      { lesson: "Lesson 2", learningIntention: "LI 2.1", ... },
      { lesson: "",         learningIntention: "LI 2.2", ... }
    ]
  }
]
```

### After PASS 3 (HTML Output)
```html
<tr>
  <td rowspan="3">Lesson 1</td>
  <td>LI 1.1</td>
  <td>CK 1.1</td>
  <td><input type="checkbox"></td>
</tr>
<tr>
  <!-- lesson cell skipped (rowspan) -->
  <td>LI 1.2</td>
  <td>CK 1.2</td>
  <td><input type="checkbox"></td>
</tr>
<tr>
  <!-- lesson cell skipped (rowspan) -->
  <td>LI 1.3</td>
  <td>CK 1.3</td>
  <td><input type="checkbox"></td>
</tr>
<tr>
  <td rowspan="2">Lesson 2</td>
  <td>LI 2.1</td>
  <td>CK 2.1</td>
  <td><input type="checkbox"></td>
</tr>
<tr>
  <!-- lesson cell skipped (rowspan) -->
  <td>LI 2.2</td>
  <td>CK 2.2</td>
  <td><input type="checkbox"></td>
</tr>
```

## Edge Cases Handled

### 1. Empty Lesson at Start
```
Input: ["", "LI 1", ...], ["Lesson 2", "LI 2", ...]
Output: Group 1: {lesson: "", rows: [...]}, Group 2: {lesson: "Lesson 2", rows: [...]}
```

### 2. All Lessons Non-Empty (No Merging)
```
Input: ["Lesson 1", ...], ["Lesson 2", ...], ["Lesson 3", ...]
Output: Each gets rowspan=1 (normal single row)
```

### 3. Single Topic Row
```
Input: Only one row for topic
Output: One group with rowspan=1
```

### 4. Multiple Merged Groups
```
Input: Merged group, merged group, single row, merged group
Output: Correctly identifies and groups all patterns
```

## Performance Analysis

- **Time Complexity**: O(n) where n = number of rows in sheet
  - Pass 1: O(n) - iterate all rows once
  - Pass 2: O(m) - iterate topic rows once (m ≤ n)
  - Pass 3: O(m) - render topic rows once
  - Total: O(n + m + m) = O(n)

- **Space Complexity**: O(m) where m = number of rows for current topic
  - topicRows array: O(m)
  - lessonGroups array: O(m)
  - Total: O(m)

- **No Performance Impact**: Same complexity as original implementation

## Browser Rendering

The `rowspan` attribute is handled natively by browsers:
- ✅ Standard HTML5 attribute
- ✅ No JavaScript required for display
- ✅ Accessibility-friendly (screen readers understand rowspan)
- ✅ Works with table sorting/filtering libraries
- ✅ Responsive-friendly

## Summary

This implementation:
1. ✅ Detects merged cells via empty lesson values
2. ✅ Groups consecutive rows correctly
3. ✅ Applies rowspan efficiently
4. ✅ Maintains all original functionality
5. ✅ Has no performance impact
6. ✅ Is browser-compatible
7. ✅ Is accessible
8. ✅ Handles edge cases
