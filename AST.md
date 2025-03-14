<!-- markdownlint-disable -->
# Music Blocks (v4) – AST (Abstract Syntax Tree) PRD

This document introduces the core concepts behind the Abstract Syntax Tree (AST) in Music Blocks (v4), focusing on how bricks in the graphical interface map to an internal tree structure. The AST is essentially the “blueprint” for all the logic in a Music Blocks program.

--------------------------------------------------------------------------------
## 1. Introduction

Music Blocks (v4) gives users a visual way to assemble programs with bricks. Under the hood, each brick arrangement is mapped to an AST so the logic is captured in a structured format. This AST:
- Acts as the definitive guide for how the program operates.
- Can be saved or loaded via JSON.
- Powers the interpreter or compiler to run the code.

--------------------------------------------------------------------------------
## 2. Core AST Requirements

### 2.1 Node Types
1. **BlockNode**  
   Represents a higher-level control structure or container (for loops, conditionals, etc.).  
   It includes a list/array of child statements or sub-blocks.

2. **StatementNode**  
   Reflects individual operations, like “play a note” or “draw a line.”  
   Can accept arguments (expressions, data).

3. **ArgumentNode**  
   Holds data or expressions that feed into statement or block parameters.  
   Subcategories include:  
   - **DataNode** – literal values (numbers, strings, booleans, and so on)  
   - **ExpressionNode** – operators or function calls that return values

### 2.2 Hierarchical Links
- Every node except the root has exactly one parent reference.  
- Block or Statement nodes can contain zero or more child nodes.  
- Argument nodes attach to “parameters” in their parent node.

--------------------------------------------------------------------------------
## 3. AST Construction & Synchronization

### 3.1 Incremental Updates
- When you snap a brick into place, the AST creates the corresponding node and attaches it to the right parent.  
- Pulling a brick (and any nested bricks) from the workspace removes the entire sub-tree.

### 3.2 Validation & Constraints
- The AST enforces logical rules, such as “repeat” needing a numerical expression.  
- Anything violating these rules is flagged both visually and in the AST.

### 3.3 Serialization & Deserialization
- The AST can be turned into JSON for saving projects.  
- Loading a JSON file recreates all nodes and syncs them back to the visual layout.  
- References for routines or concurrent processes must be resolved on reload.

--------------------------------------------------------------------------------
## 4. Execution Model

### 4.1 Traversal
- An interpreter or compiler systematically walks the AST.  
- Statements run in order, while blocks introduce branching (loops, conditionals).  
- If parallel work is needed, multiple sub-trees (like multiple “Start” nodes) run at the same time.

### 4.2 Concurrency & Routines
- Users can define routines, which appear as sub-trees that a “call” node can trigger.  
- Each “Start” node at the top level is treated as a separate piece of execution.

### 4.3 Debugging Hooks
- Users can place breakpoints on AST nodes.  
- Stepping through the code shows which node is currently active.

--------------------------------------------------------------------------------
## 5. Implementation Outline

### 5.1 Phase 1: Simple Node Structures
- Create minimal BlockNode, StatementNode, ArgumentNode classes.  
- Add/remove child operations.  
- Check basic data types (e.g., integer vs. string).

### 5.2 Phase 2: Node Utilities & Refined APIs
- Provide internal AST APIs (createNode, removeNode, reattachNode).  
- Expand data type checks (numeric, string, boolean).  
- Start incorporating partial error detection.

### 5.3 Phase 3: Serialization & Execution
- Implement JSON save/load.  
- Build an interpreter for straightforward, sequential logic.  
- Lay groundwork for concurrency (multiple sub-trees/threads).

### 5.4 Phase 4: Debugging & Polishing
- Add breakpoints, step-through, and watch variables.  
- Finalize support for parallelism (threads, message passing).  
- Thoroughly test big projects for performance.

--------------------------------------------------------------------------------
## 6. User-Facing Considerations

### 6.1 Consistency with Visual Blocks
- There is a one-to-one relationship between bricks in the UI and nodes in the AST.  
- Any errors in the AST show up similarly in the visual interface.

### 6.2 Versioning & Upgrades
- Keep the AST format adaptable for older and future project versions.  
- Small block-definition updates should avoid breaking earlier projects.

--------------------------------------------------------------------------------
