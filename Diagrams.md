<!--markdownlint-disable-->
## AST Structure Diagram
Where to add: In the Technical Specifications section, right after introducing the "Program Representation Architecture"

```mermaid
classDiagram
    class ASTNode {
        <<abstract>>
        +id: string
        +position: Point
        +parent: ASTNode
        +clone()
        +toString()
    }
    
    class BlockNode {
        +children: ASTNode[]
        +addChild(node: ASTNode)
        +removeChild(id: string)
    }
    
    class StatementNode {
        +execute()
    }
    
    class ArgumentNode {
        +evaluate()
    }
    
    class LoopBlockNode {
        +iterations: number
        +execute()
    }
    
    class ConditionalBlockNode {
        +condition: ArgumentNode
        +elseBlock: BlockNode
        +evaluate()
    }
    
    class NoteStatementNode {
        +pitch: string
        +duration: number
        +volume: number
        +execute()
    }
    
    class VariableStatementNode {
        +variableName: string
        +value: ArgumentNode
        +execute()
    }
    
    class MathExpressionNode {
        +operator: string
        +operands: ArgumentNode[]
        +evaluate()
    }
    
    class LiteralValueNode {
        +value: any
        +type: string
        +evaluate()
    }
    
    ASTNode <|-- BlockNode
    ASTNode <|-- StatementNode
    ASTNode <|-- ArgumentNode
    
    BlockNode <|-- LoopBlockNode
    BlockNode <|-- ConditionalBlockNode
    
    StatementNode <|-- NoteStatementNode
    StatementNode <|-- VariableStatementNode
    
    ArgumentNode <|-- MathExpressionNode
    ArgumentNode <|-- LiteralValueNode
```

## Program Engine Architecture Diagram
Where to add: At the beginning of the Implementation Plan section, before diving into components

```mermaid
graph TB
    subgraph "MB Engine"
        AST["AST:- 
        Stores program structure"]
        Parser["Parser:- 
        Traverses and validates AST"]
        StateManager["State Manager:- 
        Manages variables and execution state"]
        Interpreter["Interpreter:- 
        Executes program logic"]
        Scheduler["Scheduler:- 
        Handles time-based execution"]
        ThreadManager["Thread Manager:- 
        Manages concurrent execution paths"]
    end

    UILayer["UI Layer"] -.-> AST
    AudioSystem["Audio System"] -.-> Interpreter

    Parser -->|reads from| AST
    Interpreter -->|uses| Parser
    Interpreter -->|accesses/modifies| StateManager
    Interpreter -->|uses| Scheduler
    ThreadManager -->|coordinates| Interpreter
    ThreadManager -->|synchronizes| StateManager

    classDef component fill:#f9f,stroke:#333,stroke-width:2px;
    classDef external fill:#bbf,stroke:#33f,stroke-width:1px;
    
    class AST,Parser,StateManager,Interpreter,Scheduler,ThreadManager component;
    class UILayer,AudioSystem external;
```

## Execution Flow Diagram
Where to add: In the Dynamic Components section, after introducing all components

```mermaid
flowchart TD
    A([Program Loaded]) --> B[Parser validates and prepares AST]
    B --> C{Validation successful?}
    C -->|No| D[Report Errors to User]
    D --> E([End])
    
    C -->|Yes| F[Initialize State Manager]
    F --> G[Begin Execution]
    
    %% Parallel execution paths
    G --> H1[Main Thread]
    G --> H2[Music Thread]
    
    %% Main Thread path
    H1 --> I1[Process Control Blocks]
    I1 --> J1[Update Variables]
    J1 --> K1[Check Conditions]
    
    %% Music Thread path
    H2 --> I2[Schedule Notes]
    I2 --> J2[Play Sounds]
    J2 --> K2[Maintain Timing]
    
    %% Synchronization point
    K1 --> L[Synchronization Point]
    K2 --> L
    
    %% Decision for loop or exit
    L --> M{Program Complete?}
    M -->|No| G
    M -->|Yes| N[Cleanup Resources]
    N --> O([End])
    
    %% Styling
    classDef process fill:#a8d5ba,stroke:#178344,stroke-width:1px;
    classDef decision fill:#ffd78c,stroke:#d68f00,stroke-width:1px;
    classDef terminal fill:#d3d3d3,stroke:#333,stroke-width:1px,border-radius:10px;
    
    class A,E,O terminal;
    class B,D,F,G,H1,H2,I1,I2,J1,J2,K1,K2,L,N process;
    class C,M decision;
```

## Sample AST Node Type Definitions
Where to add: In the AST Structure Design section, after describing the node types

