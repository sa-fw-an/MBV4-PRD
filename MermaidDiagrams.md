<!--markdownlint-disable-->
## Node Types
```mermaid
graph TD
    ASTNode[Abstract ASTNode]
    BlockNode[BlockNode]
    StatementNode[StatementNode]
    ArgumentNode[ArgumentNode]
    
    ASTNode --> BlockNode
    ASTNode --> StatementNode
    ASTNode --> ArgumentNode
    
    BlockNode --> SequenceBlock[SequenceBlock]
    BlockNode --> LoopBlock[LoopBlock]
    BlockNode --> ConditionalBlock[ConditionalBlock]
    BlockNode --> StartBlock[StartBlock]
    BlockNode --> FunctionDefBlock[FunctionDefBlock]
    
    StatementNode --> PlayNoteStmt[PlayNoteStatement]
    StatementNode --> RestStmt[RestStatement]
    StatementNode --> SetInstrumentStmt[SetInstrumentStatement]
    StatementNode --> VariableAssignStmt[VariableAssignmentStatement]
    
    ArgumentNode --> DataNode[DataNode]
    ArgumentNode --> ExpressionNode[ExpressionNode]
    ArgumentNode --> VariableRefNode[VariableReferenceNode]
    
    DataNode --> NumberDataNode[NumberDataNode]
    DataNode --> StringDataNode[StringDataNode]
    DataNode --> BooleanDataNode[BooleanDataNode]
```

## Node Type Hierarchy
```mermaid
classDiagram
    class ASTNode {
        <<abstract>>
        +id: string
        +nodeType: NodeType
        +parent: ASTNode
        +validate(): ValidationResult
        +clone(): ASTNode
        +toJSON(): object
    }
    
    class BlockNode {
        <<abstract>>
        +children: ASTNode[]
        +addChild(child: ASTNode): ASTNode
        +removeChild(id: string): ASTNode
    }
    
    class StatementNode {
        <<abstract>>
        +parameters: Map<string, ArgumentNode>
        +setParameter(name: string, value: ArgumentNode): void
        +getParameter(name: string): ArgumentNode
    }
    
    class ArgumentNode {
        <<abstract>>
        +evaluate(context: ExecutionContext): any
    }
    
    ASTNode <|-- BlockNode
    ASTNode <|-- StatementNode
    ASTNode <|-- ArgumentNode
    
    BlockNode <|-- SequenceBlock
    BlockNode <|-- LoopBlock
    BlockNode <|-- StartBlock
    
    StatementNode <|-- PlayNoteStatement
    StatementNode <|-- RestStatement
    
    ArgumentNode <|-- NumberDataNode
    ArgumentNode <|-- ExpressionNode
```

## Node Metadata and Serialization
```mermaid
flowchart TD
    subgraph Serialization
        node[ASTNode] --> toJSON("node.toJSON()")
        toJSON --> jsonObj[JSON Object]
        jsonObj --> storage[Storage/File]
        
        storage --> jsonData[JSON Data]
        jsonData --> ASTSerializer("ASTSerializer.fromJSON")
        ASTSerializer --> factory[NodeFactory]
        factory --> reconstructed[Reconstructed AST]
    end
    
    subgraph Metadata
        NodeMetadata[NodeMetadata Interface] --> id[id: string]
        NodeMetadata --> position["position?: {x,y}"]
        NodeMetadata --> comments["comments?: string[]"]
        NodeMetadata --> colorOverride["colorOverride?: string"]
        
        node2[ASTNode] --> metadata["_metadata: NodeMetadata"]
        metadata --> serialization[Included in serialization]
    end
```
## AST Manipulation
```mermaid
flowchart TD
    subgraph AST
        ast[AST Class] --> rootNodes("rootNodes: ASTNode[]")
        ast --> nodesById("nodesById: Map<string,ASTNode>")
        
        manipulate[AST Manipulation] --> addRoot[addRootNode]
        manipulate --> removeRoot[removeRootNode]
        manipulate --> getById[getNodeById]
        
        addRoot --> register[registerNodeRecursive]
        removeRoot --> unregister[unregisterNodeRecursive]
        
        register --> idMap[Add to nodesById map]
        register --> scanChildren[Scan children]
        register --> scanParams[Scan parameters]
        
        unregister --> idMapRemove[Remove from nodesById map]
        unregister --> scanChildrenRemove[Scan children]
        unregister --> scanParamsRemove[Scan parameters]
    end
```
## Reference Management
```mermaid
flowchart TD
    subgraph References
        refMgr[ReferenceManager] --> ast[AST]
        refMgr --> funcDefs("functionDefs: Map<string,BlockNode>")
        refMgr --> funcCalls("functionCalls: Map<string,StatementNode[]>")
        
        build[buildReferenceIndex] --> scan[Scan all AST nodes]
        scan --> identifyDefs[Identify function definitions]
        scan --> identifyCalls[Identify function calls]
        
        identifyDefs --> mapDefs[Map function names to definition nodes]
        identifyCalls --> mapCalls[Map function names to call sites]
        
        validate[validateReferences] --> checkCalls[Check all calls have definitions]
        validate --> reportErrors[Report missing function errors]
    end
```

