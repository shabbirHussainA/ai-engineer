# Data Analyzer

A small, modular Python CLI for loading structured CSV/JSON data and computing statistics for numeric columns.

This project was built during my transition from production JavaScript/TypeScript engineering toward Python-based AI and backend engineering.

The objective was not to relearn programming fundamentals, but to become productive with Python while applying familiar software engineering principles such as modularity, type hints, reusable functions, exception handling, and testing.

---

## Problem

When working with structured data, even simple analysis tasks often involve repetitive steps:

* Reading data from files
* Handling different file formats
* Selecting a column
* Converting values into numeric data
* Ignoring values that cannot be converted
* Calculating basic statistics
* Reporting errors to the user

This project packages those steps into a small command-line application with separate modules for data ingestion, statistical processing, and CLI interaction.

---

## What It Does

The CLI accepts a CSV or JSON file and a column name, then calculates:

* Count
* Mean
* Median
* Minimum
* Maximum

Currently supported input formats:

* CSV
* JSON

JSON input is expected to contain a list of objects.

For numeric analysis, values that cannot be converted to `float` are ignored.

---

## Architecture

The project separates file handling, statistical processing, and the command-line interface.

```text
                         ┌──────────────────┐
                         │       CLI        │
                         │    cli.py        │
                         └────────┬─────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │    load_data()   │
                         │    reader.py     │
                         └────────┬─────────┘
                                  │
                    ┌─────────────┴─────────────┐
                    │                           │
                    ▼                           ▼
             ┌──────────────┐           ┌──────────────┐
             │  read_csv()  │           │ read_json()  │
             └──────────────┘           └──────────────┘
                    │                           │
                    └─────────────┬─────────────┘
                                  ▼
                         ┌──────────────────┐
                         │ compute_column_  │
                         │     stat()       │
                         │    stats.py      │
                         └────────┬─────────┘
                                  │
                                  ▼
                         ┌──────────────────┐
                         │ Numeric Column   │
                         │   Statistics     │
                         └──────────────────┘
```

### Module responsibilities

**`reader.py`**

Responsible for:

* Reading CSV files
* Reading JSON files
* Selecting the appropriate reader based on file extension
* Validating that JSON input is a list
* Reporting file and format errors

**`stats.py`**

Responsible for:

* Extracting numeric values from a column
* Converting values to `float`
* Computing count, mean, median, minimum, and maximum

**`cli.py`**

Responsible for:

* Parsing command-line arguments
* Loading the requested data
* Running the analysis
* Displaying results
* Handling application-level errors

---

## Project Structure

```text
data_analyzer/
│
├── analyzer/
│   ├── __init__.py
│   ├── cli.py
│   ├── reader.py
│   └── stats.py
│
├── tests/
│   ├── __init__.py
│   └── test_stats.py
│
├── __init__.py
└── .vscode/
    └── settings.json
```

The implementation intentionally keeps the project small. File ingestion and statistical logic are separated so that the core functionality can be used independently of the CLI.

---

## Technical Decisions

### Separate data ingestion from analysis

File parsing is kept in `reader.py`, while statistical operations live in `stats.py`.

This avoids coupling the statistical logic to a particular input format.

The analysis functions operate on:

```python
List[Dict[str, Any]]
```

rather than directly on files.

This means the statistical layer does not need to know whether the original data came from CSV or JSON.

---

### Use type hints

The project uses Python type annotations for function inputs and return values.

For example:

```python
def load_data(file_path: str) -> List[Dict[str, Any]]:
    ...
```

and:

```python
def compute_column_stat(
    data: List[Dict[str, Any]],
    column_name: str
) -> Dict[str, Union[float, int]]:
    ...
```

The goal is to make function contracts clearer while working in a dynamically typed language.

---

### Keep statistical logic reusable

The core statistical operation is exposed through:

```python
compute_column_stat()
```

rather than being implemented directly inside the CLI.

This keeps the CLI responsible for interaction while the calculation remains independently callable and testable.

---

### Handle non-numeric values

The numeric extraction step attempts to convert column values to `float`.

Values that cannot be converted are ignored.

For example, given:

```text
25
30
invalid
```

the numeric analysis operates on:

```text
25
30
```

This behavior is currently implemented in `extract_numeric_column()`.

---

## Error Handling

The application handles several failure cases.

### Missing files

A missing CSV or JSON file raises a `FileNotFoundError`.

### Unsupported file formats

Files other than `.csv` and `.json` result in a `ValueError`.

### Invalid JSON structure