```typescript
/**
 * Music Blocks Program Engine - Abstract Syntax Tree
 * Core TypeScript definitions
 */

// Basic types and interfaces
export interface Position {
  x: number;
  y: number;
}

export type NodeId = string;

// Base AST Node - Abstract class for all nodes
export abstract class ASTNode {
  id: NodeId;
  position: Position;
  parent: ASTNode | null;
  
  constructor(id: NodeId, position: Position) {
    this.id = id;
    this.position = position;
    this.parent = null;
  }
  
  // Tree navigation methods
  getRoot(): ASTNode {
    return this.parent ? this.parent.getRoot() : this;
  }
  
  getAncestors(): ASTNode[] {
    const ancestors: ASTNode[] = [];
    let current = this.parent;
    
    while (current) {
      ancestors.push(current);
      current = current.parent;
    }
    
    return ancestors;
  }
  
  // Abstract methods that all nodes must implement
  abstract clone(): ASTNode;
  abstract accept<T>(visitor: NodeVisitor<T>): T;
}

// BlockNode - For control structures that contain other nodes
export class BlockNode extends ASTNode {
  children: ASTNode[];
  
  constructor(id: NodeId, position: Position) {
    super(id, position);
    this.children = [];
  }
  
  addChild(node: ASTNode): void {
    this.children.push(node);
    node.parent = this;
  }
  
  removeChild(id: NodeId): ASTNode | null {
    const index = this.children.findIndex(child => child.id === id);
    
    if (index !== -1) {
      const [removed] = this.children.splice(index, 1);
      removed.parent = null;
      return removed;
    }
    
    return null;
  }
  
  replaceChild(oldNode: ASTNode, newNode: ASTNode): boolean {
    const index = this.children.indexOf(oldNode);
    
    if (index !== -1) {
      this.children[index] = newNode;
      newNode.parent = this;
      oldNode.parent = null;
      return true;
    }
    
    return false;
  }
  
  clone(): BlockNode {
    const cloned = new BlockNode(this.id, { ...this.position });
    this.children.forEach(child => {
      const childClone = child.clone();
      cloned.addChild(childClone);
    });
    return cloned;
  }
  
  accept<T>(visitor: NodeVisitor<T>): T {
    return visitor.visitBlockNode(this);
  }
}

// StatementNode - For executable statements
export abstract class StatementNode extends ASTNode {
  constructor(id: NodeId, position: Position) {
    super(id, position);
  }
  
  // Abstract method that must be implemented by concrete statement nodes
  abstract execute(context: ExecutionContext): void;
}

// ArgumentNode - For expressions and values
export abstract class ArgumentNode extends ASTNode {
  constructor(id: NodeId, position: Position) {
    super(id, position);
  }
  
  // Abstract method that must be implemented by concrete argument nodes
  abstract evaluate(context: ExecutionContext): any;
}

// LoopBlockNode - A specialized block for repetition
export class LoopBlockNode extends BlockNode {
  iterations: ArgumentNode;
  
  constructor(id: NodeId, position: Position, iterations: ArgumentNode) {
    super(id, position);
    this.iterations = iterations;
    this.iterations.parent = this;
  }
  
  execute(context: ExecutionContext): void {
    const count = this.iterations.evaluate(context);
    
    for (let i = 0; i < count; i++) {
      context.setLoopVariable('index', i);
      
      for (const child of this.children) {
        if (child instanceof StatementNode) {
          child.execute(context);
        }
      }
    }
  }
  
  clone(): LoopBlockNode {
    const clonedIterations = this.iterations.clone() as ArgumentNode;
    const cloned = new LoopBlockNode(this.id, { ...this.position }, clonedIterations);
    
    this.children.forEach(child => {
      const childClone = child.clone();
      cloned.addChild(childClone);
    });
    
    return cloned;
  }
  
  accept<T>(visitor: NodeVisitor<T>): T {
    return visitor.visitLoopBlockNode(this);
  }
}

// NoteStatementNode - A statement that plays a musical note
export class NoteStatementNode extends StatementNode {
  pitch: ArgumentNode;
  duration: ArgumentNode;
  volume: ArgumentNode;
  
  constructor(
    id: NodeId, 
    position: Position, 
    pitch: ArgumentNode, 
    duration: ArgumentNode, 
    volume: ArgumentNode
  ) {
    super(id, position);
    this.pitch = pitch;
    this.duration = duration;
    this.volume = volume;
    
    // Set parent references
    this.pitch.parent = this;
    this.duration.parent = this;
    this.volume.parent = this;
  }
  
  execute(context: ExecutionContext): void {
    const pitchValue = this.pitch.evaluate(context);
    const durationValue = this.duration.evaluate(context);
    const volumeValue = this.volume.evaluate(context);
    
    // In a real implementation, this would invoke the audio system
    context.playNote(pitchValue, durationValue, volumeValue);
  }
  
  clone(): NoteStatementNode {
    return new NoteStatementNode(
      this.id,
      { ...this.position },
      this.pitch.clone() as ArgumentNode,
      this.duration.clone() as ArgumentNode,
      this.volume.clone() as ArgumentNode
    );
  }
  
  accept<T>(visitor: NodeVisitor<T>): T {
    return visitor.visitNoteStatementNode(this);
  }
}

// NumberLiteralNode - A literal number value
export class NumberLiteralNode extends ArgumentNode {
  value: number;
  
  constructor(id: NodeId, position: Position, value: number) {
    super(id, position);
    this.value = value;
  }
  
  evaluate(_context: ExecutionContext): number {
    return this.value;
  }
  
  clone(): NumberLiteralNode {
    return new NumberLiteralNode(this.id, { ...this.position }, this.value);
  }
  
  accept<T>(visitor: NodeVisitor<T>): T {
    return visitor.visitNumberLiteralNode(this);
  }
}

// Visitor interface for implementing the Visitor pattern
export interface NodeVisitor<T> {
  visitBlockNode(node: BlockNode): T;
  visitLoopBlockNode(node: LoopBlockNode): T;
  visitNoteStatementNode(node: NoteStatementNode): T;
  visitNumberLiteralNode(node: NumberLiteralNode): T;
  // Additional visit methods would be added for other node types
}

// Execution context interface for running the program
export interface ExecutionContext {
  getVariable(name: string): any;
  setVariable(name: string, value: any): void;
  setLoopVariable(name: string, value: any): void;
  playNote(pitch: number, duration: number, volume: number): void;
  // Additional methods for program execution would be defined here
}
```



