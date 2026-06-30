# Detailed Design Document: Project Terminal-Tetris
**Target Audience:** 2-Person Trainee Graduation Project
**Environment:** Operating System Terminal / Command Line (No external graphics libraries required)
**Scope:** Ultra-Simplified Tetris Clone (No Rotations, 3 Core Shapes, Character-Based Rendering)
## Document Outline
 * **1. Executive Summary & Terminal Game Story**
 * **2. Collaborative Architecture & Task Assignment Table**
 * **3. System Data Structures & Terminal Visual Guide**
 * **4. Task Details: Phase 1 (The Terminal Sandbox)**
   * *Task 1.1: Screen Management & Main Engine Loop*
   * *Task 1.2: Grid Definition & Character Renderer*
 * **5. Task Details: Phase 2 (The Falling & Control Engine)**
   * *Task 2.1: Gravity & Vertical Collision Logic*
   * *Task 2.2: Keyboard Listener & Side Boundaries*
 * **6. Task Details: Phase 3 (The Stacking & Game Rules)**
   * *Task 3.1: Stacking & Respawn Logic*
   * *Task 3.2: Row-Clearing (Zap) & Score System*
## 1. Executive Summary & Terminal Game Story
### The Game Story
> "Deep inside a dusty, forgotten supercomputer, the system memory blocks are disorganized and leaking data! Colorful data-bricks are falling uncontrollably from the processor down into the system grid. If they pile up to the top, the computer will overheat and crash.
> As the System Administrator, your mission is to guide these data-bricks safely down using your keyboard. By arranging them into perfectly solid, unbroken horizontal lines, you lock the data safely away—causing that row to **ZAP** out of existence, cooling the system down, and earning you corruption-clearance points. Keep the grid clean, protect the mainframe, and don't let the blocks reach the top!"
> 
### Technical Approach
This version runs entirely within the standard terminal output. Instead of dealing with pixels, coordinates, or windowing libraries, the game updates by clearing the terminal screen text and printing a fresh text matrix. The tasks are split cleanly by features so both trainees get hands-on experience with core logical structures, timing mechanics, and text-stream control.
## 2. Collaborative Architecture & Task Assignment Table
The architectural model breaks down into feature ownership, requiring both developers to write backend logic and terminal display routines.
| Task ID | Task Name | Primary Owner | System Component |
|---|---|---|---|
| **Task 1.1** | Screen Management & Main Engine Loop | **Trainee 2** | Controller (Loop & Screen Refresh) |
| **Task 1.2** | Grid Definition & Character Renderer | **Trainee 1** | View (Text Matrix Compiler) |
| **Task 1.3** | **Milestone 1: The Terminal Sandbox** | **Joint** | Core Text Display Verification |
| **Task 2.1** | Gravity & Vertical Collision Logic | **Trainee 1** | Model (Y-Axis Gravity Physics) |
| **Task 2.2** | Keyboard Listener & Side Boundaries | **Trainee 2** | Controller / Model (X-Axis Input) |
| **Task 2.3** | **Milestone 2: The Steerable Falling Block** | **Joint** | Input & Physics Synchronization |
| **Task 3.1** | Stacking & Respawn Logic | **Trainee 1** | Model (State Locking) |
| **Task 3.2** | Row-Clearing (Zap) & Score System | **Trainee 2** | Model / View (Scoring & UI Print) |
| **Task 3.3** | **Milestone 3: Complete Finished Game** | **Joint** | Edge Case Polishing & Sign-Off |
## 3. System Data Structures & Terminal Visual Guide
### 3.1 Text Character Mapping
 * Empty space cell: Rendered as two space characters (  ).
 * Active/Locked block cell: Rendered as two bracket characters ([]).
 * Side boundary walls: Rendered as pipe symbols (|).
 * Floor boundary: Rendered as dashes (--).
### 3.2 Visual Examples & ASCII Layouts
#### The Core Matrix Layout View
The backend array maps directly to text outputs. Below is how the 10 \times 20 grid array maps visually in the console frame:
```text
       ARRAY DATA STATE                     TERMINAL SCREEN OUTPUT
   
      Cols: 0 1 2 3 4 5 6 7 8 9
   Row 0:  [0,0,0,0,0,0,0,0,0,0]        |                    |  Score: 30
   Row 1:  [0,0,0,0,1,1,0,0,0,0]        |        []          |
   Row 2:  [0,0,0,1,1,1,1,0,0,0]        |      [][][]        |
   ...                                  |                    |
   Row 18: [0,0,0,0,0,0,0,0,1,1]        |                [][]|
   Row 19: [1,1,1,1,1,0,1,1,1,1]        |##########  ########|
          ----------------------        ----------------------

```
### 3.3 The 3 Core Non-Rotatable Shapes
Shapes are represented using flat nested list grids where 1 represents a block and 0 represents empty space.
```text
1. The Square (2x2 Matrix)    2. The Line (1x3 Matrix)      3. The L-Shape (2x2 Matrix)
      Array: [[1,1],               Array: [[1,1,1]]              Array: [[1,0],
              [1,1]]                                                     [1,1]]
      Visual: [] []                Visual: [] [] []              Visual: []
              [] []                                                      [] []

```
## 4. Task Details: Phase 1 (The Terminal Sandbox)
### Task 1.1: Screen Management & Main Engine Loop (Owner: Trainee 2)
 * **Description:** Set up the basic loop environment and the commands required to clear old frames from the terminal view.
 * **Technical Deliverables:**
   * Construct the continuous execution loop (while game_running:).
   * Implement an operating system call inside the loop to clear the terminal screen cleanly before printing a new frame (e.g., executing os.system('cls') on Windows or os.system('clear') on Linux/macOS).
   * Add a minor sleep/delay pause (e.g., 0.05 seconds) using system time libraries so the processor does not run at 100% usage capacity.
