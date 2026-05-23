# DLX Sudoku

A Sudoku game written in C++17 that uses Donald Knuth's DLX algorithm (Dancing Links) to solve puzzles by transforming them into an exact cover problem, based on Knuth's [paper](https://arxiv.org/pdf/cs/0011047). The GUI is built with Qt Widgets (Qt5 and Qt6) laid out in Qt Designer, and the project builds with CMake.

## Prerequisites

- CMake 3.5+
- Qt5 or Qt6 with the Widgets module
- A C++17-compatible compiler

## Installation

```bash
mkdir build && cd build
cmake ..
cmake --build .
```

## Usage

```bash
# macOS
open build/dlx_sudoku.app

# Linux / Windows
./build/dlx_sudoku
```
