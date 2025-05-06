## Enhanced 3MF Export and File Writing Features for OpenSCAD

This section describes two new features added to OpenSCAD to improve its export and scripting capabilities:
1. **Enhanced 3MF Export with Color-Based Grouping and Logging**: Exports `MeshObject`s grouped by color with detailed color logging, ideal for multi-material rendering.
2. **Write Function for Text File Output**: Introduces a `write` function to output text to files during script execution, enabling logging and data export.

These features enhance OpenSCAD’s ability to handle complex models and generate external data, suitable for various 3D modeling workflows.

### 1. Enhanced 3MF Export with Color-Based Grouping and Logging

This feature enhances OpenSCAD's 3MF export functionality to group triangles by color within a `PolySet`, creating separate `MeshObject`s for each unique color, and logs the color information for verification. It ensures that models with multiple colored components are exported as distinct objects for multi-material rendering in tools like Rhinoceros.

#### Features
- **Color-Based Grouping**:
  - Groups triangles by their color index in a `PolySet`, creating a `Lib3MF::PMeshObject` for each unique color.
  - Exports objects with different colors as separate components in the 3MF file.
- **Color Logging**:
  - Logs RGB and alpha values for each `MeshObject` using `LOG` with `message_group::Echo`.
  - Outputs messages like: `ECHO: Exporting mesh object <number> with color: R=<red>, G=<green>, B=<blue>, A=<alpha>` for colored objects, or `ECHO: Exporting mesh object <number> with no color` for objects without color.
  - Logs are visible in OpenSCAD’s console or can be redirected to a file.

#### Use Case
- **Example Input**:
 
  ```openscad
  difference() {
      union() {
          color("red") cube(10);
          translate([0,0,10]) color("blue") cube(10);
      }
      translate([5,5,0]) cylinder(h=60, d=5, center=true);
  }
  ```
  
- **Output**:
  - Exports a 3MF file with two `MeshObject`s: one for `red` and one for `blue`.
  - Log output:

    ```
    ECHO: Exporting mesh object 1 with color: R=1.000000, G=0.000000, B=0.000000, A=1.000000
    ECHO: Exporting mesh object 2 with color: R=0.000000, G=0.000000, B=1.000000, A=1.000000
    ```

  - In Rhinoceros, the 3MF file imports as two separate objects, allowing material assignment based on colors.

#### Implementation
- **Modified Files**:
  - Updated 3MF export logic to support color-based grouping and logging.
- **Dependencies**: Requires `lib3mf` for 3MF file handling and `utils/printutils.h` for logging.
- **Compatibility**: Works with Rhinoceros 8 and other 3MF-compatible tools.

#### Limitations
- Requires colors to be specified in the script (e.g., via `color()`).
- May increase export time for complex models with many colors.
- Some 3MF importers may merge objects; verify with a 3MF viewer or Rhinoceros 8.

### 2. Write Function for Text File Output

This feature introduces a new built-in function `write(filename, text, append = true)` to OpenSCAD, allowing users to write text to files during script execution. It enables logging, data export, or generating external files, complementing the 3MF export feature for workflows requiring file-based output.

#### Features
- **Function Signature**: `write(filename, text, append = true)`
  - `filename`: String specifying the output file path.
  - `text`: String to write to the file.
  - `append`: Boolean (default `true`) to append or overwrite the file.
- Logs errors for invalid arguments or file access issues.
- Supports child geometries, similar to the `echo` function, integrating with the CSG tree.

#### Use Case
- **Example Input**:

  ```openscad
  write("output.txt", "Hello from OpenSCAD!");
  write("output.txt", "Another line.", true); // Appends
  write("log.txt", "Overwritten.", false);    // Overwrites
  ```

- **Output**:
  - `output.txt`:

    ```
    Hello from OpenSCAD!
    Another line.
    ```

  - `log.txt`:

    ```
    Overwritten.
    ```

- **Combined with 3MF Export**:

  ```openscad
  write("log.txt", "Exporting red object", true);
  color("red") cube(10);
  write("log.txt", "Exporting blue object", true);
  translate([0,0,10]) color("blue") cube(10);
  ```

  - Logs export steps to `log.txt` alongside console output from 3MF export.

#### Implementation
- **Modified Files**:
  - Added `write` function and supporting logic for file output.
  - Updated parser, lexer, and documentation to recognize `write` as a built-in function.
- **Dependencies**: Uses standard C++ file I/O.
- **Compatibility**: Integrates with OpenSCAD’s CSG tree and supports child geometries.

#### Limitations
- File paths are relative to the script’s directory; ensure write permissions.
- Errors (e.g., invalid paths) are logged but do not halt execution.
- Large text outputs may impact performance; test with extensive writes.

### Installation

1. **Clone OpenSCAD**:

  ```bash
  git clone https://github.com/Dante-U/openscad.git  
  cd openscad
  ```

2. **Apply Changes**:
  - Apply patches or pull requests for 3MF export and `write` function (see repository for details).

3. **Build**:

  ```bash
  mkdir build
  cd build
  cmake ..
  make
  ```

4. **Test**:
  - Run `./build/openscad`.
  - Load a script using `write` and 3MF export, render (`F6`), and export as 3MF.
  - Check console logs, output files (e.g., `log.txt`), and 3MF file in a compatible viewer.