### Task 1.2: Grid Definition & Character Renderer (Owner: Trainee 1)
 * **Description:** Establish the tracking game board matrix and write the loop logic to cleanly draw it line-by-line using text streams.
 * **Technical Deliverables:**
   * Define the playfield board dataset as a static 2D array of size 10 \times 20 filled with 0 integers.
   * Write a drawing function that parses each row of the matrix. It should print a leading |, print [] for cells containing 1, print    (two blank spaces) for cells containing 0, and append a trailing | at the end of each row.
   * Print a bottom row border consisting of 22 dashes (----------------------) below the matrix loop.
> **Integration Milestone 1 (The Terminal Sandbox):** When code blocks are combined, running the script should reveal a clean, hollow box matrix inside the command terminal, stable and ready to accept dynamic piece components.
> 
## 5. Task Details: Phase 2 (The Falling & Control Engine)
### Task 2.1: Gravity & Vertical Collision Logic (Owner: Trainee 1)
 * **Description:** Introduce automatic downward piece movement and rules to handle hitting the floor bottom.
 * **Technical Deliverables:**
   * Manage variables tracking the active shape's current location (piece_x and piece_y).
   * Integrate a timer condition inside the loop: every 0.5 seconds, automatically increment piece_y by 1.
   * Write a validation method check_vertical_collision(). If piece_y plus the height of the shape matrix exceeds row index 19, or if any 1 in the shape matrix overlaps a cell marked with a value higher than 0 in the board array, trigger a collision signal.
```text
VERTICAL COLLISION VISUAL EXAMPLE:
   Row 18: |          |             |          |
   Row 19: |    []    |  ==> Hits   |####[]####| => Stop piece!
   Floor:  -----------   Bottom     -----------

```
### Task 2.2: Keyboard Listener & Side Boundaries (Owner: Trainee 2)
 * **Description:** Capture letter keystrokes to steer blocks horizontally without breaking terminal view margins.
 * **Technical Deliverables:**
   * Configure a non-blocking keyboard reader function. Map the A key to move left (piece_x -= 1) and the D key to move right (piece_x += 1).
   * Write a validation method check_horizontal_collision(). If a user presses A, ensure piece_x does not drop below column index 0. If a user presses D, ensure piece_x plus the width of the shape does not exceed column index 9.
   * If a move violates these rules or hits an existing block, intercept the action and hold the position steady.
```text
HORIZONTAL BOUNDARY VISUAL EXAMPLE:
   |[]        |  Player presses 'A' (Left Key)
   |[]        |  Collision Logic detects Column < 0!
   |          |  RESULT: Ignore keypress, keep piece in place.

```
> **Integration Milestone 2 (The Steerable Falling Block):** Merging these mechanics enables an interactive demo: a block falls down from the top of the terminal screen, moves left/right when typing A or D, and halts safely when it contacts the bottom edge.
> 
## 6. Task Details: Phase 3 (The Stacking & Game Rules)
### Task 3.1: Stacking & Respawn Logic (Owner: Trainee 1)
 * **Description:** Turn moving pieces into permanent structures when they land, then loop back to spawn a new shape.
 * **Technical Deliverables:**
   * Write lock_piece_to_grid(). When Trainee 1's vertical collision flags a stop condition, write the shape's structural 1s directly into the permanent 10 \times 20 grid array.
   * Reset coordinates to spawn a new piece: piece_y = 0 and piece_x = 4. Randomly select the next layout pattern from the 3 core shapes.
   * *Defeat Condition:* If a newly generated piece instantly flags a collision right at row 0, print "GAME OVER" and close the program loop.
### Task 3.2: Row-Clearing (Zap) & Score System (Owner: Trainee 2)
 * **Description:** Track points and eliminate rows that have been packed with text blocks.
 * **Technical Deliverables:**
   * Write clear_full_lines(). Run a loop searching the rows from index 19 up to 0. If a row array contains entirely nonzero entries, purge that row index completely.
   * Insert a brand new row filled entirely with 0s at the top (index 0) of the board, pushing all remaining rows downward.
   * Maintain a local score counter. Award 10 points for every cleared line and use a standard terminal print() statement to display the value clearly right next to the game board layout.
```text
ROW-CLEARING ("ZAP") MECHANIC VISUAL:
   Before Zap (Row 19 full):            After Zap (Rows slide down):
   Row 18: |  [][]    |                 Row 18: |          |
   Row 19: |##########| <== Solid Row!  Row 19: |  [][]    | <== (Old Row 18)
   --------------------                 --------------------
   Score: 0                             Score: 10

```
> **Integration Milestone 3 (Complete Finished Game):** The final evaluation milestone. Both trainees combine all components. The terminal game loop handles block falling, user keyboard steering via keys A and D, block stacking upon landing, line clear removal operations, and real-time score metric generation directly inside the terminal interface.
> 
