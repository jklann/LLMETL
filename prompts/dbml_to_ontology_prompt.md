**Objective:** Develop a Python program to automatically generate an
i2b2 ontology based on a structured data specification provided in JSON
format. The program should be able to interpret various data types,
relationships, and specific field characteristics from the JSON and
translate them into the components of an i2b2 ontology. I've included a
sample specification.

**i2b2 Ontology Overview:**

-   **Purpose:** i2b2 ontologies serve as the structured vocabulary and
    organization framework for clinical data. They define how data is
    grouped, related, and accessed within the i2b2 tool, forming the
    backbone of the query capabilities in i2b2.

-   **Definition:** An i2b2 ontology is a hierarchical structure of
    terms (folders and leaf nodes) that categorize data elements within
    the i2b2 system. It dictates the "navigation path" for users
    querying data through the i2b2 web client.

**Key Components of i2b2 Ontologies:**

-   **Folders and Nodes:** The ontology consists of folders (which can
    contain other folders or leaf nodes) and leaf nodes (which represent
    specific data elements).

-   **Hierarchy Levels (C\_HLEVEL):** Each entry in the ontology has a
    hierarchical level indicating its depth in the structure. This is
    usually the number of backslashes in the c\_fullname.

-   **Full Path (C\_FULLNAME):** A unique path for each element,
    representing its position in the hierarchy. It is a set of short
    strings for each folder and leaf, separated by \\ and ending with a
    trailing \\ . For example, a set of folders representing a plate in
    a house might look like HOUSE\KITCHEN\TABLE\PLATE\\ . There must
    never be spaces in the C\_FULLNAME; use underscores instead.

-   **Display Name (C\_NAME):** The name displayed in the i2b2 web
    client.

-   **Visual Attributes (C\_VISUALATTRIBUTES):** Codes that determine
    whether an item is a folder, leaf, or active/inactive. The two most
    important are LA, which should be used for all active leafs, and FA,
    which is for active folders.

-   **Base Code (C\_BASECODE):** A code that links ontology terms to
    fact entries of actual events in the data. This is optional for
    folders. Usually, these are of a format
    &lt;coding\_system&gt;:&lt;code&gt; - for example, ICD10:J29.1.

-   **Metadata XML (C\_METADATAXML):** Optional XML content that can
    define additional properties or querying aids for specific data
    elements, such as value ranges or specific formats. This is useful
    for situations where free text or number values are used.

**Requirements for the Program:**

1.  **Input Processing:** The program must read a JSON file (from the
    current working directory) containing the data specification,
    including details like field names, descriptions, data types, and
    any special attributes or relationships. The DBML JSON file follows the structure in the attached example. Optionally, it can have one table at the top level, or it can have an array of tables. In the single table case, the root node can be the table name. In the array-of-tables case, each table should be output as a folder under a parent node derived from the filename. Be sure to think carefully about the data fields that would change, especially C_FULLNAME and C_HLEVEL.
    
2.  **Ontology Generation:** Based on the input:

    -   Construct the ontology hierarchy. In the situation where a
        standard terminology is used and the tree needs to be separately
        imported (e.g., ICD-10), a placeholder row should be output that
        indicates the terminology required.

    -   Define each element's properties (path, name, level, visual
        attributes, etc.).

    -   Include specific handling for different data types (e.g.,
        enumerations, strings with multiple values).

3.  **Output:** The program should output the complete ontology
    structure in a format suitable for import into the i2b2 database,
    such as a series of SQL insert statements or a CSV file representing
    all the attributes of the ontology.

**Additional Features:**

-   Considerations for data integrity and consistency, ensuring that the
    ontology accurately reflects the structure and constraints of the
    input data.

-   Extensibility to accommodate changes in the data specification or
    enhancements in the ontology structure.

**Additional details**:

1.  **Ontology Construction**:

    -   Convert each field in the JSON into ontology nodes, categorizing
        them appropriately as folders or leaf nodes. Each field will
        translate into one row in the ontology output, except in the
        case of enumerations (and booleans, which are a special case of enumerations), which will output a row for each value of the enumeration. (See ‘enumerations’ below.)

    -   Apply specific visual attributes to nodes:

        -   **FA** for active folders.

        -   **LA** for active leaf nodes.

        -   **LH** for hidden leaf nodes (specifically for fields
            containing 'id' or 'description' in their names).

2.   **Enumeration Handling**:
	- Enumerations (and booleans) should be represented as multiple output rows, with a parent folder for the field and child nodes for each valid enumeration value.
	- The program should support enumerations provided as either simple strings or dictionaries with `code` and `value` fields:
	    - For enumerations specified as dictionaries with `code` and `value` fields (e.g., `{"code": "C99269", "value": "Case"}`):
	        - Use the `code` as the `C_BASECODE`.
	        - Use the `value` (in title case) as the `C_NAME`.
	    - For enumerations specified as simple strings, set `C_BASECODE` to `<field_name>:<enumeration_value>` in all-caps snake case, truncated as needed to keep the length under 50 characters.
	        - If the string begins with a recognized prefix (e.g., `MONDO:`, `HP:`), perform a BioPortal lookup to retrieve the label; otherwise, use the string directly for `C_BASECODE` and `C_NAME`.

