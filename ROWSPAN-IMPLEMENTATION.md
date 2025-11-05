# Rowspan Implementation for Merged Cells

## Overview
This document explains the implementation of HTML rowspan for merged cells in Table 1's Lesson column.

## Problem
The webpage displays Table 1 which pulls data from a Google Sheet. Column B (Lesson) in the Google Sheet has some merged cells. When exported as CSV, merged cells appear as one cell with a value followed by empty cells. The original implementation showed empty cells where merged cells existed.

## Solution
The `populateComponentKnowledgeTable` function has been modified to detect merged cells and apply appropriate rowspan attributes.

## Implementation Details

### Three-Pass Approach

#### Pass 1: Data Collection
- Iterate through all sheet data
- Filter rows matching the current topic
- Collect relevant data (lesson, learning intention, prerequisite, component knowledge)
- Store raw lesson values (before HTML escaping) to accurately detect empty cells

#### Pass 2: Grouping
- Iterate through collected rows
- When a non-empty lesson value is encountered: Start a new lesson group
- When an empty lesson value is encountered: Add to the current lesson group (indicates merged cell continuation)
- Store each group with its lesson name and array of rows

#### Pass 3: Rendering
- For each lesson group:
  - Calculate rowspan as the number of rows in the group
  - For the **first row** of the group:
    - Render lesson cell with `rowspan="N"` attribute
    - Render learning intention, component knowledge, and checkbox
  - For **subsequent rows** in the group:
    - Skip the lesson cell entirely (it's covered by the rowspan)
    - Render only learning intention, component knowledge, and checkbox

## Example

### Input Data (CSV format)
```
Topic,Lesson,Learning Intention,Prereq,Extra,Component Knowledge
Atomic Structure,Lesson 1,LI 1.1,Prereq 1,,CK 1.1
Atomic Structure,,LI 1.2,Prereq 2,,CK 1.2
Atomic Structure,,LI 1.3,Prereq 3,,CK 1.3
Atomic Structure,Lesson 2,LI 2.1,Prereq 4,,CK 2.1
Atomic Structure,,LI 2.2,Prereq 5,,CK 2.2
```

### Output HTML (simplified)
```html
<tr>
  <td rowspan="3">Lesson 1</td>
  <td>LI 1.1</td>
  <td>CK 1.1</td>
  <td><input type="checkbox"></td>
</tr>
<tr>
  <!-- lesson cell skipped (covered by rowspan above) -->
  <td>LI 1.2</td>
  <td>CK 1.2</td>
  <td><input type="checkbox"></td>
</tr>
<tr>
  <!-- lesson cell skipped (covered by rowspan above) -->
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
  <!-- lesson cell skipped (covered by rowspan above) -->
  <td>LI 2.2</td>
  <td>CK 2.2</td>
  <td><input type="checkbox"></td>
</tr>
```

## Test Files

### Test Implementation File
- **File**: `A-level SOW-test-rowspan.html`
- **Description**: Complete copy of the main HTML file with rowspan implementation
- **Usage**: Open in a web browser to see the rowspan functionality in action

### Standalone Test File
- **File**: `/tmp/test-rowspan-logic.html`
- **Description**: Minimal HTML file with simulated data for testing
- **Usage**: Open in a web browser to see a simple demonstration

### Node.js Test Script
- **File**: `/tmp/test-rowspan.js`
- **Description**: Command-line test that validates the logic
- **Usage**: Run `node test-rowspan.js` to see console output showing the grouping logic

## Edge Cases Handled

1. **No merged cells**: Each lesson has only one row → rowspan=1 for each
2. **Multiple merged rows**: Lesson spans multiple rows → rowspan=N
3. **Empty lesson at start**: Creates a group with empty lesson name (handled gracefully)
4. **Mixed merged and non-merged**: Correctly groups only consecutive empty cells

## Testing

To test the implementation:

1. **Open the test file in a browser**:
   ```bash
   # Start a local web server
   python3 -m http.server 8080
   # Navigate to: http://localhost:8080/A-level%20SOW-test-rowspan.html
   ```

2. **Verify the visual output**:
   - Lesson cells should span multiple rows where appropriate
   - No empty lesson cells should be visible
   - Table should maintain proper alignment
   - All checkboxes should work correctly

3. **Run the Node.js test**:
   ```bash
   node /tmp/test-rowspan.js
   ```
   Expected output shows proper grouping with rowspans: 3, 2, 1

## Code Changes

The main change is in the `populateComponentKnowledgeTable` function (lines 414-467 in the original file). The new implementation:
- Adds ~80 lines of code
- Maintains backward compatibility (no API changes)
- Preserves all existing functionality (checkboxes, data attributes, filtering)
- Improves visual presentation by properly displaying merged cells

## Next Steps

After validation:
1. Test with actual Google Sheets data
2. Verify across different topics
3. Check responsive behavior on mobile devices
4. If successful, apply changes to the main `A-level SOW.html` file
