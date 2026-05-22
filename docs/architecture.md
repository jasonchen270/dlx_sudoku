# Architecture

## Overview

DLX Sudoku is a single-window Qt desktop application. The entire codebase consists of one entry point (`main.cpp`), one class (`MainWindow`), one supporting struct (`Node`), and a Qt Designer UI file. All solver logic, board management, and GUI interaction are contained within `MainWindow`.

## Main Components

### Node (mainwindow.h)

A struct representing one element in the Dancing Links sparse matrix. Each node has:

- Four directional pointers: `left`, `right`, `up`, `down`
- A `col_header` pointer to the column header node
- `size` (used only on column headers to track the number of nodes in that column)
- `value`, `row_index`, `col_index` -- metadata mapping the node back to a Sudoku placement

Nodes provide `link_left_right`, `unlink_left_right`, `link_up_down`, and `unlink_up_down` methods. These are the "dancing" operations: unlinking temporarily removes a node from its neighbors, and relinking restores it, enabling efficient backtracking without copying data.

### MainWindow (mainwindow.h, mainwindow.cpp)

Responsible for both GUI and solver logic. Key responsibilities:

**GUI operations:**
- `style_cells()` -- applies styling to the 81 `QLineEdit` widgets
- `print_board()` -- writes `board[9][9]` values to the UI; highlights initial clues in a different color and makes them read-only
- `read_board()` -- reads current cell values from the UI back into `board[9][9]`
- `check_board()` -- validates whether the current board satisfies all Sudoku constraints

**Puzzle generation:**
- `generate_board()` -- places 9 random values, solves with DLX, then removes cells based on the selected difficulty level
- `save_initial_board()` -- copies `board` to `initial_board` to track which cells are clues

**Solver (DLX):**
- `create_matrix_array()` -- populates the 729x324 boolean matrix encoding the four Sudoku constraints
- `create_matrix_linkedlist()` -- converts the boolean matrix into a sparse doubly-linked list with column headers chained off a root sentinel node
- `cover(Node*)` -- removes a column and all rows intersecting it from the linked list
- `uncover(Node*)` -- restores a previously covered column and its rows
- `find_smallest_col_header()` -- returns the column header with the smallest `size` (S-heuristic)
- `find_solution()` -- recursive Algorithm X search; returns true when all columns are covered
- `cover_unsolved_board()` -- pre-covers columns corresponding to given clue cells
- `find_matching_node(value, row, col)` -- locates the node in the linked list matching a specific Sudoku placement
- `format_solution()` -- extracts solved values from the solution node vector back into `board[9][9]`

### UI Layout (mainwindow.ui)

A Qt Designer file defining:
- 81 `QLineEdit` widgets named `lineEdit_0` through `lineEdit_80`, arranged in a 9x9 grid
- Four buttons: New, Solve, Check, Clear
- A `QComboBox` (`difficultyBox`) for selecting difficulty level (Easy, Medium, Hard, Very Hard)
- Fixed window size of 1024x768

## Data Flow

The solver pipeline follows a clear sequence:

```
User clicks "Solve" (or "New" triggers internal solve)
         |
         v
    read_board()
    Reads QLineEdit values into board[9][9]
         |
         v
    create_matrix_array()
    Builds 729x324 boolean matrix encoding
    cell, row, column, and subgrid constraints
         |
         v
    create_matrix_linkedlist()
    Converts boolean matrix into sparse
    doubly-linked list with column headers
         |
         v
    cover_unsolved_board()
    Finds nodes matching pre-filled cells,
    adds them to solution vector, and covers
    their associated columns
         |
         v
    find_solution()
    Recursive Algorithm X:
    - Pick column with fewest nodes
    - Try each row, cover, recurse
    - Uncover on backtrack
         |
         v
    format_solution()
    Extracts (value, row, col) from solution
    nodes back into board[9][9]
         |
         v
    print_board()
    Writes board[9][9] to QLineEdit widgets
```

For puzzle generation (`on_newButton_clicked`), the flow starts by placing 9 random values, running the full solve pipeline to complete the grid, then removing a difficulty-dependent number of cells.

## Design Rationale

**Single class**: The project is small enough (under 600 lines of logic) that splitting into multiple classes would add indirection without meaningful benefit. The solver has no external consumers and is tightly coupled to the board representation.

**Two-phase matrix construction**: Building the boolean array first and then converting to a linked list separates the concern of "what are the constraints" from "how are they represented for DLX." The boolean array step includes detailed comments mapping column ranges to constraint types, making the encoding easier to verify.

**Stateful board arrays**: Using `board[9][9]` and `initial_board[9][9]` as class members rather than passing them as parameters simplifies the method signatures and reflects the fact that there is exactly one puzzle active at a time.
