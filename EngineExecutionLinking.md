<!-- markdownlint-disable -->
# Music Blocks (v4) – Linking Program Building, AST, and Execution Engine PRD

Below is a clearer, more conversational overview of how the Program Building workspace, the Abstract Syntax Tree (AST), and the Execution Engine all connect to create a seamless user experience in Music Blocks (v4).

--------------------------------------------------------------------------------
## 1. Introduction

1. **Program Building (Workspace & Masonry)**  
   - A drag-and-drop interface that lets users assemble code “bricks.”  
   - Handles layout, snap mechanics, and user feedback.

2. **AST (Abstract Syntax Tree)**  
   - A structured representation of a user’s program logic.  
   - Preserves parent-child relationships and parallel execution references.

3. **Execution Engine**  
   - Runs or compiles the AST to produce music, graphics, or other outcomes.  
   - Supports running multiple processes in parallel.

These three components work together so users can effortlessly create, run, and debug their projects in real time.

--------------------------------------------------------------------------------
## 2. Linking Plan

### 2.1 Real-Time Synchronization
- **Add or Remove Bricks → AST Update**  
  Each time a user drags a block into place or removes one, the AST is instantly updated to mirror those changes.  
- **AST Changes → UI Reflow**  
  If the AST is altered (e.g., when loading a project), the Masonry layout automatically adjusts.

### 2.2 Shared Error & Warning States
- If the AST flags a missing argument or a type mismatch, the associated brick in the UI is highlighted in red.  
- Invalid UI actions that violate the AST’s rules (like plugging a string where a number is required) get rejected.

### 2.3 Common Data Model
- **Node Metadata**  
  Each UI brick has a matching AST node, and both remain in sync.  
- **Parameter Mapping**  
  Parameters in the visual blocks connect directly to the AST’s node attributes.

--------------------------------------------------------------------------------
## 3. Execution Engine Requirements

### 3.1 Basic Interpreter Flow
1. **Initialization**  
   - Finds all top-level “Start” nodes in the AST.  
   - Prepares separate threads or processes for each of those “Start” nodes.  
2. **Node Traversal**  
   - Executes statements one by one, following child links in the AST.  
   - Handles looping or branching in block nodes (e.g., “repeat,” “if”).  
3. **Data Handling**  
   - Pulls arguments from ArgumentNodes.  
   - Resolves ExpressionNodes to get their computed values (numbers, strings, etc.).

### 3.2 Concurrency & Parallel Execution
- Each top-level sub-tree in the AST can run independently.  
- Shared variables and other resources must be managed carefully to avoid conflicts.  
- The engine orchestrates multiple musical threads or parallel flows.

### 3.3 Debug & Breakpoints
- When debugging is enabled, the engine can stop at breakpoints or after each statement, depending on the user’s settings.  
- A real-time highlight on the UI brick shows which node is currently running.  
- A Variable Inspector shows variable values at each pause.

--------------------------------------------------------------------------------
## 4. Flow of an Edit→Run Cycle

1. **User Edits UI**  
   - Drags bricks around in the workspace.  
   - The Masonry system snaps them in place, and the AST updates accordingly.
2. **User Clicks “Run”**  
   - The Execution Engine locates all “Start” nodes in the AST.  
   - Performs the steps required to generate music or graphics.  
   - The running bricks get highlighted in the UI.
3. **User Debugs & Observes**  
   - Sets breakpoints or uses step-through to watch how the program flows.  
   - Errors or logs are shown in the console, with visual aids in the workspace.
4. **User Refines or Rebuilds**  
   - Modifies the blocks if needed.  
   - AST is updated instantly, ready for another run.

--------------------------------------------------------------------------------
## 5. Combined Implementation & Testing

### 5.1 Integration Strategy
- The UI calls into specialized AST APIs (like `createNode` or `attachChild`) whenever a new brick is placed.  
- The Execution Engine reads the AST structure whenever the user presses “Run.”

### 5.2 Testing & Validation
- **Unit Tests**  
  - Check node add/remove operations, and typical user actions.  
- **Integration Tests**  
  - Verify complex block arrangements, concurrency, function calls, and UI highlights line up correctly.
- **Performance Benchmarks**  
  - Track efficiency in real-time editing.  
  - Optimize for large or deeply nested ASTs.

--------------------------------------------------------------------------------
## 6. Roadmap for Future Enhancements

1. **Advanced Execution Models**  
   - Event-based designs or real-time triggers for sub-blocks.  
2. **Shared Memory & Variables**  
   - More robust concurrency handling, possibly using locks or message-passing.  
3. **Expanded Debug Features**  
   - Conditional breakpoints, time-based stepping for music-driven scenarios, and additional profiling tools.

--------------------------------------------------------------------------------
## 7. Conclusion

Linking the Masonry-based Program Builder, the AST, and the Execution Engine ensures a user-friendly, yet powerful, platform for coding. By unifying visual editing, logic validation, and real-time execution, Music Blocks (v4) provides an engaging space for exploring music, graphics, and general programming concepts.