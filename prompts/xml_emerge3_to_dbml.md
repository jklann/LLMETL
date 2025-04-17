
# Prompt

Create a Python script that reads an XML file and converts it into a JSON file in the DBML format. The JSON should have the following structure:

```json
{
  "table": "TABLE_NAME",
  "required": true,
  "columns": [
    {
      "column": "COLUMN_NAME",
      "description": "COLUMN_DESCRIPTION",
      "data_type": "DATA_TYPE",
      "enumerations": [
      		{code:"ENUM_CODE_1", "value":"ENUM_VALUE_1" },
      		{code:"ENUM_CODE_2", "value":"ENUM_VALUE_2" }, ...],
      "multi_value_delimiter": "|"
    },
    ...
  ]
}
```

### Details:
1. **XML Structure**: The XML file contains a `<data_table>` element with multiple `<variable>` elements. Each `<variable>` has:
   - `<name>`: The name of the column.
   - `<description>`: A description of the column.
   - `<type>`: The data type of the column.
   - `<comment>`: (Optional) Additional comments for the column.
   - `<value>`: (Optional) Multiple `<value>` elements for enumerated types. Value can have an attribute called code.

2. **Output JSON Structure**:
   - `table`: Set to the value of the `id` attribute of the `<data_table>` element.
   - `required`: Set to `true`.
   - `columns`: An array of column definitions. Each column has:
     - `column`: The value of the `<name>` element.
     - `description`: The concatenated value of `<description>` and `<comment>` elements.
     - `data_type`: The first part of the `<type>` element value (before any comma). This should always be lowercase.
     - `enumerations`: An array of objects with `code` and `value` keys. `value` is the text values of all `<value>` elements (if any), and `code` is the value of the `code` attributes.
     - `multi_value_delimiter`: Set to `"|"` if there are enumerations.

3. **Edge Cases**:
   - If `<comment>` is missing, only use `<description>`.
   - If `<value>` elements are missing, do not include `enumerations` or `multi_value_delimiter` in the column definition.
   - If `enumerations` are present, `<data_type>` is `enumeration`
   - The code associated with an enumeration should be the end of the column name (everything after the final underscore to the end), followed by a colon, followed by the code in the XML. 
   
### Example Usage:
- **Input XML**: A sample XML structure as provided.
- **Output JSON**: The JSON should conform to the specified DBML structure.

### Sample Code:

```python
import xml.etree.ElementTree as ET
import json

def xml_to_dbml_json(xml_file):
    tree = ET.parse(xml_file)
    root = tree.getroot()
    
    dbml = {
        "table": root.get("id"),
        "required": True,
        "columns": []
    }

    for variable in root.findall(".//variable"):
        column = {
            "column": variable.find("name").text,
            "description": variable.find("description").text,
            "data_type": variable.find("type").text.split(",")[0]
        }

        if variable.find("comment") is not None:
            column["description"] += f" {variable.find('comment').text}"
        
        values = variable.findall("value")
        if values:
            column["data_type"] = "enumeration"
            column["enumerations"] = [value.text for value in values]
            column["multi_value_delimiter"] = "|"
        
        dbml["columns"].append(column)

    return dbml

xml_file_path = "path/to/your/xml_file.xml"
dbml_json = xml_to_dbml_json(xml_file_path)

output_file_path = "path/to/output/dbml.json"
with open(output_file_path, "w") as json_file:
    json.dump(dbml_json, json_file, indent=2)

print(f"DBML JSON file created successfully at {output_file_path}")
```
