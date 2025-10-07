# Fix for "Parcourir" Button Mouse Click Issue

## Problem Description

The "Parcourir" (Browse) button in the save location section was:
- ✅ Accessible via keyboard navigation (Tab + Space)
- ❌ Not clickable with mouse

## Root Cause

The issue was caused by a combination of factors related to mouse event handling in macOS:

1. **Text Field Selection**: The `pathField` text field, even though marked as not editable, was still selectable by default. This meant it could intercept mouse clicks in its vicinity for text selection operations.

2. **Button State**: The button's enabled state was not explicitly set, which in some edge cases with complex window configurations (like fullscreen mode) could lead to ambiguous behavior.

3. **Window Mouse Event Handling**: With the complex focus management required for gamepad and keyboard control, the window's mouse event handling needed to be explicitly confirmed.

## Solution

Three targeted changes were made to ensure mouse events reach the button:

### 1. Window Mouse Events (Line 116)
```swift
window.ignoresMouseEvents = false  // Explicitly allow mouse events
```
**Why**: When a window enters fullscreen mode or has complex focus management (as required for gamepad support), explicitly ensuring mouse events are not ignored helps prevent edge cases where clicks might be blocked.

### 2. Text Field Selection (Line 919)
```swift
pathField.isSelectable = false  // Prevent text selection to avoid consuming clicks
```
**Why**: By default, NSTextField can be selectable even when not editable, which means it can capture mouse clicks for text selection. This can interfere with clicking nearby UI elements. Setting `isSelectable = false` ensures the text field doesn't intercept clicks.

### 3. Button Enabled State (Line 934)
```swift
selectButton.isEnabled = true  // Explicitly enable the button
```
**Why**: Explicitly setting the enabled state ensures the button is ready to receive mouse clicks, especially important in complex view hierarchies and focus management scenarios.

## Testing

To test the fix:

1. Launch the application
2. Navigate to the "💾 ENREGISTREMENT PHOTOS/VIDÉOS" section
3. **Test with mouse**: Click directly on the "Parcourir..." button - it should open the folder selection dialog
4. **Test with keyboard**: Press Tab until the button is focused (should show focus ring), then press Space - it should still work as before

## Technical Notes

- The changes are minimal and surgical - only affecting the specific components involved
- Keyboard navigation is preserved and unaffected
- The fix addresses the interaction between fullscreen mode, complex focus management, and standard AppKit controls
- No changes to Info.plist were needed - mouse/pointer permissions are not required for standard UI interactions in macOS applications

## Compatibility

- macOS 13.0+ (as specified in Package.swift)
- Swift 5.9+
- No breaking changes to existing functionality
