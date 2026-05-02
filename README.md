# Vivek
Hybrid Inventory Manager is a console app combining C and C++. C handles binary file storage using structs to persist inventory data, while C++ manages the menu and display using classes and STL. It supports adding, viewing, updating, deleting (soft delete), and listing items with validation.
# Hybrid Inventory Manager

A console-based inventory system with a **C data layer** (binary file I/O) and a **C++ UI layer** (classes + STL).

---

## Project Structure

```
capstone/
├── include/
│   ├── inventory.h          # Item struct + C function declarations (extern "C")
│   └── InventoryManager.h   # C++ class declaration
├── src/
│   ├── inventory.c          # C backend: fread/fwrite/fseek binary storage
│   ├── InventoryManager.cpp # C++ wrapper: std::vector + std::sort
│   └── main.cpp             # Menu loop / entry point
├── Makefile
├── CMakeLists.txt
└── README.md
```

---

## Build & Run

### Using Make (recommended)

```bash
# Build
make

# Run
./inventory_manager

# Or build + run in one step
make run

# Clean all build artifacts and the data file
make clean
```

### Using CMake

```bash
mkdir build_cmake && cd build_cmake
cmake ..
make
./inventory_manager
```

> **Requirements**: GCC / G++ with C11 and C++17 support (any modern version ≥ 7).

---

## Menu Options

| Choice | Action               |
|--------|----------------------|
| `1`    | Add item             |
| `2`    | View item by ID      |
| `3`    | Update item          |
| `4`    | Delete item          |
| `5`    | List all (sort by ID)|
| `5n`   | List all (sort by Name)|
| `6`    | Exit                 |

---

## Input Validation Rules

- **ID**: must be a positive integer (≥ 1)
- **Quantity**: must be ≥ 0
- **Price**: must be ≥ 0.00
- **Name**: must be non-empty (leading/trailing whitespace stripped)

Invalid input shows an error and re-prompts — no crash.

---

## Architecture

### C Layer (`inventory.c` / `inventory.h`)
- `Item` struct with `id`, `name[40]`, `quantity`, `price`, `is_deleted`
- Binary file (`inventory.dat`): fixed-size records, `fseek` for O(1) slot access
- `add_item` – appends, rejects duplicate active IDs
- `get_item` – scans for matching non-deleted ID
- `update_item` – overwrites slot in-place with `fseek`
- `delete_item` – sets `is_deleted = 1` in-place (soft delete)
- `list_items` – fills caller buffer, skips deleted records

### C++ Layer (`InventoryManager.cpp`)
- `InventoryManager` class calls each C function
- `std::vector<Item>` holds the listing buffer
- `std::sort` with lambda sorts by `id` or `name` (strcmp)
- Input helpers use `std::cin` with validation loops

---

## Test Cases

- **TC-1 – Persistence**: Added items 1, 2, 3 → exited → re-launched → `List all` showed all three items correctly. ✅

- **TC-2 – Update persists**: Updated item 2's name and price → exited → re-launched → `View item 2` showed the new values. ✅

- **TC-3 – Soft delete**: Deleted item 3 → `List all` showed only items 1 and 2 → `View item 3` returned "not found". ✅

- **TC-4 – Duplicate ID rejected**: Tried to add a new item with ID 1 (already exists) → program printed failure message and did not overwrite. ✅

- **TC-5 – Invalid input recovery**: Entered `-5` for ID, then `abc` for quantity, then left name blank — program re-prompted each time without crashing, then accepted valid input. ✅

---

## Data File

`inventory.dat` is created in the **working directory** on first add. Each record is exactly `sizeof(Item)` bytes. Delete it to reset the inventory (`make clean` does this automatically).