JSON input must contain a list.

For example, an object such as:

```json
{
  "name": "example"
}
```

is rejected because the reader expects a list of objects.

### No numeric values

If the requested column contains no values that can be converted to numbers, `compute_column_stat()` raises a `ValueError`.

The CLI catches these errors and reports them to the user with a non-zero exit status.

---

## Testing

The repository uses `pytest`.

Current tests cover:

* Numeric column extraction
* Ignoring invalid numeric values
* Statistical calculation
* Count
* Mean
* Minimum
* Maximum

The test data includes both numeric values represented as strings and values that cannot be converted to numbers.

Example:

```python
[
    {"name": "Ali", "age": "25", "salary": 50000},
    {"name": "Sara", "age": 30, "salary": 60000},
    {"name": "Ahmed", "age": "invalid", "salary": 45000}
]
```

This tests an important part of the implementation: handling mixed-quality input rather than assuming every value is already numeric.

---

## Running the Application

The CLI expects two positional arguments:

```text
FILE
COLUMN
```

The intended usage is:

```bash
python -m analyzer.cli <file> <column>
```

For example:

```bash
python -m analyzer.cli data.csv salary
```

The application loads the file, analyzes the requested column, and prints the calculated statistics.

---

## Example Output

For a numeric column, the CLI formats the calculated values as:

```text
------------------------------
Results for 'salary'
COUNT: ...
MEAN: ...
MEDIAN: ...
MAX: ...
MIN: ...
------------------------------
```

The exact values depend on the input data.

---

## Running Tests

Install `pytest` in the project environment and run:

```bash
pytest
```

The tests are located in:

```text
tests/test_stats.py
```

---

## What This Project Demonstrates

Although the application itself is intentionally small, it demonstrates several Python engineering practices:

* Modular project organization
* Separation of concerns
* Reusable functions
* Type annotations
* CSV and JSON file handling
* Exception handling
* Command-line argument parsing
* Numeric data processing
* Pytest-based testing
* Working with Python collections and standard-library modules

More importantly, the project represents a shift from primarily JavaScript/TypeScript-based backend development toward becoming productive in Python.

---

## Limitations

This project is intentionally lightweight.

Current limitations include:

* Only CSV and JSON input are supported
* Statistical operations are limited to basic descriptive statistics
* Numeric conversion is based on `float()`
* Non-numeric values are ignored rather than reported separately
* There is no persistent storage
* There is no web/API interface
* There is no distributed processing
* There is no performance benchmarking
* There is no automated CI pipeline currently included in the repository

These are boundaries of the current implementation rather than claims about capabilities that have not been implemented.

---

## Future Improvements

Potential improvements include:

### Data quality reporting

Instead of silently ignoring values that cannot be converted to numbers, report:

* Total values
* Valid numeric values
* Invalid values
* Missing values

### Expanded statistics

Add additional descriptive statistics such as:

* Standard deviation
* Variance
* Percentiles

### Better input validation

Improve validation around:

* File extensions
* Column existence
* Empty datasets
* Malformed input

### Automated quality checks

Add development tooling such as:

* Type checking
* Linting
* Formatting
* Continuous integration

### More comprehensive testing

Expand tests around:

* CSV parsing
* JSON parsing
* Invalid JSON
* Missing files
* Unsupported formats
* Empty datasets
* Missing columns
* Mixed-quality data

---

## Learning Outcome

The main outcome of this project was not learning how to calculate an average.

It was becoming more productive in Python while applying an existing software engineering mindset.

Coming from JavaScript/TypeScript, the transition involved adapting familiar engineering concepts to Python's ecosystem:

```text
Existing Software Engineering Experience
                ↓
        Python Fundamentals
                ↓
       Python Project Structure
                ↓
       Type Hints & Exceptions
                ↓
      Testing with Pytest
                ↓
       Python for AI/Backend
```

This project is the first step in a broader AI Engineering transition:

```text
Production Full Stack / MERN
              ↓
     Python for AI/Backend
              ↓
       Machine Learning
              ↓
      LLM Engineering
              ↓
             RAG
              ↓
           Agents
              ↓
    Production AI Engineering
```

The goal is not to replace existing software engineering experience with AI tooling.

The goal is to combine strong software engineering fundamentals with increasingly sophisticated AI engineering capabilities.

---

## Status

**Week 1 — Python for AI Engineers**

The project represents the hands-on implementation component of the first stage of the transition toward Python-based AI and backend engineering.

---

## License

No license is currently included in the repository.
