# Benchmark Task Schema

Each benchmark task is a JSON object with the following fields:

```json
{
  "id": "unique_task_id",
  "difficulty": "basic|intermediate|advanced|expert",
  "category": "syntax|qsql|temporal|optimization|tick",
  "instruction": "Natural language description of the task",
  "context": "Optional: table schemas, sample data, or constraints",
  "solution": "Reference q code solution",
  "test_cases": [
    {
      "input": "Test input (if applicable)",
      "expected_output": "Expected output or assertion"
    }
  ],
  "metrics": {
    "correctness": "How to verify correctness (automated test or manual check)",
    "idiomatic_quality": "Criteria for idiomatic q code"
  }
}
```

## Difficulty Levels

- **Basic (20 tasks):** Variable assignment, list operations, simple functions
- **Intermediate (30 tasks):** Table manipulation, qSQL queries, joins
- **Advanced (30 tasks):** Temporal joins, tick architecture patterns, complex qSQL
- **Expert (20 tasks):** Performance optimization, memory management, advanced idioms

## Categories

- **syntax:** Basic q syntax and operators
- **qsql:** Table queries (select, update, delete, exec)
- **temporal:** Time-series operations (aj, asof, window joins)
- **optimization:** Performance tuning, vectorization, attributes
- **tick:** Tick architecture (TP, RDB, HDB, subscribers)

## Examples

### Basic Example
```json
{
  "id": "basic_001",
  "difficulty": "basic",
  "category": "syntax",
  "instruction": "Create a list of integers from 1 to 10",
  "context": "",
  "solution": "1+til 10",
  "test_cases": [
    {
      "input": "",
      "expected_output": "1 2 3 4 5 6 7 8 9 10"
    }
  ],
  "metrics": {
    "correctness": "Output matches expected list",
    "idiomatic_quality": "Uses til operator"
  }
}
```

### Intermediate Example
```json
{
  "id": "intermediate_001",
  "difficulty": "intermediate",
  "category": "qsql",
  "instruction": "Select all rows from a trades table where price is greater than 100",
  "context": "trades:([] sym:`AAPL`GOOG`MSFT; price:150 95 200; size:100 200 150)",
  "solution": "select from trades where price>100",
  "test_cases": [
    {
      "input": "count select from trades where price>100",
      "expected_output": "2"
    }
  ],
  "metrics": {
    "correctness": "Returns 2 rows (AAPL, MSFT)",
    "idiomatic_quality": "Uses standard select syntax"
  }
}
```

### Advanced Example
```json
{
  "id": "advanced_001",
  "difficulty": "advanced",
  "category": "temporal",
  "instruction": "Perform an asof join to match trades to quotes by symbol and time",
  "context": "trades:([] time:09:30:00 09:30:05 09:30:10; sym:`AAPL`AAPL`GOOG; price:150.5 150.7 2800.2)\nquotes:([] time:09:29:55 09:30:03 09:30:08; sym:`AAPL`AAPL`GOOG; bid:150.4 150.6 2800.0; ask:150.5 150.7 2800.3)",
  "solution": "aj[`sym`time;trades;quotes]",
  "test_cases": [
    {
      "input": "cols aj[`sym`time;trades;quotes]",
      "expected_output": "`time`sym`price`bid`ask"
    }
  ],
  "metrics": {
    "correctness": "Join produces correct matches",
    "idiomatic_quality": "Uses aj operator correctly"
  }
}
```

### Expert Example
```json
{
  "id": "expert_001",
  "difficulty": "expert",
  "category": "optimization",
  "instruction": "Optimize a query that calculates VWAP by symbol using attributes",
  "context": "trades:([] time:asc 1000000?09:30:00+til 23400; sym:1000000?`AAPL`GOOG`MSFT; price:1000000?100.0; size:1000000?1000)",
  "solution": "update `g#sym from trades; select vwap:size wavg price by sym from trades",
  "test_cases": [
    {
      "input": "Check that sym has grouped attribute",
      "expected_output": "meta[trades][`sym;`a] = `g"
    }
  ],
  "metrics": {
    "correctness": "Query produces correct VWAP",
    "idiomatic_quality": "Uses grouped attribute for performance"
  }
}
```