## Concurrency Model Visualization
Where to add: In the Concurrency and Time Management section

```mermaid
sequenceDiagram
    participant MT as Main Thread
    participant SM as State Manager
    participant SCH as Scheduler
    participant MET as Melody Thread
    participant DT as Drum Thread
    participant AS as Audio System

    Note over MT, AS: Initialization Phase
    
    MT->>SM: Initialize state manager
    MT->>SCH: Initialize scheduler
    MT->>AS: Initialize audio system
    
    MT->>MET: Create and initialize thread
    activate MET
    MT->>DT: Create and initialize thread
    activate DT
    
    MT->>SM: Register threads
    SM-->>MT: Confirmation
    
    Note over MT, AS: Program Execution Begins
    
    par Parallel Thread Setup
        MET->>SM: Request melody variables
        SM-->>MET: Return melody data
        MET->>SCH: Register for timing events
        
        DT->>SM: Request drum patterns
        SM-->>DT: Return drum data
        DT->>SCH: Register for timing events
    end
    
    MT->>SCH: Start program execution
    SCH-->>MET: Begin execution signal
    SCH-->>DT: Begin execution signal
    
    Note over MT, AS: Resource Acquisition
    
    MET->>AS: Request exclusive audio channel
    AS-->>MET: Channel 1 allocated
    
    DT->>AS: Request exclusive audio channel
    AS-->>DT: Channel allocation error (limited resources)
    DT->>MT: Report resource conflict
    
    Note over MT, AS: Resource Conflict Resolution
    
    MT->>SM: Update resource allocation policy
    MT->>AS: Request shared channel for drum thread
    AS-->>MT: Shared channel allocated
    MT-->>DT: Use shared channel
    
    Note over MT, AS: Concurrent Execution With Synchronization Point
    
    SCH->>MET: Trigger measure start event
    SCH->>DT: Trigger measure start event
    
    par Concurrent Playback
        MET->>AS: Play note C4 (t=0.0s)
        MT->>SM: Update UI - highlight C4 note
        DT->>AS: Play kick drum (t=0.0s)
        
        MET->>MET: Wait 0.5s
        DT->>DT: Wait 0.5s
        
        MET->>AS: Play note E4 (t=0.5s)
        MT->>SM: Update UI - highlight E4 note
        DT->>AS: Play hi-hat (t=0.5s)
    end
    
    Note over MT, AS: Thread Synchronization
    
    MET->>SCH: Request synchronization at measure end
    DT->>SCH: Request synchronization at measure end
    
    MET->>MET: Complete melody phrase
    MET->>SCH: Signal ready for sync
    
    DT->>DT: Still completing drum pattern
    MET->>MET: Wait for sync point
    
    DT->>AS: Play final drum hit
    DT->>SCH: Signal ready for sync
    
    SCH->>MET: All threads synced, continue
    SCH->>DT: All threads synced, continue
    
    Note over MT, AS: Error Handling Across Threads
    
    MET->>AS: Play complex chord with effects
    AS-->>MET: Processing overload error
    MET->>MT: Report audio processing error
    
    MT->>SM: Log error state
    MT->>SM: Get fallback configuration
    SM-->>MT: Simplified audio parameters
    
    MT->>MET: Use simplified parameters
    MET->>AS: Play simplified chord
    AS-->>MET: Success
    
    Note over MT, AS: Time-based Scheduling
    
    SCH->>MET: Scheduled tempo change event
    SCH->>DT: Scheduled tempo change event
    
    MET->>SM: Update tempo variable
    DT->>SM: Update tempo variable
    
    Note over MT, AS: Program Termination
    
    MT->>SCH: Signal program end
    SCH->>MET: Terminate signal
    SCH->>DT: Terminate signal
    
    MET->>AS: Release audio resources
    DT->>AS: Release audio resources
    
    MET-->>MT: Thread terminated
    deactivate MET
    DT-->>MT: Thread terminated
    deactivate DT
    
    MT->>AS: Shutdown audio system
    MT->>SM: Save final program state
```
