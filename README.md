# DLX Sudoku

## Description
A Sudoku game written in C++ that utilizes Donald Knuth's DLX algorithm (Dancing Links) to solve puzzles by transforming them into an exact cover problem. This implementation is based on Knuth's paper, which can be found [here](https://arxiv.org/pdf/cs/0011047). The game features a GUI created with Qt.

![dlx_sudoku](https://github.com/jasonchen270/dlx_sudoku/blob/main/screenshots/dlx_sudoku.png?raw=true)

## Tech Stack

- **Language**: C++17
- **GUI**: Qt (supports Qt5 and Qt6) via Qt Widgets
- **Build**: CMake (minimum 3.5)
- **UI layout**: Qt Designer (`.ui` file with 81 `QLineEdit` widgets for cells)

## Setup and Build

### Prerequisites

- CMake 3.5+
- Qt5 or Qt6 with the Widgets module
- A C++17-compatible compiler

### Build Steps

```bash
mkdir build && cd build
cmake ..
cmake --build .
```

### Running

```bash
# macOS
open build/dlx_sudoku.app

# Linux / Windows
./build/dlx_sudoku
```
