<!-- markdownlint-disable -->
# Music Blocks (v4) - Masonry & AST Product Requirements Document (PRD)

# 1 Definations

### 1.1 Masonry Definition
- Masonry refers to the layout mechanism that arranges bricks like pieces in a puzzle or blocks in a wall.  
- Each category or type of brick (Statements, Blocks, Arguments) has specific connection rules and geometry constraints handled by the Masonry engine.

### 1.2 Abstract Syntax Tree Definition
- The AST is a tree-based data structure representing the program’s logic.  
- Each node corresponds to a programming construct (Statement, Block, Expression, etc.).  
- The AST is continuously synchronized with the on-screen arrangement of bricks.

---

## 2. Masonry System Requirements

### 2.1 Visual Arrangement of Bricks
1. Snap-to-Grid or Snap-to-Brick:  
   - Bricks must attach along predefined edges or “ports.”  
   - Visual guides or ghost outlines appear when a user drags a brick near a compatible connection point.
2. Stacking Logic:  
   - Statement bricks can stack vertically, while nested blocks enclose instructions.  
   - Argument bricks (expressions or data) align to the right side or within designated slot areas.
3. Drag & Drop:  
   - Users can freely drag bricks from the palette into the workspace.  
   - Dropping a brick onto another should attempt an auto-align if compatible.  
   - Incompatible or disallowed attachments display a warning highlight or rejection animation.
4. Masonry Sizing & Resizing:  
   - Bricks may resize based on the arguments they hold or nested instructions.  
   - Blocks can expand vertically as users add more statements.  
   - Dynamic reflow when a user adds or removes arguments to keep the layout readable.
5. Keyboard/Mouse Controls:  
   - Users can connect, detach, or rearrange via mouse clicks or keyboard shortcuts.  
   - Right-click context menu: “Duplicate,” “Delete,” “Help,” etc.

### 2.2 Rendering & Performance
1. Real-Time Layout Updates:  
   - On any drag or structural change, the interface reflows to maintain clarity.  
   - Use minimal CPU/GPU resources to sustain a smooth user experience.
2. Hierarchy & Layering:  
   - Ensure blocks with nested instructions remain visually distanced from other code blocks to avoid overlap.  
   - Possibly use layering or z-index management for overlapping elements (e.g., open vs. collapsed blocks).

### 2.3 Error Handling & Validation
1. Invalid Connection Handling:  
   - Display a distinct highlight (red outline) for an incorrect drop.  
   - Possibly show an error message or tooltip near the attempted connection.  
2. Partial or Orphaned Bricks:  
   - If a Statement or Argument is detached from a proper parent block, highlight or label it as incomplete.  
3. Debug-Mode Overlays:  
   - In a debugging session, highlight the active or stepped-through bricks with color-coded borders.

---

## 3. Abstract Syntax Tree (AST) Requirements

### 3.1 Core AST Structure
1. Node Types Matching Brick Types:  
   - **BlockNode**: Contains child nodes for nested instructions.  
   - **StatementNode**: Single-step operation, e.g., play a note.  
   - **ArgumentNode**: Data or Expression.  
     - **DataNode**: Literal or constant values (e.g., 42, “hello”).  
     - **ExpressionNode**: Operators or function calls returning a single value (e.g., add, subtract).
2. Links to Parent/Child Nodes:  
   - Each node keeps a reference to its parent node (except the root).  
   - Blocks hold arrays/lists of children (for contained instructions).  
   - Argument nodes attach to the instruction that references them.

### 3.2 AST Construction & Synchronization
1. Incremental AST Update:  
   - Whenever a user attaches a new brick, the corresponding AST node is generated and linked to the parent.  
   - Detaching a brick removes the corresponding AST sub-tree.  
2. Serialization/Deserialization:  
   - The AST must be serializable to JSON for saving or exporting the program.  
   - Loading a project from JSON should reconstruct the AST, then reflow the Masonry layout.
3. Consistency Checks:  
   - The AST enforces logic constraints (e.g., “repeat” block must have a numeric argument).  
   - If a user overrides constraints, the system either blocks the action or flags an error visually and in the AST.

### 3.3 Execution Model
1. Mapping to Processes & Routines:  
   - Each top-level sub-tree under “Start” can act as a separate Process.  
   - Routine definitions are sub-trees that can be invoked by Processes.  
2. Traversal & Interpretation:  
   - An interpreter or compiler systematically walks the AST nodes for execution.  
   - For concurrency, multiple AST sub-trees can be executed in parallel.

---

## 4. Implementation Plan

### 4.1 Phase 1: Masonry Foundation
1. **Initial Brick Geometry & Drag-Placement**  
   - Implement a grid or bounding-box system that detects valid connection points.  
   - Minimal validations: block vs. statement shapes.
2. **Basic AST Linking**  
   - Attach nodes to a single global tree structure.  
   - Ensure real-time synchronization: drag action → AST updates.

### 4.2 Phase 2: Advanced Masonry Features
1. **Dynamic Resizing & Auto-Arrange**  
   - Implement a layout algorithm that rearranges blocks to avoid collisions.  
   - Introduce collapsible sections for large blocks.
2. **Detailed Error Handling & Visual Feedback**  
   - Show user-friendly error messages/warnings in the workspace.  
   - Highlight or colorize incorrect bricks or connections.
3. **Refined Connection Logic**  
   - Enforce data type checking for arguments.  
   - Distinguish between numeric, string, or boolean expressions where relevant.

### 4.3 Phase 3: AST 
1. **Serialization & Deserialization**  
   - Convert the AST to and from JSON.  
   - Maintain referential integrity (e.g., routine references).
2. **Execution & Debug Hooks**  
   - Provide an interpreter or runtime that processes the AST.  
   - Deploy breakpoints in the AST (matching masonry bricks).  
   - Step execution to highlight active bricks.
3. **Routine & Process Management**  
   - Implement concurrency logic, letting multiple branches of the AST run in parallel.  
   - Incorporate user-defined routines invoked by multiple threads.

### 4.4 Phase 4: Integration & Testing
1. **Integration with the Main IDE**  
   - Combine with palette, project saving/loading, and UI for a cohesive experience.  
   - Synchronize with the Planet repository for shared projects.
2. **Extensive Testing & QA**  
   - Stress tests with large or deeply nested programs.  
   - Edge cases (e.g., cycle detection if user tries to reference a block within itself).
3. **Documentation & Tutorials**  
   - Document the Masonry system workflows and AST data structures.  
   - Provide visual guides and sample code.

---

## 5. UI/UX Documentation

### 5.1 Workspace Visualization
- **Bricks** appear as color-coded blocks, with small, labeled “slots” for arguments or nested instructions.  
- **Zooming**: Zoom in/out to focus on large or dense programs.  
- **Scrolling**: Scroll horizontally or vertically through the workspace.  
- **Collapsible Blocks**: A small toggle arrow in the corner of each block.  

### 5.2 Conflict Resolution
- If two bricks collide due to auto-arrange, the system gently shifts them apart, preserving clarity.  
- If a block is forced to overlap, the system signals an error, prompting the user to reposition.

### 5.3 User Feedback Elements
- **Hover Highlights**: Slight color shift or outline when the mouse is over a valid connect site.  
- **Tooltips**: Provide quick help on brick usage, argument types, and possible connections.  
- **Error Markers**: Red/colored outlines for incorrectly placed bricks, with an icon containing details.

---
