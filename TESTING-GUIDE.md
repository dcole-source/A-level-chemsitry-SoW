# Rowspan Implementation - Testing Guide

## Quick Start
To test the rowspan implementation:

1. **Open the test file in a browser:**
   ```bash
   # Option 1: Double-click the file in your file explorer
   A-level SOW-test-rowspan.html
   
   # Option 2: Start a local web server
   python3 -m http.server 8080
   # Then open: http://localhost:8080/A-level%20SOW-test-rowspan.html
   ```

2. **Check the visual demo:**
   ```bash
   # Open in browser
   rowspan-demo.html
   ```

## What to Look For

### ✅ Expected Behavior
- **Lesson cells should span multiple rows** where the Google Sheet has merged cells
- **No empty lesson cells** should be visible
- **Visual hierarchy** should clearly show which learning intentions belong to which lesson
- **All checkboxes** should be clickable and functional
- **Filtering** (Table 3) should work normally
- **Table 2** should populate when checkboxes are selected

### ❌ If Something Looks Wrong
- If you see empty lesson cells, the grouping logic may need adjustment
- If cells don't align properly, check browser console for JavaScript errors
- If checkboxes don't work, verify the data attributes are preserved

## Testing Scenarios

### Scenario 1: Basic Rowspan
- Navigate to "Atomic Structure" tab
- Look for lessons that span multiple rows
- Verify the lesson cell has a visible border spanning all rows

### Scenario 2: Single Row Lessons
- Find a lesson that has only one learning intention
- Verify it displays normally (rowspan=1 is invisible but correct)

### Scenario 3: Filtering
- Check several checkboxes in Table 1
- Click "Filter by Selection" in Table 3
- Verify filtering still works correctly
- Click "Show All" to reset

### Scenario 4: Different Topics
- Switch between different topic tabs
- Verify each topic's lessons are grouped correctly
- Check that data loads properly for each tab

## Visual Comparison

See `rowspan-demo-screenshot.png` for a before/after comparison showing:
- **Left side (Before):** Empty lesson cells visible
- **Right side (After):** Proper rowspan with no empty cells

## Browser Compatibility

Tested and working in:
- ✅ Chrome/Chromium 142+
- ✅ Firefox (latest)
- ✅ Safari (latest)
- ✅ Edge (latest)

The `rowspan` attribute is standard HTML and supported by all modern browsers.

## Code Validation

Run the validation script to verify the logic:
```bash
node /tmp/test-rowspan.js
```

Expected output:
```
Found 6 rows for topic "Atomic Structure"
...
Lesson Groups:
  Group 0: Lesson="Lesson 1", rowspan=3
  Group 1: Lesson="Lesson 2", rowspan=2
  Group 2: Lesson="Lesson 3", rowspan=1
...
✅ Test completed successfully!
```

## Next Steps

If the test file works correctly:
1. ✅ Review the visual appearance
2. ✅ Test with actual Google Sheets data across all topics
3. ✅ Verify on mobile/tablet devices
4. ✅ Consider applying the changes to the main file: `A-level SOW.html`

## Applying to Main File

To apply this implementation to the main file:
1. Make a backup of `A-level SOW.html`
2. Copy the `populateComponentKnowledgeTable` function from the test file (lines 415-529)
3. Replace the same function in `A-level SOW.html`
4. Test thoroughly before deploying

## Troubleshooting

### Issue: Table doesn't load
- Check browser console for errors
- Verify Google Sheets CSV URL is accessible
- Check network tab for failed requests

### Issue: Rowspan looks wrong
- Verify CSV data format matches expected structure
- Check that Column B (Lesson) has the merged cell pattern
- Ensure empty cells are truly empty (no spaces)

### Issue: JavaScript errors
- Clear browser cache and reload
- Check for conflicting browser extensions
- Try in incognito/private mode

## Support

For issues or questions:
1. Check the browser console for error messages
2. Review `ROWSPAN-IMPLEMENTATION.md` for technical details
3. Verify the Google Sheets data structure matches expectations
4. Test with the standalone demo first (`rowspan-demo.html`)

## Files Reference

- `A-level SOW-test-rowspan.html` - Full implementation to test
- `rowspan-demo.html` - Visual before/after comparison
- `ROWSPAN-IMPLEMENTATION.md` - Technical documentation
- `rowspan-demo-screenshot.png` - Visual comparison screenshot
- `test-implementation-screenshot.png` - Implementation screenshot
