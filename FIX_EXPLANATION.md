# Fix for "Parcourir" Button Mouse Click Issue

## Problem Description

The "Parcourir" (Browse) button in the save location section was:
- ✅ Accessible via keyboard navigation (Tab + Space)
- ❌ Not clickable with mouse
- ❌ The entire "ENREGISTREMENT PHOTOS/VIDÉOS" section was not responding to mouse clicks

## Root Cause

The issue was caused by a **layout constraint problem**, not just event handling:

**Critical Issue:** The container for the save location section (`createMappingAndSaveSection`) was missing a bottom anchor constraint. Without this constraint connecting the `saveLocationBox.bottomAnchor` to the `container.bottomAnchor`, the container's intrinsic height was ambiguous. This caused the save location section to be positioned outside the properly laid out view hierarchy, making it unable to receive mouse events.

**Contributing Factors:**
1. The text field was selectable by default, which could intercept clicks
2. The button's enabled state was not explicitly set
3. Window mouse event handling needed explicit configuration

## Solution

Four targeted changes were made:

### 1. Container Bottom Anchor Constraint (Line 375) - **PRIMARY FIX**
```swift
saveLocationBox.bottomAnchor.constraint(equalTo: container.bottomAnchor)
```
**Why**: This constraint ensures the container properly encompasses the save location box, positioning it within the view hierarchy where it can receive mouse events.

### 2. Window Mouse Events (Line 116)
```swift
window.ignoresMouseEvents = false  // Explicitly allow mouse events
```
**Why**: Ensures the window processes mouse events properly in fullscreen mode.

### 3. Text Field Selection (Line 919)
```swift
pathField.isSelectable = false  // Prevent text selection to avoid consuming clicks
```
**Why**: Prevents the text field from intercepting mouse clicks meant for nearby controls.

### 4. Button Enabled State (Line 934)
```swift
selectButton.isEnabled = true  // Explicitly enable the button
```
**Why**: Ensures the button is ready to receive mouse clicks.

## Testing

To test the fix:

1. Launch the application
2. Navigate to the "💾 ENREGISTREMENT PHOTOS/VIDÉOS" section
3. **Test with mouse**: Click directly on the "Parcourir..." button - it should open the folder selection dialog
4. **Test with keyboard**: Press Tab until the button is focused (should show focus ring), then press Space - it should still work as before

## Technical Notes

- The primary fix was the layout constraint - without it, the entire section was not properly positioned in the view hierarchy
- The additional changes help ensure robust mouse event handling across different scenarios
- Keyboard navigation is preserved and unaffected
- No changes to Info.plist were needed - mouse/pointer permissions are not required for standard UI interactions in macOS applications

## Compatibility

- macOS 13.0+ (as specified in Package.swift)
- Swift 5.9+
- No breaking changes to existing functionality