## Parser Implementation
```mermaid
flowchart TD
    start[Start Parsing] --> findStartBlocks[Find Start Blocks]
    findStartBlocks --> loopStartBlocks[For Each Start Block]
    loopStartBlocks --> createStartNode[Create Start Node]
    createStartNode --> parseContents[Parse Block Contents]
    parseContents --> addToAST[Add to AST]
    
    parseContents --> loopChildren[For Each Child Block]
    loopChildren --> checkType[Check Block Type]
    checkType -- Block --> createBlockNode[Create Block Node]
    createBlockNode --> parseBlockChildren[Parse Block Children]
    checkType -- Statement --> createStatementNode[Create Statement Node]
    createStatementNode --> parseParameters[Parse Parameters]
    
    parseParameters --> loopParams[For Each Parameter]
    loopParams --> parseArgument[Parse Argument]
    parseArgument -- Literal --> createDataNode[Create Data Node]
    parseArgument -- Expression --> createExpressionNode[Create Expression Node]
    
    parseBlockChildren --> loopChildren
    addToAST --> done[Parsing Complete]
```

## State Manager
```mermaid
classDiagram
    class ExecutionContext {
        -variables: Map<string, any>
        -parent: ExecutionContext
        -globalContext: ExecutionContext
        +setVariable(name, value, scope)
        +getVariable(name)
        +createChildContext()
    }
    
    ExecutionContext --> ExecutionContext : parent
    ExecutionContext --> ExecutionContext : globalContext
    
    note for ExecutionContext "Variables are looked up in this context first,then in parent contexts if not found"
```

## Interpreter
```mermaid
flowchart TD
    start[Start Execution] --> findEntryPoints[Find Entry Points]
    findEntryPoints --> createContext[Create Global Context]
    createContext --> executeThreads[Execute Each Thread]
    
    subgraph "Node Execution"
        executeNode[Execute Node] --> checkBreakpoint[Check for Breakpoint]
        checkBreakpoint -- Has Breakpoint --> pauseDebugger[Pause for Debugger]
        pauseDebugger --> resumeExecution[Resume After User Action]
        checkBreakpoint -- No Breakpoint --> nodeTypeCheck[Check Node Type]
        resumeExecution --> nodeTypeCheck
        
        nodeTypeCheck -- Block --> executeBlock[Execute Block]
        nodeTypeCheck -- Statement --> executeStatement[Execute Statement]
        nodeTypeCheck -- Argument --> evaluateArgument[Evaluate Argument]
        
        executeBlock --> blockTypeCheck[Check Block Type]
        blockTypeCheck -- Sequence --> executeSequence[Execute Children in Order]
        blockTypeCheck -- Loop --> executeLoop[Execute Children Multiple Times]
        blockTypeCheck -- Conditional --> evaluateCondition[Evaluate Condition]
        evaluateCondition -- True --> executeTrueBranch[Execute True Branch]
        evaluateCondition -- False --> executeFalseBranch[Execute False Branch]
        
        executeStatement --> evaluateParams[Evaluate Parameters]
        evaluateParams --> statementTypeCheck[Check Statement Type]
        statementTypeCheck -- PlayNote --> scheduleNote[Schedule Note]
        statementTypeCheck -- Rest --> scheduleRest[Schedule Rest]
        statementTypeCheck -- Assignment --> assignVariable[Set Variable]
    end
    
    executeThreads --> waitForCompletion[Wait for All Threads]
    waitForCompletion --> done[Execution Complete]
```

## Concurrency and Time Management
```mermaid
flowchart TD
    subgraph "Thread Management"
        spawnThread[Spawn Thread] --> createThread[Create Thread Entry]
        createThread --> executeFunction[Execute Thread Function]
        executeFunction -- Success --> markComplete[Mark Thread Complete]
        executeFunction -- Error --> markError[Mark Thread Error]
        
        waitForAll[Wait for All Threads] --> checkStatus[Check Thread Status]
        checkStatus -- All Complete --> collectResults[Collect Results]
        checkStatus -- Some Running --> waitMore[Wait More]
        waitMore --> checkStatus
    end
    
    subgraph "Time Scheduling"
        scheduleNote[Schedule Note] --> calculateTiming[Calculate Note Timing]
        calculateTiming --> createOscillator[Create Audio Oscillator]
        createOscillator --> createGain[Create Gain Node]
        createGain --> connectNodes[Connect Audio Nodes]
        connectNodes --> scheduleStart[Schedule Start Time]
        scheduleStart --> scheduleStop[Schedule Stop Time]
        scheduleStop --> addToEvents[Add to Scheduled Events]
        addToEvents --> advanceTime[Advance Time Offset]
    end
    
    subgraph "Synchronization"
        thread1[Thread 1] --> access[Access Shared Resource]
        thread2[Thread 2] --> access
        access --> checkLock[Check Resource Lock]
        checkLock -- Locked --> waitUnlock[Wait for Unlock]
        waitUnlock --> checkLock
        checkLock -- Unlocked --> acquireLock[Acquire Lock]
        acquireLock --> useResource[Use Resource]
        useResource --> releaseLock[Release Lock]
    end
```

## Debugging Support
```mermaid
stateDiagram-v2
    [*] --> Inactive: Create debugger
    
    Inactive --> Active: enable()
    Active --> Inactive: disable()
    
    Active --> Paused: Hit breakpoint
    Paused --> Active: continue()
    
    Paused --> StepOver: stepOver()
    StepOver --> Paused: Pause at next statement
    
    Paused --> StepInto: stepInto()
    StepInto --> Paused: Pause in function
    
    Paused --> StepOut: stepOut()
    StepOut --> Paused: Pause after function returns
    
    state Paused {
        [*] --> InspectingVariables
        InspectingVariables --> ModifyingVariables
        ModifyingVariables --> InspectingVariables
        InspectingVariables --> AddingWatches
        AddingWatches --> InspectingVariables
    }
```