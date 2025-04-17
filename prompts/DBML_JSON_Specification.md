### DBML JSON Specification

This document provides a concise specification for the DBML (Database Markup Language) JSON format, which describes database schemas, tables, and columns. This format is structured to define table structures, column attributes, data types, and relationships between tables.

#### Top-Level Structure
A DBML JSON file consists of a collection of tables, each containing metadata and column definitions. The root structure contains the following fields:

- **`table`** (string): The name of the database table (required).
- **`required`** (boolean): Specifies whether this table is mandatory (optional, defaults to `false`).
- **`columns`** (array of objects): An array of column definitions for this table (required).

#### Column Object Structure
Each table contains multiple columns, which are represented as objects with the following fields:

1. **`column`** (string): The name of the column (required).
   
2. **`primary_key`** (boolean): Indicates if the column is a primary key (optional).
   
3. **`required`** (boolean): Specifies if the column must have a value (optional).

4. **`description`** (string): A human-readable explanation of the column's purpose (optional).

5. **`data_type`** (string): The type of data stored in the column (required). Common data types include:
   - `string`
   - `enumeration`
   - `float`
   - `integer`
   
6. **`enumerations`** (array of strings): For `enumeration` types, lists all allowable values (required if `data_type` is `enumeration`).

7. **`multi_value_delimiter`** (string): Specifies the delimiter for columns that store multiple values (optional).

8. **`examples`** (string or array of strings): Example values for this column (optional).

9. **`notes`** (string): Additional context or remarks about the column (optional).

10. **`references`** (string): Specifies a foreign key relationship in the format `> table_name.column_name` (optional).

#### Data Types
- **String**: A textual value, often used for identifiers or names.
- **Enumeration**: A fixed set of possible values that a column can take.
- **Float**: A floating-point number for numerical data.
- **Integer**: An integer for whole numbers.

#### Relationships and Foreign Keys
Columns can reference other tables using the `references` field, allowing for relationships such as foreign keys. The format is `> table_name.column_name`, denoting that the current column references another column from another table.

#### Example: Table Definition
```json
{
  "table": "participant",
  "required": true,
  "columns": [
    {
      "column": "participant_id",
      "primary_key": true,
      "required": true,
      "description": "Unique identifier for each participant",
      "data_type": "string",
      "examples": ["BCM_Subject_1", "BROAD_subj89054"]
    },
    {
      "column": "gregor_center",
      "required": true,
      "description": "Center associated with the participant",
      "data_type": "enumeration",
      "enumerations": ["BCM", "BROAD", "CNH_I", "UCI", "UW_CRDR", "GSS", "UW_DCC"]
    }
  ]
}
```

### Allowable Fields Summary
1. **`table`**: The table name (required).
2. **`required`**: Specifies if the table is mandatory (optional).
3. **`columns`**: An array of column objects (required).

#### Column Fields:
- **`column`**: Column name (required).
- **`primary_key`**: Whether the column is part of the primary key (optional).
- **`required`**: If the column must be filled (optional).
- **`description`**: Explanation of the column (optional).
- **`data_type`**: Specifies the type of data the column holds (required).
- **`enumerations`**: For enumerations, lists possible values (required for `enumeration` types).
- **`multi_value_delimiter`**: For columns containing multiple values, specifies the delimiter (optional).
- **`examples`**: Sample values for the column (optional).
- **`notes`**: Additional comments or details (optional).
- **`references`**: Foreign key reference to another column (optional).

### Example: Complete Table Definition
```json
{
  "table": "participant",
  "required": true,
  "columns": [
    {
      "column": "participant_id",
      "primary_key": true,
      "required": true,
      "description": "Unique identifier for each participant",
      "data_type": "string",
      "examples": ["BCM_Subject_1", "BROAD_subj89054"]
    },
    {
      "column": "gregor_center",
      "required": true,
      "description": "Center associated with the participant",
      "data_type": "enumeration",
      "enumerations": ["BCM", "BROAD", "CNH_I", "UCI", "UW_CRDR", "GSS", "UW_DCC"]
    },
    {
      "column": "family_id",
      "description": "Identifier for family",
      "data_type": "string",
      "references": "> family.family_id",
      "examples": ["CNH-I_fam89052", "UW_CRDR_Family_2"]
    },
    {
      "column": "sex",
      "required": true,
      "description": "Biological sex assigned at birth",
      "data_type": "enumeration",
      "enumerations": ["Female", "Male", "Unknown"]
    }
  ]
}
```

### Notes on Usage
- Tables and columns are extensible, allowing additional metadata to be included through `notes` and `description`.
- The schema supports complex relationships between tables through foreign key references.
- The specification allows for flexible data modeling by supporting enumerations, primary keys, optional fields, and multi-value delimiters.

This specification provides a clear structure for defining and understanding the DBML JSON format for database schemas.