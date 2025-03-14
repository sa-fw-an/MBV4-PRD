<!-- markdownlint-disable -->
# Music Blocks (v4) – Engine Execution PRD

This document lays out how the Execution Engine in Music Blocks (v4) runs the compiled or interpreted AST and produces real-time outputs like music or graphics. It focuses on how to handle concurrency, timing, and feedback to keep the entire system responsive and intuitive.

--------------------------------------------------------------------------------
## 1. Introduction

In Music Blocks (v4), developers build programs visually, and the AST captures the underlying logic. The Execution Engine’s job is to:
- Take the AST and run it, whether there’s basic sequencing or multiple threads happening in parallel.
- Coordinate sounds, graphics, and any other outputs in real time.
- Integrate with debugging tools like breakpoints and variable inspection, so you can pause and explore what’s going on under the hood.

--------------------------------------------------------------------------------
## 2. Top-Level Responsibilities

1. **Initialization & Setup**  
   - Find each “Start” node in the AST and treat those as concurrent entry points.  
   - Set up data structures for variables, timers, audio, or anything else the engine needs.

2. **Execution Flow Control**  
   - Traverse each node in the AST in a systematic way (like depth-first, or triggered by events).  
   - Handle control mechanisms: loops, if-statements, concurrency.  
   - Manage timing or delays (for instance in music playback).

3. **Resource & State Management**  
   - Keep tabs on global and local variables for each process.  
   - Resolve conflicts if multiple threads share the same data.

4. **Real-Time Feedback**  
   - Send signals to the UI for debugging—like highlighting active bricks.  
   - Display alerts for runtime problems or logs in a console.

--------------------------------------------------------------------------------
## 3. High-Level Execution Architecture

### 3.1 Processes & Threads
- Each “Start” node spawns its own process or thread.
- These processes can be round-robined or prioritized (depending on settings).
- If variables are shared, the engine uses locks or another safe method to avoid race conditions.

### 3.2 Node Traversal
1. **Statement Nodes**  
   - Each statement is a single operation (e.g., play a note or draw a shape).  
   - Can either be instant or block until the action completes.

2. **Block Nodes**  
   - Used for loops (“repeat,” “while”) and conditionals (“if,” “else”).  
   - Control how nested statements or sub-blocks run.

3. **Argument & Expression Nodes**  
   - Resolve expressions, returning numbers, strings, and so on.  
   - May include function calls or nested evaluations.

### 3.3 Musical Timing & Scheduling
- The engine needs precise scheduling for notes.  
- Can base timing on tempo or custom intervals.  
- Runs multiple lines of music in parallel so they sync according to a global clock.

--------------------------------------------------------------------------------
## 4. Concurrency Model

### 4.1 Parallel Execution of Modules
- Each “Start” node or user-defined routine is its own module running simultaneously.  
- An internal clock or timer helps synchronize everything.

### 4.2 Shared Resource Handling
- If two processes share the same variables, the engine either:
  - Uses locks or semaphores to manage access, or  
  - Employs a message-passing approach if direct data sharing is too risky.  
- The user might see warnings if the engine detects a potential conflict.

### 4.3 Event Handling
- The engine can fire events (e.g., when a note ends or a loop completes).  
- Other threads can subscribe to these events and trigger new paths in the AST.

--------------------------------------------------------------------------------
## 5. Debug & Instrumentation

### 5.1 Breakpoints & Stepping
- Users can mark certain nodes as breakpoints.  
- The engine halts at these points, letting you step through line by line or jump deeper into sub-routines.  
- The UI highlights the node in question and shows any relevant variables.

### 5.2 Variable Inspector
- When the engine pauses, you can inspect both global and local variables.  
- If concurrency is at play, variables might be locked to prevent timing issues.

### 5.3 Logging & Error Reporting
- Runtime errors (like invalid function calls) show up in logs and highlight the error’s location.  
- Depending on severity, the engine may keep running or stop altogether.

--------------------------------------------------------------------------------
## 6. Lifecycle & States

1. **Idle**  
   - Nothing is running. Users can freely edit their programs.
2. **Initialization**  
   - The engine reviews the AST, sets up concurrency, and preloads needed resources.
3. **Running**  
   - The program is live. The engine processes nodes, and the UI often locks certain changes.
4. **Debug Pause**  
   - The engine is on hold at a breakpoint. Inspect or step further as you like.
5. **Stopped**  
   - Execution concluded or was halted. Everything reverts to Idle.

--------------------------------------------------------------------------------
## 7. Implementation Phases

### 7.1 Phase 1: Core Single-Thread Execution
- Allow straightforward sequential runs of statements and blocks.  
- Basic music timing (delays, tempo-based scheduling).  
- Minimal debugging (logs).

### 7.2 Phase 2: Concurrency & Routines
- Launch multiple concurrent “Start” processes.  
- Introduce user-defined subroutines with local variables.  
- Start handling shared resources carefully.

### 7.3 Phase 3: Full Debug & Instrumentation
- Support breakpoints, step-through, and variable inspections.  
- Better error logs and real-time UI linking.  
- Fine-tune concurrency for solid multi-thread audio playback.

### 7.4 Phase 4: Optimization & Advanced Features
- Improve scheduling to handle complex programs at scale.  
- Add features for event-driven or reactive programming.  
- Expand concurrency support (thread pools, advanced messaging).

--------------------------------------------------------------------------------
## 8. Integration Testing

1. **Unit Tests**  
   - Verify correct operation of statements, blocks, and concurrency bits.  
   - Check timing logic for music playback.
2. **Integration Tests**  
   - Combine AST creation, scheduling, and audio playback with multi-thread testing.  
   - Confirm debug stepping works across complex projects.
3. **Performance Benchmarks**  
   - Stress-test large ASTs or multiple parallel tracks.  
   - Ensure smooth audio with minimal delay.

--------------------------------------------------------------------------------