3.  **Node Attributes**:

    -   **C\_NAME**: Use a concise label derived from each field name.
        It is preferred to have spaces between words and the first
        letter of each word capitalized.

    -   **C\_FULLNAME**: Generate hierarchical paths based on field
        relationships. The first element of the path should be derived
        from the JSON table name. Remember, there should never be spaces in this field, so in the event that a JSON source field has a space, it should be replaced with an underscore when being inserted into the fullname. See more detailed rules on generation of this field further down in this prompt.

    -   **C\_BASECODE**: Generate unique identifiers for each field,
        especially for enumerated values.

    -   **C\_COMMENT**, **C\_TOOLTIP**: Include detailed descriptions
        and any additional notes from the JSON.

4.  **Output**: Write the resulting ontology structure to a CSV file (in
    the current working directory) with detailed attributes for each
    node, ready for import into an i2b2 database.

5.  **Visual and Structural Rules**:

    -   Implement rules for visual attributes to manage the visibility
        of nodes based on specific keywords in field names.

    -   Ensure the hierarchical structure reflects the nesting and
        relationships defined in the JSON.

    -   There must be a root node with C\_HLEVEL=1, corresponding to the
        table name in the JSON.

6. **C_FULLNAME Generation**:
	- `C_FULLNAME` represents the full hierarchical path for each node in the ontology. It should always end with a trailing backslash (`\`).
	- The structure of `C_FULLNAME` is built as follows:
   		- Start with the parent node's path, passed as `parent_path`.
    	- Append the name of the current node, ensuring:
       		- For table-level nodes, use the table name, replacing spaces with underscores.
       		- For column-level nodes, use the field name, replacing spaces with underscores.
    	- For enumeration nodes:
       	 - If the enumeration is specified as a code/value dictionary, use only the `code` field from the dictionary in `C_FULLNAME`.
        	- If the enumeration is specified as a simple string, use the string itself (after replacing spaces with underscores).
    	- For all cases, ensure the resulting `C_FULLNAME` ends with a trailing backslash (`\`).

	**Examples**:

	- For a table named `Demographics`, the `C_FULLNAME` might look like:
  ```
  \ParentNode\Demographics\
  ```
  
	- For a field named `Gender` under the `Demographics` table:
  ```
  \ParentNode\Demographics\Gender\
  ```
  
	- For an enumeration value `Male` under the `Gender` field:
  		- If `Male` is specified as a simple string:
    ```
    \ParentNode\Demographics\Gender\Male\
    ```
  		- If `Male` is specified as `{"code": "M", "value": "Male"}`:
    ```
    \ParentNode\Demographics\Gender\M\
    ```

**Additional Information**:

-   Use today’s date for fields like **UPDATE\_DATE**,
    **DOWNLOAD\_DATE**, and **IMPORT\_DATE**.

-   Ensure error handling for missing or incomplete data fields in the
    JSON.

-   Comment the code adequately for maintainability and future
    modifications.

**Deliverable**:

-   A Python script capable of reading the JSON structure, processing
    the data according to the specifications above, and outputting a
    structured CSV file.

# Ontology column list

I am going to provide some information on additional columns that should
be included in the output CSV file. Please modify the Python program to
output the additional columns. Also, the python program should load the
JSON spec from a file, not from a hardcoded string.

-   C\_SYNONYM\_CD: Should be N

-   C\_TOTALNUM: Should be empty.

-   C\_METADATAXML: I will explain this later. For now, it should be
    empty.

-   C\_FACTTABLECOLUMN: Should be concept\_cd

-   C\_TABLENAME: should be concept\_dimension

-   C\_COLUMNNAME: should be concept\_path

-   C\_COLUMNDATATYPE: should be T

-   C\_OPERATOR: should be LIKE

-   C\_DIMCODE: should be the same as the value in C\_FULLNAME

-   C\_COMMENT: the comment from the JSON spec.

-   C\_TOOLTIP: a longer human-readable description. In this case you
    could use usually the c\_name and any additional description from
    the JSON spec.

-   M\_APPLIED\_PATH: should be @

-   UPDATE\_DATE: Date in the format of “dd-mmm-yy”

-   DOWNLOAD\_DATE: Date in the format of “dd-mmm-yy”

-   IMPORT\_DATE: Date in the format of “dd-mmm-yy”

-   SOURCESYSTEM\_CD: You can put in the name of your organization

-   VALUETYPE\_CD: should be empty

-   M\_EXCLUSION\_CD: should be empty

-   C\_PATH: Should be the path to the parent from the c\_fullname, e.g,
    truncate the final element of the path. The trailing backslash
    should be included.

-   C\_SYMBOL: Should be the final element of the path that was
    truncated from C\_PATH

## Refinement - Metadata XML:

I have a file of XML definitions such as those described in
<https://community.i2b2.org/wiki/display/DevForum/Metadata+XML+for+Medication+Modifiers>.
The file is called metadata.xml. I am pasting the high-level outline of the file at the end of this prompt. I want to load this file in the Python program, and whenever it creates a leaf node that is not an enumerated value, it should insert the appropriate XML snippet into the c_metadataxml field. The data type specified in the JSON will match the name of the JSON object. If it doesn't, the  “Text” type should be inserted. One exception - if the field name refers to an age,
“Age” should be inserted. This should be done in a modular way so that
it is easy to specify new XML types.

&lt;?xml version="1.0" encoding="UTF-8"?&gt;

&lt;MetadataCollection&gt;

&lt;ValueMetadata name="Age"&gt;

&lt;/ValueMetadata&gt;

&lt;ValueMetadata name="Text"&gt;

&lt;/ValueMetadata&gt;

&lt;/MetadataCollection&gt;

**Include dbml_sample.json in the prompt**