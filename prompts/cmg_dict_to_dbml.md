**Prompt:**

Write a Python program that reads a CSV file and converts it to a JSON file based on a specific structure. The CSV file has the following columns: `Tables`, `Field`, `Data element description`, `Field type`, `Field Priority`, `Enumerations`, `Multi-value delimiter`, and `Examples`. The JSON file should have the following structure:

- The top-level JSON is an array of tables.
- Each table object has the following keys:

  - `table`: (string) The name of the table.
  - `required`: (boolean) Indicates if the table is required.
  - `columns`: (array) An array of column objects, each representing a column in the table.

Each column object has the following keys:

- `column`: (string) The name of the column.
- `primary_key`: (boolean, optional) Indicates if the column is a primary key. Default is false.
- `required`: (boolean, optional) Indicates if the column is required. Default is false.
- `description`: (string) A description of the column.
- `data_type`: (string) The data type of the column. Possible values include "string", "enumeration", "float", etc.
- `examples`: (array, optional) An array of example values for the column.
- `notes`: (string, optional) Additional notes about the column. If the data type is "enumeration", this can include mappings of enumeration values to their descriptions.
- `multi_value_delimiter`: (string, optional) The delimiter used for multi-value fields, if applicable.
- `enumerations`: (array, optional) An array of valid enumeration values if the data type is "enumeration".
- `references`: (string, optional) Indicates if the column references another table and column.

The program should handle cases where:

1. The `Field Priority` can be a text string or a number. If it is a number and equals 1, the column is primary and required.
2. The `Enumerations` can be separated by commas or newline characters.
3. The `Examples` can be quoted strings separated by spaces.

