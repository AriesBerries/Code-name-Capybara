# LAYOUT_EDIT_MODE_COMPLETE_SPECIFICATION.md

**Version:** 1.0.0  
**Status:** APPROVED FOR IMPLEMENTATION  
**Last Updated:** 2025-01-02  
**Authors:** Platform Engineering  

---

## Table of Contents

1. [Executive Summary](#1-executive-summary)
2. [Edit Mode Toggle](#2-edit-mode-toggle)
3. [Rails System](#3-rails-system)
4. [Draggable Components Inventory](#4-draggable-components-inventory)
5. [Drag-and-Drop Behavior](#5-drag-and-drop-behavior)
6. [Keyboard Controls](#6-keyboard-controls)
7. [Control Bar](#7-control-bar)
8. [Layout Persistence](#8-layout-persistence)
9. [Import/Export](#9-importexport)
10. [Default Layouts](#10-default-layouts)
11. [Edge Cases & Error Handling](#11-edge-cases--error-handling)
12. [Implementation Notes](#12-implementation-notes)

---

## 1. Executive Summary

### 1.1 Feature Purpose

The Layout Edit Mode provides users with the ability to customize their composer interface through an intuitive drag-and-drop system. Users can reorder, hide, and restore UI controls within defined constraint zones (Rails) to match their individual workflow preferences.

### 1.2 User Value Proposition

**Problem:** The default composer layout serves the median user but fails power users who have specific workflow patterns. IDE-first developers require keyboard-driven workflows and minimal visual clutter. Different tasks (code review, documentation, chat) benefit from different control arrangements.

**Solution:** Layout Edit Mode empowers users to:
- Reorder controls within logical groupings (Rails)
- Hide rarely-used controls to reduce visual noise
- Restore hidden controls when needed
- Save custom layouts per user account
- Export/import layouts for team standardization
- Switch between preset layouts with one click

### 1.3 Technical Approach

The system implements a **constrained drag-and-drop model** using:

1. **Rails Architecture:** Three horizontal zones (Left, Right, Secondary) plus a Hidden Tray constrain component placement to valid locations only
2. **Snap-to-Slot Mechanics:** Components snap to predefined positionsâ€”no pixel-perfect dragging required
3. **Keyboard-First Design:** Full functionality via keyboard for IDE-integrated workflows
4. **JSON Persistence:** Layout configurations stored as JSON in user profiles with sync across devices
5. **Graceful Degradation:** Invalid/corrupted layouts fall back to defaults without data loss

**Target Implementation Time:** 4-6 sprint cycles  
**Dependencies:** Existing user preferences API, design system tokens  

---

## 2. Edit Mode Toggle

### 2.1 Button Placement

**RECOMMENDED:** Settings menu dropdown item

| Option | Location | Pros | Cons |
|--------|----------|------|------|
| **A. Settings Menu** âœ“ | User settings dropdown | Clean UI, discoverable, follows conventions | One extra click |
| B. Composer Footer | Right side, near send | Immediate access | Clutters primary interface |
| C. Keyboard Only | No visible button | Zero visual overhead | Poor discoverability |

**Implementation:** Add "Customize Layout" item to the user settings dropdown menu. Include a subtle keyboard shortcut hint `(âŒ˜â‡§L)`.

### 2.2 Visual Design

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  âš™ï¸ Settings                    â–¼   â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚  ðŸ‘¤ Account                         â”‚
â”‚  ðŸŽ¨ Appearance                      â”‚
â”‚  â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€  â”‚
â”‚  ðŸ”² Customize Layout      âŒ˜â‡§L      â”‚  â† Edit Mode Toggle
â”‚  â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€  â”‚
â”‚  ðŸ“¤ Export Layout                   â”‚
â”‚  ðŸ“¥ Import Layout                   â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

### 2.3 States

| State | Visual Treatment | Behavior |
|-------|------------------|----------|
| **OFF** (Default) | Menu item shows "Customize Layout" | Normal composer interaction |
| **ON** (Edit Mode) | Menu item shows "âœ“ Exit Edit Mode", blue highlight on composer | Rails visible, drag handles appear, control bar shown |

### 2.4 Keyboard Shortcut

**Primary:** `âŒ˜â‡§L` (Mac) / `Ctrl+Shift+L` (Windows/Linux)  
**Secondary:** None (avoid conflicts with common IDE shortcuts)

**Conflict Check:** This combination is unused in VS Code, JetBrains IDEs, and major browsers.

### 2.5 First-Use Onboarding

On first activation, display a dismissable tooltip overlay:

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  ðŸŽ¨ CUSTOMIZE YOUR LAYOUT                                   âœ•   â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚                                                                 â”‚
â”‚  Drag controls to reorder them within each colored zone.        â”‚
â”‚  Drag to the Hidden Tray to hide controls you don't use.        â”‚
â”‚                                                                 â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”  Left Rail (Setup)                                 â”‚
â”‚  â”‚ ðŸŸ¦ðŸŸ¦ðŸŸ¦  â”‚  Right Rail (Actions)                              â”‚
â”‚  â”‚    ðŸŸ§ðŸŸ§ â”‚  Secondary Strip (Optional)                        â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜                                                    â”‚
â”‚                                                                 â”‚
â”‚  Keyboard: Tab to navigate, Space to pick up, Arrows to move    â”‚
â”‚                                                                 â”‚
â”‚  [Got It]                              [ ] Don't show again     â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

**Persistence:** Onboarding dismissed state stored in `user_preferences.layout_onboarding_seen: boolean`

---

## 3. Rails System

### 3.1 Architecture Overview

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                              COMPOSER CONTAINER                             â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚                                                                             â”‚
â”‚   RAIL 1 (Left Cluster)              â”‚           RAIL 2 (Right Cluster)     â”‚
â”‚   â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”   â”‚
â”‚   â”‚ [Prompt Hub] [Add/Attach â–¼] [Mode â–¼]    [ðŸŒ Web] [âš¡ Effort] [Sendâž¤]â”‚   â”‚
â”‚   â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜   â”‚
â”‚                                                                             â”‚
â”‚   RAIL 3 (Secondary Strip) â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€    â”‚
â”‚   â”‚ [Tokens: 2.4k/100k]  [Queue: 3]  [Project: acme-api]                â”‚   â”‚
â”‚   â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜   â”‚
â”‚                                                                             â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚   HIDDEN TRAY (Collapsed by default in edit mode)                           â”‚
â”‚   â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”   â”‚
â”‚   â”‚ [Hidden Items: 2]  â–¶ [Mode Selector] [Token Counter]                â”‚   â”‚
â”‚   â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜   â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

### 3.2 Rail 1: Left Cluster (Setup Components)

**Purpose:** Components used to configure the request before sending

**Components:**
| Component | Default Position | Draggable | Hideable |
|-----------|------------------|-----------|----------|
| Prompt Hub | 1 | âœ… Yes | âœ… Yes |
| Add/Attach | 2 | âœ… Yes | âŒ No |
| Mode Selector | 3 | âœ… Yes | âœ… Yes |

**Visual Appearance in Edit Mode:**

```
â”Œâ”€ RAIL 1: Setup â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                                                         â”‚
â”‚  â•”â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•—  â•”â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•—  â•”â•â•â•â•â•â•â•â•â•â•â•â•â•â•—  â”‚
â”‚  â•‘ â‹®â‹® Prompt Hub â•‘  â•‘ â‹®â‹® Add/Attach â•‘  â•‘ â‹®â‹® Mode    â•‘  â”‚
â”‚  â•šâ•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•  â•šâ•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•  â•šâ•â•â•â•â•â•â•â•â•â•â•â•â•â•  â”‚
â”‚        [1]                [2]               [3]         â”‚
â”‚                                                         â”‚
â”‚  Background: rgba(59, 130, 246, 0.1) - Light Blue       â”‚
â”‚  Border: 2px dashed #3B82F6                             â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

**Drop Zone Indicators:**
- **Valid:** Green vertical line (2px, animated pulse) appears between components
- **Invalid (wrong rail):** Red overlay with ðŸš« icon and shake animation

**Constraints:**
- Components in Rail 1 CANNOT be dragged to Rail 2 or Rail 3
- Minimum 1 component must remain in Rail 1 (Add/Attach is non-hideable)

**Error Messages:**
| Attempted Action | Message |
|------------------|---------|
| Drag to Rail 2 | "Setup controls must stay in the left cluster" |
| Drag to Rail 3 | "Setup controls must stay in the left cluster" |
| Hide last component | "At least one setup control must remain visible" |

### 3.3 Rail 2: Right Cluster (Execution Components)

**Purpose:** Components related to executing/sending the request

**Components:**
| Component | Default Position | Draggable | Hideable |
|-----------|------------------|-----------|----------|
| Web Toggle | 1 | âœ… Yes | âœ… Yes |
| Effort Toggle | 2 | âœ… Yes | âœ… Yes |
| Send/Stop Button | 3 (Fixed) | âŒ No | âŒ No |

**Visual Appearance in Edit Mode:**

```
â”Œâ”€ RAIL 2: Actions â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                                                         â”‚
â”‚  â•”â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•—  â•”â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•—  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”   â”‚
â”‚  â•‘ â‹®â‹® ðŸŒ Web    â•‘  â•‘ â‹®â‹® âš¡ Effort   â•‘  â”‚ ðŸ”’ Send âž¤ â”‚   â”‚
â”‚  â•šâ•â•â•â•â•â•â•â•â•â•â•â•â•â•â•  â•šâ•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜   â”‚
â”‚        [1]               [2]              [FIXED]       â”‚
â”‚                                                         â”‚
â”‚  Background: rgba(34, 197, 94, 0.1) - Light Green       â”‚
â”‚  Border: 2px dashed #22C55E                             â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

**Send/Stop Button Treatment:**
- Displayed with lock icon and "FIXED" label
- Gray drag handle (non-functional)
- Tooltip on hover: "This control cannot be moved or hidden"
- Always occupies rightmost position

**Drop Zone Indicators:**
- **Valid:** Green vertical line between draggable components only
- **Invalid (at Send position):** No drop zone indicator renders

**Constraints:**
- Components in Rail 2 CANNOT be dragged to Rail 1 or Rail 3
- Send/Stop button is immovable anchor

**Error Messages:**
| Attempted Action | Message |
|------------------|---------|
| Drag to Rail 1 | "Action controls must stay in the right cluster" |
| Drag Send button | "The Send button cannot be moved" |
| Hide Send button | "The Send button cannot be hidden" |

### 3.4 Rail 3: Secondary Strip (Optional Row)

**Purpose:** Supplementary information and project context

**Components:**
| Component | Default Position | Draggable | Hideable |
|-----------|------------------|-----------|----------|
| Token Counter | 1 | âœ… Yes | âœ… Yes |
| Queue Indicator | 2 | âœ… Yes | âœ… Yes |
| Project Selector | 3 | âœ… Yes | âœ… Yes |

**Visual Appearance in Edit Mode:**

```
â”Œâ”€ RAIL 3: Info Strip (Can be hidden entirely) â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                                                         â”‚
â”‚  â•”â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•—  â•”â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•—  â•”â•â•â•â•â•â•â•â•â•â•â•â•â•—  â”‚
â”‚  â•‘ â‹®â‹® Tokens     â•‘  â•‘ â‹®â‹® Queue      â•‘  â•‘ â‹®â‹® Project â•‘  â”‚
â”‚  â•šâ•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•  â•šâ•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•  â•šâ•â•â•â•â•â•â•â•â•â•â•â•â•  â”‚
â”‚         [1]               [2]               [3]         â”‚
â”‚                                                         â”‚
â”‚  [Hide entire strip]                                    â”‚
â”‚                                                         â”‚
â”‚  Background: rgba(168, 85, 247, 0.1) - Light Purple     â”‚
â”‚  Border: 2px dashed #A855F7                             â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

**Hide Entire Strip:**
- Button at bottom of Rail 3: "Hide entire strip"
- When all Rail 3 components are hidden, the strip collapses
- To restore: Drag any Rail 3 component from Hidden Tray

**Constraints:**
- Components in Rail 3 CANNOT be dragged to Rail 1 or Rail 2
- All components are hideable

### 3.5 Hidden Tray

**Purpose:** Repository for components the user wants to hide but may restore later

**Visual Appearance:**

```
â”Œâ”€ HIDDEN TRAY â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                                                         â”‚
â”‚  ðŸ“¦ Hidden Components (3)                     [Expand â–¼]â”‚
â”‚                                                         â”‚
â”‚  â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”‚
â”‚  â”‚                                                   â”‚  â”‚
â”‚  â”‚  â•”â•â•â•â•â•â•â•â•â•â•â•â•â•—  â•”â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•—  â•”â•â•â•â•â•â•â•â•â•â•â•—  â”‚  â”‚
â”‚  â”‚  â•‘ Mode       â•‘  â•‘ Token Counter â•‘  â•‘ Queue    â•‘  â”‚  â”‚
â”‚  â”‚  â•šâ•â•â•â•â•â•â•â•â•â•â•â•â•  â•šâ•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•  â•šâ•â•â•â•â•â•â•â•â•â•â•  â”‚  â”‚
â”‚  â”‚  Rail 1           Rail 3              Rail 3      â”‚  â”‚
â”‚  â”‚                                                   â”‚  â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜  â”‚
â”‚                                                         â”‚
â”‚  Background: rgba(107, 114, 128, 0.1) - Light Gray      â”‚
â”‚  Border: 2px dashed #6B7280                             â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

**Behavior:**
- **Drag to hide:** Drag any hideable component to the Hidden Tray
- **Drag to restore:** Drag from Hidden Tray back to the component's original rail
- **Rail indicator:** Each hidden component shows which rail it belongs to
- **Restore shortcut:** Double-click restores component to its default position

**Constraints:**
- Components can ONLY be dragged from Hidden Tray to their designated rail
- Attempting to drag to wrong rail shows error

---

## 4. Draggable Components Inventory

### 4.1 Complete Component Registry

| ID | Component Name | Default Rail | Default Position | Draggable | Hideable | Dependencies |
|----|----------------|--------------|------------------|-----------|----------|--------------|
| `prompt-hub` | Prompt Hub | Rail 1 | 1 | âœ… Yes | âœ… Yes | None - prompts accessible via keyboard |
| `add-attach` | Add/Attach | Rail 1 | 2 | âœ… Yes | âŒ No | Core functionality - cannot be hidden |
| `mode-selector` | Mode Selector | Rail 1 | 3 | âœ… Yes | âœ… Yes | Defaults to "Chat" if hidden |
| `web-toggle` | Web Toggle | Rail 2 | 1 | âœ… Yes | âœ… Yes | Web search disabled if hidden |
| `effort-toggle` | Effort Toggle | Rail 2 | 2 | âœ… Yes | âœ… Yes | Uses default effort if hidden |
| `send-stop` | Send/Stop Button | Rail 2 | 3 | âŒ No | âŒ No | Core functionality - fixed position |
| `token-counter` | Token Counter | Rail 3 | 1 | âœ… Yes | âœ… Yes | Info only - no functional impact |
| `queue-indicator` | Queue Indicator | Rail 3 | 2 | âœ… Yes | âœ… Yes | Queue still functions if hidden |
| `project-selector` | Project Selector | Rail 3 | 3 | âœ… Yes | âœ… Yes | Uses current project if hidden |

### 4.2 Component Metadata Schema

```json
{
  "component_id": "prompt-hub",
  "display_name": "Prompt Hub",
  "icon": "library",
  "rail_constraint": "rail-1",
  "default_position": 1,
  "is_draggable": true,
  "is_hideable": true,
  "dependencies": [],
  "hidden_behavior": "Prompts accessible via âŒ˜K shortcut",
  "min_width": 120,
  "max_width": 200
}
```

### 4.3 Dependency Warnings

When hiding a component with dependencies, display a warning:

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  âš ï¸ Hiding Web Toggle                                   â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚                                                         â”‚
â”‚  Web search will be disabled when this control is       â”‚
â”‚  hidden. You can still enable it via the command        â”‚
â”‚  palette (âŒ˜K â†’ "Toggle Web Search").                   â”‚
â”‚                                                         â”‚
â”‚  [Cancel]                              [Hide Anyway]    â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

---

## 5. Drag-and-Drop Behavior

### 5.1 Snap-to-Slot Mechanics

**Philosophy:** No pixel-perfect positioning. Components snap to discrete slots within their rail.

**Slot Definition:**
- Each rail contains N slots where N = max(current_components, 6)
- Slots are evenly distributed across rail width
- Components occupy exactly one slot
- Empty slots are valid drop targets

**Snap Animation:**
```
Duration: 150ms
Easing: cubic-bezier(0.25, 0.1, 0.25, 1)
Properties: transform, opacity
```

### 5.2 Drag Handle Design

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  â‹®â‹®                                  â”‚
â”‚  â‹®â‹®  [Component Content]             â”‚
â”‚  â‹®â‹®                                  â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜

Handle Specs:
- Width: 16px
- Height: Component height - 8px
- Position: Left edge, vertically centered
- Color: #9CA3AF (gray-400)
- Hover: #6B7280 (gray-500), cursor: grab
- Active: #4B5563 (gray-600), cursor: grabbing
```

**Non-Draggable Handle (Send Button):**
```
Handle Color: #D1D5DB (gray-300)
Cursor: not-allowed
Tooltip: "This control cannot be moved"
```

### 5.3 Insertion Indicator Design

```
Valid Insertion Point:
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”â”‚â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ Component  â”‚â”‚â”‚ Component  â”‚
â”‚     A      â”‚â”‚â”‚     B      â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜â”‚â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
              â”‚
              â–¼
         Indicator Specs:
         - Width: 2px
         - Height: Component height + 16px
         - Color: #22C55E (green-500)
         - Animation: opacity pulse 0.5â†’1.0, 800ms, infinite
```

### 5.4 Invalid Placement Feedback

**Visual Feedback:**
```
1. Rail background flashes red (200ms)
2. Drop zone shows ðŸš« icon
3. Component shakes (CSS animation):
   @keyframes shake {
     0%, 100% { transform: translateX(0); }
     25% { transform: translateX(-4px); }
     75% { transform: translateX(4px); }
   }
4. Toast notification appears
```

**Error Toast:**
```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚ ðŸš« Invalid Position                     â”‚
â”‚ Setup controls must stay in left rail   â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
Duration: 3000ms, dismissable
```

### 5.5 Drag States

| State | Visual Treatment |
|-------|------------------|
| **Idle** | Normal appearance |
| **Hover** | Drag handle highlighted, subtle elevation shadow |
| **Dragging** | 50% opacity at origin, full opacity drag ghost follows cursor |
| **Over Valid Drop** | Insertion indicator visible, drop zone highlights |
| **Over Invalid Drop** | Red overlay on target rail, cursor: no-drop |
| **Dropped** | Snap animation to final position |

### 5.6 Touch Support (Mobile/Tablet)

**Touch Activation:**
- Long press (300ms) activates drag mode
- Visual feedback: Component scales to 1.05x, subtle haptic (if supported)

**Touch Drag Behavior:**
- Component follows touch point with 8px offset (finger visibility)
- Auto-scroll when dragging near rail edges
- Tap outside component cancels drag

**Touch-Specific Affordances:**
- Larger touch targets: Drag handle minimum 44Ã—44px
- Drop zones expand by 16px on touch devices
- "Done Editing" floating button appears during touch edit mode

### 5.7 Multi-Select Dragging (Optional - Phase 2)

**Activation:**
- Hold `âŒ˜/Ctrl` while clicking components
- Selected components show checkmark overlay

**Behavior:**
- All selected components move together
- Must be within same rail
- Positions maintain relative order

**Visual:**
```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  â•”â•â•â•â•â•â•â•â•â•â•â•â•—  â•”â•â•â•â•â•â•â•â•â•â•â•â•â•—  â•”â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•—  â”‚
â”‚  â•‘ âœ“ Comp A  â•‘  â•‘    Comp B  â•‘  â•‘  âœ“ Comp C        â•‘  â”‚
â”‚  â•šâ•â•â•â•â•â•â•â•â•â•â•â•  â•šâ•â•â•â•â•â•â•â•â•â•â•â•â•  â•šâ•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•  â”‚
â”‚  [Selected]                       [Selected]          â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

---

## 6. Keyboard Controls

### 6.1 Complete Keyboard Reference

| Key | Action | Context |
|-----|--------|---------|
| `Tab` | Move focus to next component | Edit mode active |
| `Shift+Tab` | Move focus to previous component | Edit mode active |
| `Space` | Pick up focused component | Component focused |
| `Enter` | Pick up focused component (alias) | Component focused |
| `â†` | Move held component left one slot | Component picked up |
| `â†’` | Move held component right one slot | Component picked up |
| `â†‘` | Move to Hidden Tray | Component picked up |
| `â†“` | Move from Hidden Tray to rail | In Hidden Tray |
| `Escape` | Cancel drag / Exit edit mode | Any state |
| `Delete` | Send focused component to Hidden Tray | Component focused |
| `Backspace` | Send focused component to Hidden Tray (alias) | Component focused |
| `H` | Toggle Hidden Tray visibility | Edit mode active |
| `1` | Jump focus to Rail 1 | Edit mode active |
| `2` | Jump focus to Rail 2 | Edit mode active |
| `3` | Jump focus to Rail 3 | Edit mode active |
| `0` | Jump focus to Hidden Tray | Edit mode active |
| `âŒ˜S` / `Ctrl+S` | Save layout | Edit mode active |
| `âŒ˜Z` / `Ctrl+Z` | Undo last move | Edit mode active |
| `âŒ˜â‡§Z` / `Ctrl+Shift+Z` | Redo | Edit mode active |

### 6.2 Focus Indicator Design

```
Focused Component:
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  â‹®â‹®  [Component Content]          â”‚
â”‚      â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•          â”‚  â† 3px blue underline
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜

Focus Ring: 3px solid #3B82F6 (blue-500)
Offset: 2px from component edge
Animation: None (instant focus for accessibility)
```

### 6.3 Picked-Up State

```
Picked-Up Component:
â•”â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•—
â•‘  â‹®â‹®  [Component Content]    ðŸ”€    â•‘  â† Move icon appears
â•šâ•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•

Background: #DBEAFE (blue-100)
Border: 2px solid #3B82F6
Status Bar: "Use â† â†’ to move, â†‘ to hide, Space to drop"
```

### 6.4 Screen Reader Announcements

| Event | Announcement |
|-------|--------------|
| Focus component | "{Component name}, position {N} of {total} in {Rail name}" |
| Pick up | "{Component name} picked up. Use arrow keys to move." |
| Move | "{Component name} moved to position {N}" |
| Invalid move | "Cannot move {Component name} to {Rail name}" |
| Drop | "{Component name} dropped at position {N}" |
| Hide | "{Component name} hidden" |
| Restore | "{Component name} restored to {Rail name}" |

---

## 7. Control Bar

### 7.1 Control Bar Layout

When edit mode is active, a control bar appears below the composer:

```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                             EDIT MODE ACTIVE                                â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚                                                                             â”‚
â”‚   [ðŸ’¾ Save Layout]    [âœ• Cancel]    [â†º Reset to Default]    [ðŸ“¥ Presets â–¼] â”‚
â”‚                                                                             â”‚
â”‚   Status: 2 unsaved changes                                                 â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

### 7.2 Button Specifications

| Button | Appearance | Behavior | Shortcut |
|--------|------------|----------|----------|
| **Save Layout** | Primary (blue), filled | Commits current layout to database | `âŒ˜S` |
| **Cancel** | Secondary (gray), outlined | Reverts to last saved state | `Escape` |
| **Reset to Default** | Destructive (red), text only | Restores factory default | None |
| **Presets** | Secondary dropdown | Opens preset layouts menu | `P` |

### 7.3 Confirmation Dialogs

**Cancel with Unsaved Changes:**
```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  Discard Changes?                                   âœ•   â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚                                                         â”‚
â”‚  You have 2 unsaved changes. Are you sure you want      â”‚
â”‚  to discard them?                                       â”‚
â”‚                                                         â”‚
â”‚  [Keep Editing]                        [Discard]        â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

**Reset to Default:**
```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  Reset Layout?                                      âœ•   â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚                                                         â”‚
â”‚  This will restore the default component arrangement.   â”‚
â”‚  Your current layout will be lost.                      â”‚
â”‚                                                         â”‚
â”‚  ðŸ’¡ Tip: Export your layout first to save a backup.    â”‚
â”‚                                                         â”‚
â”‚  [Cancel]                              [Reset Layout]   â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

### 7.4 Status Indicator

The control bar displays real-time status:

| Status | Display | Color |
|--------|---------|-------|
| No changes | "Layout unchanged" | Gray |
| Unsaved changes | "{N} unsaved changes" | Amber |
| Saving | "Saving..." with spinner | Blue |
| Saved | "Layout saved âœ“" (3s) | Green |
| Error | "Failed to save. Retry?" | Red |

---

## 8. Layout Persistence

### 8.1 Database Schema

**Table: `user_composer_layouts`**

```sql
CREATE TABLE user_composer_layouts (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    layout_name     VARCHAR(100) NOT NULL DEFAULT 'Custom',
    layout_data     JSONB NOT NULL,
    is_active       BOOLEAN NOT NULL DEFAULT false,
    device_id       VARCHAR(255),                    -- NULL = synced across devices
    created_at      TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    updated_at      TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    
    CONSTRAINT unique_active_layout_per_user 
        UNIQUE (user_id, is_active) WHERE is_active = true
);

CREATE INDEX idx_user_layouts_user_id ON user_composer_layouts(user_id);
CREATE INDEX idx_user_layouts_active ON user_composer_layouts(user_id, is_active);
```

### 8.2 JSON Layout Configuration Schema

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "ComposerLayout",
  "type": "object",
  "required": ["version", "rails", "hidden"],
  "properties": {
    "version": {
      "type": "string",
      "pattern": "^\\d+\\.\\d+\\.\\d+$",
      "description": "Schema version for migration support"
    },
    "rails": {
      "type": "object",
      "properties": {
        "rail1": {
          "type": "array",
          "items": { "type": "string" },
          "description": "Component IDs in order"
        },
        "rail2": {
          "type": "array",
          "items": { "type": "string" }
        },
        "rail3": {
          "type": "array",
          "items": { "type": "string" }
        }
      },
      "required": ["rail1", "rail2", "rail3"]
    },
    "hidden": {
      "type": "array",
      "items": { "type": "string" },
      "description": "Component IDs in Hidden Tray"
    },
    "metadata": {
      "type": "object",
      "properties": {
        "created_at": { "type": "string", "format": "date-time" },
        "modified_at": { "type": "string", "format": "date-time" },
        "preset_base": { "type": "string" }
      }
    }
  }
}
```

**Example Layout:**

```json
{
  "version": "1.0.0",
  "rails": {
    "rail1": ["add-attach", "prompt-hub"],
    "rail2": ["effort-toggle", "web-toggle", "send-stop"],
    "rail3": ["project-selector"]
  },
  "hidden": ["mode-selector", "token-counter", "queue-indicator"],
  "metadata": {
    "created_at": "2025-01-02T10:30:00Z",
    "modified_at": "2025-01-02T14:22:00Z",
    "preset_base": "minimal"
  }
}
```

### 8.3 Save/Load Workflow

**Save Workflow:**
```
1. User clicks "Save Layout" or presses âŒ˜S
2. Client validates layout JSON against schema
3. Client sends PUT /api/user/composer-layout
4. Server validates and persists to database
5. Server returns { success: true, layout_id: "..." }
6. Client shows "Layout saved âœ“" confirmation
```

**Load Workflow:**
```
1. User logs in / App initializes
2. Client sends GET /api/user/composer-layout
3. Server returns active layout or null (use default)
4. Client validates layout version
5. If version mismatch â†’ run migration
6. Client applies layout to composer
```

### 8.4 Per-User vs Per-Device Options

**Sync Modes:**

| Mode | `device_id` Value | Behavior |
|------|-------------------|----------|
| **Synced** (Default) | `NULL` | Same layout on all devices |
| **Per-Device** | Device fingerprint | Unique layout per device |

**User Setting:**
```
Settings â†’ Composer â†’ Layout Sync
â—‹ Sync across all devices (recommended)
â—‹ Separate layout per device
```

---

## 9. Import/Export

### 9.1 Export Layout

**Trigger:** Settings menu â†’ "Export Layout" or `âŒ˜â‡§E`

**File Format Options:**
| Format | Extension | Use Case |
|--------|-----------|----------|
| JSON | `.json` | Programmatic access, team tooling |
| YAML | `.yaml` | Human-readable editing |

**Export File Structure:**

```json
{
  "export_format": "composer-layout",
  "export_version": "1.0.0",
  "exported_at": "2025-01-02T10:30:00Z",
  "layout": {
    "version": "1.0.0",
    "rails": { ... },
    "hidden": [ ... ]
  }
}
```

**File Naming:** `composer-layout-{timestamp}.json`

### 9.2 Import Layout

**Trigger:** Settings menu â†’ "Import Layout" or `âŒ˜â‡§I`

**Import Flow:**
```
1. User selects file via native file picker
2. Client parses JSON/YAML
3. Client validates against schema
4. If valid â†’ Preview dialog shown
5. User confirms â†’ Layout applied and saved
6. If invalid â†’ Error dialog with details
```

### 9.3 Validation on Import

**Validation Steps:**

| Check | Failure Behavior |
|-------|------------------|
| File parseable | "Invalid file format" error |
| Schema valid | "Layout structure invalid" error |
| Version compatible | Run migration or reject |
| Component IDs exist | Remove unknown, warn user |
| No duplicate IDs | Remove duplicates, warn |
| Rail constraints | Move to correct rail, warn |

### 9.4 Conflict Resolution

**Unknown Component Handling:**
```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  Import Warning                                     âœ•   â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚                                                         â”‚
â”‚  The imported layout contains 2 unknown components:     â”‚
â”‚  â€¢ "custom-plugin-a"                                    â”‚
â”‚  â€¢ "deprecated-feature"                                 â”‚
â”‚                                                         â”‚
â”‚  These will be skipped. Continue with partial import?   â”‚
â”‚                                                         â”‚
â”‚  [Cancel]                              [Import Anyway]  â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

### 9.5 Community Layout Sharing (Optional - Phase 2)

**Public Layout Gallery:**
- Browse community-shared layouts
- Preview before applying
- Rating/popularity metrics
- "Share My Layout" button (anonymized export)

**Implementation Note:** Requires moderation pipeline for shared content.

---

## 10. Default Layouts

### 10.1 Preset Definitions

**1. Minimal**
```json
{
  "rails": {
    "rail1": ["add-attach"],
    "rail2": ["send-stop"],
    "rail3": []
  },
  "hidden": ["prompt-hub", "mode-selector", "web-toggle", "effort-toggle", "token-counter", "queue-indicator", "project-selector"]
}
```
*Best for: Simple chat interactions, minimal UI.*

**2. Balanced (DEFAULT)**
```json
{
  "rails": {
    "rail1": ["prompt-hub", "add-attach", "mode-selector"],
    "rail2": ["web-toggle", "effort-toggle", "send-stop"],
    "rail3": ["token-counter", "project-selector"]
  },
  "hidden": ["queue-indicator"]
}
```
*Best for: Most users, good balance of features and simplicity.*

**3. Power User**
```json
{
  "rails": {
    "rail1": ["prompt-hub", "add-attach", "mode-selector"],
    "rail2": ["web-toggle", "effort-toggle", "send-stop"],
    "rail3": ["token-counter", "queue-indicator", "project-selector"]
  },
  "hidden": []
}
```
*Best for: Users who want all controls visible.*

**4. Code Review**
```json
{
  "rails": {
    "rail1": ["add-attach", "prompt-hub"],
    "rail2": ["effort-toggle", "send-stop"],
    "rail3": ["token-counter", "project-selector"]
  },
  "hidden": ["mode-selector", "web-toggle", "queue-indicator"]
}
```
*Best for: Reviewing code files, effort control prioritized.*

**5. Documentation**
```json
{
  "rails": {
    "rail1": ["prompt-hub", "add-attach"],
    "rail2": ["web-toggle", "send-stop"],
    "rail3": ["token-counter"]
  },
  "hidden": ["mode-selector", "effort-toggle", "queue-indicator", "project-selector"]
}
```
*Best for: Writing documentation, web search enabled.*

### 10.2 Preset Switcher UI

```
â”Œâ”€ Select Preset â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                                                         â”‚
â”‚   â—‹ Minimal                                             â”‚
â”‚     Clean interface, only essential controls            â”‚
â”‚                                                         â”‚
â”‚   â— Balanced âœ“ Current                                  â”‚
â”‚     Good for most users (Default)                       â”‚
â”‚                                                         â”‚
â”‚   â—‹ Power User                                          â”‚
â”‚     All controls visible                                â”‚
â”‚                                                         â”‚
â”‚   â—‹ Code Review                                         â”‚
â”‚     Optimized for reviewing code                        â”‚
â”‚                                                         â”‚
â”‚   â—‹ Documentation                                       â”‚
â”‚     Optimized for writing docs                          â”‚
â”‚                                                         â”‚
â”‚   â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€     â”‚
â”‚   â—‹ Custom                                              â”‚
â”‚     Your saved custom layout                            â”‚
â”‚                                                         â”‚
â”‚  [Cancel]                               [Apply Preset]  â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

---

## 11. Edge Cases & Error Handling

### 11.1 Component Slot Conflicts

**Scenario:** Two components targeting the same slot during import/migration.

**Resolution:**
1. First component wins slot
2. Second component placed in next available slot
3. Log conflict for debugging

### 11.2 Migration: New Components in Updates

**Scenario:** App update adds new component (e.g., "voice-input").

**Migration Strategy:**
```javascript
function migrateLayout(layout, fromVersion, toVersion) {
  if (fromVersion < "1.1.0" && toVersion >= "1.1.0") {
    // Add voice-input to hidden tray by default
    if (!layout.hidden.includes("voice-input")) {
      layout.hidden.push("voice-input");
    }
  }
  layout.version = toVersion;
  return layout;
}
```

**User Notification:**
```
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  ðŸ†• New Component Available                             â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚                                                         â”‚
â”‚  "Voice Input" has been added to your Hidden Tray.      â”‚
â”‚  Enter Edit Mode to add it to your layout.              â”‚
â”‚                                                         â”‚
â”‚  [Dismiss]                         [Open Edit Mode]     â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

### 11.3 Corrupted Layout Data

**Detection:**
- JSON parse failure
- Schema validation failure
- Missing required components (e.g., `send-stop`)

**Recovery:**
```
1. Log error with layout data to monitoring
2. Fall back to "Balanced" default layout
3. Show toast: "Layout couldn't be loaded. Using default."
4. Preserve corrupted layout in separate storage for debugging
```

### 11.4 Browser Storage Limits

**Local Storage Constraints:**
- Most browsers: 5-10MB per origin
- Layout JSON: ~500 bytes (negligible)

**Mitigation:**
- Store in database (primary)
- Local storage as offline cache only
- Clear cache on storage quota error

### 11.5 Layout Rendering Failures

**Scenario:** Layout causes JS error during render.

**Recovery:**
```javascript
try {
  renderLayout(userLayout);
} catch (error) {
  logError("Layout render failed", { layout: userLayout, error });
  renderLayout(DEFAULT_LAYOUT);
  showToast("Layout error. Reset to fix.");
}
```

### 11.6 Concurrent Edit Detection

**Scenario:** User edits layout in two tabs/devices simultaneously.

**Resolution:**
- Last-write-wins (simple)
- Timestamp comparison on save
- Conflict dialog if timestamps diverge by >5 minutes

---

## 12. Implementation Notes

### 12.1 Recommended Libraries

| Concern | Recommendation | Rationale |
|---------|----------------|-----------|
| Drag-and-Drop | **@dnd-kit/core** | Accessible, keyboard-first, performant |
| State Management | Existing app state (Redux/Zustand) | Consistency with codebase |
| Animations | **Framer Motion** | Smooth snap animations |
| Schema Validation | **Zod** | TypeScript-first, runtime validation |

**@dnd-kit Advantages:**
- First-class keyboard support
- Accessible by default (ARIA)
- Sortable presets for rail ordering
- Touch sensor included
- Framework agnostic

### 12.2 Performance Considerations

**Render Optimization:**
- Virtualize Hidden Tray if >20 components
- Debounce layout saves (500ms)
- Memoize component renders during drag

**Bundle Size:**
- @dnd-kit/core: ~15KB gzipped
- Framer Motion: ~30KB gzipped
- Consider code-splitting edit mode

**Metrics to Track:**
- Time to enter/exit edit mode
- Drag operation frame rate
- Save latency (p50, p95, p99)

### 12.3 Browser Compatibility

| Browser | Version | Notes |
|---------|---------|-------|
| Chrome | 90+ | Full support |
| Firefox | 88+ | Full support |
| Safari | 14+ | Full support |
| Edge | 90+ | Full support (Chromium) |
| Mobile Safari | 14+ | Touch sensors |
| Chrome Android | 90+ | Touch sensors |

**Polyfills Required:**
- None for target browsers
- ResizeObserver (native in all targets)

### 12.4 Testing Strategy

**Unit Tests:**
- Layout JSON validation
- Migration functions
- Constraint enforcement logic
- Keyboard navigation state machine

**Integration Tests:**
- Save/load round-trip
- Import/export round-trip
- Preset switching

**E2E Tests:**
- Full drag-and-drop workflow
- Keyboard-only workflow
- Edit mode toggle
- Error recovery flows

**Accessibility Tests:**
- Screen reader announcements
- Keyboard trap prevention
- Focus management
- Color contrast in edit mode

**Visual Regression Tests:**
- Rail highlighting
- Drop indicators
- Control bar states
- Hidden Tray expanded/collapsed

---

## Appendix A: ASCII Layout Reference

```
NORMAL MODE:
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚                                                                         â”‚
â”‚  [Prompt Hub] [Add/Attach â–¼] [Mode â–¼]        [ðŸŒ] [âš¡] [Send âž¤]        â”‚
â”‚                                                                         â”‚
â”‚  Tokens: 2.4k/100k  â”‚  Queue: 3  â”‚  Project: acme-api                   â”‚
â”‚                                                                         â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜

EDIT MODE:
â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”
â”‚  â”Œâ”€ RAIL 1: Setup â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¬â”€ RAIL 2: Actions â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”‚
â”‚  â”‚                                  â”‚                               â”‚  â”‚
â”‚  â”‚ â•”â•â•â•â•â•â•â•â•â•â•â•â•— â•”â•â•â•â•â•â•â•â•â•â•â•â•— â•”â•â•â•â•—â”‚â•”â•â•â•â•â•â•â•â•— â•”â•â•â•â•â•â•â•â•— â”Œâ”€â”€â”€â”€â”€â”€â”€â”€â” â”‚  â”‚
â”‚  â”‚ â•‘â‹®â‹®PromptHubâ•‘ â•‘â‹®â‹®Add/Attachâ•‘ â•‘â‹®â‹®Mâ•‘â”‚â•‘â‹®â‹® ðŸŒ â•‘ â•‘â‹®â‹® âš¡ â•‘ â”‚ðŸ”’ Sendâ”‚ â”‚  â”‚
â”‚  â”‚ â•šâ•â•â•â•â•â•â•â•â•â•â•â• â•šâ•â•â•â•â•â•â•â•â•â•â•â• â•šâ•â•â•â•â”‚â•šâ•â•â•â•â•â•â•â• â•šâ•â•â•â•â•â•â•â• â””â”€â”€â”€â”€â”€â”€â”€â”€â”˜ â”‚  â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”´â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜  â”‚
â”‚                                                                         â”‚
â”‚  â”Œâ”€ RAIL 3: Info Strip (Can be hidden) â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”‚
â”‚  â”‚ â•”â•â•â•â•â•â•â•â•â•â•â•â•â•— â•”â•â•â•â•â•â•â•â•â•â•â•â•— â•”â•â•â•â•â•â•â•â•â•â•â•â•â•â•â•—                    â”‚  â”‚
â”‚  â”‚ â•‘â‹®â‹® Tokens  â•‘ â•‘â‹®â‹® Queue   â•‘ â•‘â‹®â‹® Project   â•‘   [Hide Strip]     â”‚  â”‚
â”‚  â”‚ â•šâ•â•â•â•â•â•â•â•â•â•â•â•â• â•šâ•â•â•â•â•â•â•â•â•â•â•â• â•šâ•â•â•â•â•â•â•â•â•â•â•â•â•â•â•                    â”‚  â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜  â”‚
â”‚                                                                         â”‚
â”‚  â”Œâ”€ HIDDEN TRAY â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”  â”‚
â”‚  â”‚ ðŸ“¦ Hidden (0)                                           [Expand] â”‚  â”‚
â”‚  â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜  â”‚
â”‚                                                                         â”‚
â”œâ”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”¤
â”‚  [ðŸ’¾ Save]  [âœ• Cancel]  [â†º Reset]  [ðŸ“¥ Presets â–¼]   â”‚ 0 unsaved changes â”‚
â””â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”€â”˜
```

---

## Appendix B: API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/user/composer-layout` | Get active layout |
| `PUT` | `/api/user/composer-layout` | Save layout |
| `DELETE` | `/api/user/composer-layout` | Reset to default |
| `GET` | `/api/composer-layout/presets` | List preset layouts |
| `POST` | `/api/user/composer-layout/validate` | Validate import JSON |

---

## Appendix C: Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0.0 | 2025-01-02 | Initial specification |

---

*End of Specification*
