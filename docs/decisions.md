# Engineering Decisions

## 1. DLX (Dancing Links) vs. Backtracking with Constraint Propagation

**Decision**: Use Knuth's Algorithm X with Dancing Links to solve Sudoku.

**Alternatives considered**:
- Simple recursive backtracking: try each value 1-9 in each empty cell, check validity, recurse. Easy to implement but explores many invalid branches.
- Backtracking with constraint propagation (e.g., arc consistency, naked pairs): prunes the search space significantly and is the most common approach in competitive Sudoku solvers.
- SAT solver reduction: encode Sudoku as a boolean satisfiability problem and use an off-the-shelf SAT solver.

**Reasoning**: DLX reformulates Sudoku as an exact cover problem, which is a well-studied abstraction. The Dancing Links data structure provides O(1) cover and uncover operations, making backtracking extremely efficient. The approach is also general-purpose -- the same DLX implementation could solve other exact cover problems (e.g., pentomino tiling, N-queens) with only a different matrix encoding.

**Tradeoffs**: DLX requires building and maintaining a complex linked-list structure, which is harder to implement and debug than simple backtracking. The exact cover matrix (729x324) adds memory overhead. For 9x9 Sudoku specifically, simpler backtracking with constraint propagation is fast enough in practice, so the DLX approach is somewhat over-engineered for this puzzle size -- but it demonstrates the algorithm well.

## 2. Single-Class Architecture vs. Separated Solver and GUI

**Decision**: Keep all logic (GUI, board management, solver) in the `MainWindow` class.

**Alternatives considered**:
- Separate `SudokuSolver` class that takes a board as input and returns a solved board, with `MainWindow` only handling UI.
- Model-View-Controller pattern with a `Board` model, `Solver` service, and `MainWindow` view.

**Reasoning**: The project is a focused demonstration of the DLX algorithm, not a production application. The entire solver is under 600 lines of implementation code. Introducing additional classes and interfaces would add boilerplate (constructors, getters, dependency injection) without improving clarity for a project of this scale.

**Tradeoffs**: The solver cannot be unit-tested independently of the Qt GUI. The `MainWindow` class has multiple responsibilities (violating single-responsibility principle). Reusing the solver in a different context (CLI tool, web backend) would require extracting it. These are acceptable costs for a self-contained demo project.

## 3. Two-Phase Matrix Construction (Boolean Array Then Linked List)

**Decision**: First populate a `bool matrix[729][324]` array, then iterate it to build the Dancing Links structure.

**Alternatives considered**:
- Build the linked list directly while computing constraints, skipping the intermediate boolean array.
- Use a sparse representation from the start (e.g., store only the positions of 1s as coordinate pairs).

**Reasoning**: The boolean array serves as readable, debuggable documentation of the constraint encoding. The comments in `create_matrix_array()` explain which column ranges correspond to which constraints. Verifying correctness is straightforward: print the array and check that each row has exactly four 1s in the expected positions. Building the linked list in a second pass (`create_matrix_linkedlist()`) is a mechanical transformation that can be reasoned about independently.

**Tradeoffs**: The boolean array uses 729 x 324 = ~236K bytes of stack memory, which is negligible but unnecessary since each row has exactly four 1s out of 324 columns. The two-pass approach is slightly slower than direct construction, though the difference is immeasurable for this matrix size.

## 4. S-Heuristic (Smallest Column First) for Column Selection

**Decision**: When choosing which column to cover next in Algorithm X, always select the column with the fewest remaining nodes.

**Alternatives considered**:
- Choose the first (leftmost) uncovered column.
- Choose a random uncovered column.

**Reasoning**: This is the heuristic recommended by Knuth in his original paper. Selecting the column with the fewest nodes minimizes the branching factor at each level of the recursion tree, which dramatically reduces the number of nodes explored. For Sudoku, this means the algorithm naturally focuses on the most constrained cells first.

**Tradeoffs**: Finding the smallest column requires a linear scan of all remaining column headers on every recursive call. This is O(n) where n is the number of uncovered columns, but n decreases as columns are covered and the constant factor is small. The pruning benefit far outweighs the scan cost.

## 5. Puzzle Generation by Random Placement and Removal

**Decision**: Generate puzzles by placing 9 random values on an empty board, solving with DLX to get a complete valid grid, then removing a fixed number of cells based on difficulty.

**Alternatives considered**:
- Start from a known solved grid and apply random valid transformations (row/column swaps within bands, digit relabeling).
- Use a sophisticated generator that removes cells one at a time, checking after each removal that the puzzle still has a unique solution.

**Reasoning**: The random-place-and-solve approach is simple and leverages the existing solver. It produces varied puzzles quickly without additional algorithmic complexity.

**Tradeoffs**: The generated puzzles are not guaranteed to have a unique solution. Multiple valid completions may exist, especially at higher difficulties where many cells are removed. This is acceptable for casual play but would be unsuitable for competitive or published Sudoku puzzles. The difficulty model is also simplistic -- it only controls how many cells are removed, not the logical complexity of the solving path.
