<!-- markdownlint-disable -->
# Music Blocks (v4) – Program Building PRD

Here’s a more conversational take on how Music Blocks (v4) manages its brick-based programming interface. It outlines key features in the Masonry layout, drag-and-drop mechanisms, error handling, and overall user interaction.

--------------------------------------------------------------------------------
## 1. Introduction

Music Blocks (v4) gives users a simple but powerful way to build programs by snapping together “bricks” (or blocks). The goals are:

• Offer a flexible workspace (a canvas) for placing these bricks.  
• Use a “Masonry” layout so bricks snap into place in a clean, puzzle-like manner.  
• Provide debugging tools so users can visually see and fix errors in their code.

--------------------------------------------------------------------------------
## 2. Masonry & Visual Arrangement

### 2.1 Masonry Overview
- “Masonry” is the layout engine that aligns bricks together.  
- Bricks have connection rules (like puzzle pieces) to ensure valid arrangements.  
- The system shows real-time visual feedback (outlines, highlights) to guide users.

### 2.2 Arranging Bricks
1. **Snap-to-Grid / Snap-to-Brick**  
   - Bricks attach to compatible “slots” with a visual cue (ghost outline or highlight).  
2. **Stacking Logic**  
   - Statements stack vertically.  
   - Blocks nest statements or sub-blocks inside them.  
   - Arguments snap into parameter slots on a given block.  
3. **Drag & Drop**  
   - Users drag bricks from a palette into the workspace.  
   - If the placement is valid, the system snaps them in.  
   - Invalid placements flash a warning or bounce back.  
4. **Masonry Sizing & Resizing**  
   - Bricks adapt their size if they contain more statements.  
   - Large blocks can expand vertically or allow collapsible sections.  
   - Masonry reflows automatically when users add or remove parameters.  
5. **Keyboard/Mouse Controls**  
   - Mouse-based dragging for simple operations.  
   - Keyboard shortcuts (optional) for navigation and attaching/detaching bricks.  
   - Right-click menus let users duplicate, delete, or seek help.

### 2.3 Rendering & Performance
1. **Real-Time Updates**  
   - The layout should update as soon as bricks are moved or connected.  
   - Must remain smooth and responsive.  
2. **Hierarchy & Layering**  
   - Nested blocks should appear visually distinct.  
   - Ensure no overlaps or flickering when moving bricks around.

### 2.4 Error Handling & Validation
1. **Invalid Connections**  
   - A red outline (or similar) appears for wrong placements.  
   - Tooltips can show the reason (“expected a number, found text,” etc.).  
2. **Orphaned Bricks**  
   - Detached bricks highlight themselves to show they’re incomplete.  
3. **Debug Mode Overlays**  
   - During debugging, highlight the currently running or stepped-through brick.

--------------------------------------------------------------------------------
## 3. Workspace Interaction & Features

### 3.1 Workspace Canvas
- The central region where all blocks are placed.  
- Offers zoom and pan for handling large, complex projects.  
- The palette can be minimized on smaller devices.

### 3.2 Snapping Mechanics
- Puzzle-like connection points guide correct placements.  
- Snap tolerance is around 10–15px to help with small misalignments.  
- Regions that can’t accept a given block don’t trigger snapping.

### 3.3 Drag and Drop
- Users see a “ghost” of the brick while dragging.  
- Dragging can be disabled if the program is running or in read-only mode.

### 3.4 Block Categorization & Searching
- Blocks are grouped by category (Data, Control, Music, etc.).  
- A search bar helps quickly filter bricks by name or function.  
- Results appear immediately, so users can grab the right block quickly.

--------------------------------------------------------------------------------
## 4. Program Building Phases

### 4.1 Phase 1: Basic Masonry
- Implement fundamental brick geometry and snapping.  
- Offer minimal checks for valid connections (e.g., statement vs. block).

### 4.2 Phase 2: Enhanced Layout & Error Handling
- Auto-arrange to avoid overlap and clutter.  
- Display user-friendly messages for incorrect placements.  
- Optionally collapse large blocks to keep the workspace tidy.

### 4.3 Phase 3: Customization & Validation
- Enforce data-type matching for argument slots (number vs. text, etc.).  
- Show real-time warnings when parameters are missing or invalid.  
- Refine keyboard controls and dynamic resizing.

### 4.4 Phase 4: Polishing & Refinement
- Integrate fully with the underlying AST (see “AST PRD”) for live updates.  
- Add debugging overlays that reveal the code path during execution.  
- Optimize performance for very large or complex programs.

--------------------------------------------------------------------------------
## 5. UI/UX Notes

### 5.1 Figma References
- The Design made by Anindiya 

### 5.2 Collapsible Blocks
- Blocks with many statements can have a collapse/expand toggle.  
- Save the collapsed state locally so it remains when the user reopens the app.

### 5.3 Interaction Tips
- Tooltips appear on hover, explaining each block or parameter.  
- Different data types can be color-coded for clarity.

--------------------------------------------------------------------------------