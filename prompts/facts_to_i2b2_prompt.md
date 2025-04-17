**Prompt for Generating a Python Program to Transform CSV Data to i2b2
Format**

**Objective**: Output a Python program that transforms data from a CSV file into the i2b2 observation\_fact format. The transformation should be guided by another CSV file that defines an ontology describing how each column in the CSV should be interpreted and mapped to i2b2 columns.

**Requirements**:

1.  **CSV Ontology Specification**:

    - An i2b2 ontology is a hierarchical structure of terms that
      categorizes data elements within the i2b2 system. It dictates the
      "navigation path" for users querying data through the i2b2 web
      client.

    - Folders can contain other folders or leaf nodes, and leaf nodes
      represent specific data elements.

    - Ensure there are no spaces in the C\_FULLNAME; use underscores
      instead.

    - Load a CSV ontology file, which can be interpreted as the mappings
      between TSV columns and i2b2 observation\_fact columns. The CSV
      ontology file contains the following columns (plus others not
      listed hee):

      - C\_FULLNAME: The full path of the term in the ontology,
        representing its position in the hierarchy.  It is a set of
        short strings for each folder and leaf, separated by \\ and
        ending with a trailing \\ (e.g., HOUSE\KITCHEN\TABLE\PLATE\\.

      - C\_BASECODE: A code that links ontology terms to fact entries of
        actual events in the data (e.g., ICD10:J29.1).

      - VALUETYPE\_CD: The type of the value (N for numeric, T for text).

      - C\_NAME: The name of the term.

      - C\_VISUALATTRIBUTES: Indicates whether the term is a folder
        (begins with F) or a leaf node (begins with L).

2.  **Input data File Handling**:

    - The input data file is a CSV, so ensure the program
      correctly handles delimiters.

    - Column headers in the CSV are in snake\_case and need to be mapped
      to the ontology entries which might be in a different case (e.g.,
      Title Case).

    - Handle columns that might have missing or special values (e.g.,
      "NA", empty, or "0") that should not result in output rows.
      
    - Ensure that you're working with strings and handle NaN values appropriately.

3.  **i2b2 observation\_fact Output**:

    - Generate output rows for the i2b2 observation\_fact format. In general, the output will have a number of rows equal to the number of rows times number of columns, e.g., each field from the input may result in multiple output rows, excluding rows with specified special values.

    - Ensure primary key fields (ENCOUNTER\_NUM, CONCEPT\_CD, etc.) are
      never empty and are appropriately unique or defaulted.

    - Include all necessary fields in the output, some of which may not
      be present in the input and need default or null values.

4.  **Translating from the data file to observation\_fact**:

    - Match each column name to an ontology folder or leaf. You can
      do this by finding a path (C\_FULLNAME) in which the final element of the path matches the column name in the data file. For example, a column 'plate' would match with `House\KITCHEN\table\Plate\\` Note that the case might be different, so comparisons should be done after converting both inputs to snake case. For example, a column labeled “Fruit” would have a C\_FULLNAME ending in “\fruit\\.

    - For each value in that column:

      - If the ontology row you found is a folder (C\_VISUALATTRIBUTES
        begins with F), look up the leaf node row in the ontology. This
        is done by looking for all the immediate children of the node
        (e.g., the folder “\fruit\\ might have children like
        “\fruit\banana\\ in the C\_FULLNAME. Compare the TSV value to the
        C\_NAME of each leaf node that is a child of the folder. For
        example, a value “banana” in a column “fruit” might correspond
        to a C\_FULLNAME of “\fruit\banana\\ with a C\_NAME of “Banana”.
        In this case, set the output CONCEPT\_CD to the C\_BASECODE in the
        child ontology row. Remember to consider the edge case in which
        no matching leaf is found. In this case, output a warning and
        skip this value.

      - If the ontology row you found is a leaf (C\_VISUALATTRIBUTES
        begins with L), the value will be a text or numeric value. This
        can be determined dynamically for each data value. The nval\_num
        and tval\_char columns in the output should be filled in
        accordingly. If the value is numeric (e.g., nval_num is used), tval_char should be set to the letter 'E'. valtype_cd should be either the letters 'T' or 'N' depending on whether the value is text or numeric, respectively. The numeric or text nature
        of the value must be determined empirically. Keep in mind that
        the value could be an int or float so that should be checked
        before string comparison operations are used. The CONCEPT\_CD
        should match the C\_BASECODE of the row.

    - Special cases:

      - The input file will have a column representing the PATIENT\_NUM
        (output column). The Python program you create should take a parameter specifying the search term for this column. For example, a user might specify that  this column will contain “participant\_id”. For example,
        “get\_ready\_for\_the\_participant\_id”. It should be output as the
        PATIENT\_NUM for every output row created from that single input
        row. There is an additional complexity. PATIENT\_NUM must be an
        integer, but participant_id might be alphanumeric. In this case, use a PatientMapping class I have defined separately. Here is an example of its use:
`from pat_map import PatientMapping`
`mapping = PatientMapping('output/synthetic/gregor_patient_map.csv')`
`patient_num=mapping.get_patient_num(‘alphanumeric_identifier’))`

        

      - The input file might have a column representing the START\_DATE
        (output column). Any column with the word “date” in it should be
        chosen. For example, “this\_date\_is\_great”. It should be output
        as the START\_DATE for every output row created from that single
        input row. If no such input column is found, a default date
        should be chosen.

      - ENCOUNTER\_NUM should be the same unique number for all facts
        that occurred in the same encounter. In the absence of a column
        indicating this, please make each row of the TSV file one
        encounter. So, for example, if the file has six columns, then
        the six facts generated from that row will have the same
        encounter number.

      - There might be values of the form “\<code\_system\>:\<code\>”, such as “ICD10:J59.4”. In this case, the output value is not dependent on the column name or an ontology lookup. These entries
        refer to an external terminology. In the case, a fact should be
        made with the CONCEPT\_CD equal to the value, and nothing in
        TVAL\_CHAR or NVAL\_NUM.
        
      - If a value has the | character, this is a multi-value delimiter. Multiple facts should be generated for this value, one for each token separated by |.


		- **Coerce Non-String Values**: All input values (whether integers, floats, or strings) should be coerced to strings before any operations like splitting or trimming (e.g., `split('|')`, `strip()`).
		- Add this to your prompt under **Special Cases**:

		- **Handle Prefix-Stripped Codes**: If a cell value doesn’t match any `C_NAME` in the ontology, assume it could be a `C_BASECODE` without its prefix (the portion after the "|") Check if the value matches the portion of any `C_BASECODE` after the "|" and use the matched code if found.


5.  **Error Handling and Logging**:

    - Implement checks and error handling for parsing issues due to file
      format errors (e.g., unexpected EOF, inconsistent row lengths).
      

    - Provide informative error messages and logging to help diagnose
      issues with input data or processing logic.
      

**Implementation Notes**:

- Account for variations in data format and content, ensuring robustness
  in data handling and processing.

- Consider performance implications if working with large datasets,
  ensuring the program remains efficient.
  
- Underscores should be treated as spaces in string comparisons. Non-alphanumeric characters (like underscores and spaces) and converting them to lowercase before comparison.
 
- Except when otherwise noted, string comparisons should be case insensitive. (Make sure both strings being compared are lowercase.)

- Ensure you do not use deprecated Python methods, such as DataFrame.append()
 
**Expected Output**:

- A Python script capable of reading specified CSV files,
  processing the data according to the described rules and mappings, and
  outputting a correctly formatted i2b2 observation\_fact CSV file.
  
- The script should be callable via a function that takes the input filenames, patient identifier search string, and any other necessary parameters.
  
- Additionally, the script should output an error log, detailing any input data that it was unable to find a mapping for in the ontology.

- Documentation or comments within the code explaining the function of
  major sections and any important variables or logic.

**Output File Column Details**:

- The complete list of output columns is provided below. Those labeled
  PK should never be empty, and ‘@’ is a good default value. Others can
  have null values:

  - ENCOUNTER\_NUM (PK, int)

  - CONCEPT\_CD (PK, varchar(50))

  - PROVIDER\_ID (PK, varchar(50))

  - START\_DATE (PK, datetime)

  - PATIENT\_NUM (PK, int)

  - MODIFIER\_CD (PK, varchar(50))

  - INSTANCE\_NUM (PK, int)

  - VALTYPE\_CD (varchar(50))

  - TVAL\_CHAR (varchar(255))

  - NVAL\_NUM (decimal(18,5))

  - VALUEFLAG\_CD (varchar(50))

  - QUANTITY\_NUM (decimal(18,5))

  - UNITS\_CD (varchar(50))

  - END\_DATE (datetime)

  - LOCATION\_CD (varchar(50))

  - OBSERVATION\_BLOB (text)

  - CONFIDENCE\_NUM (decimal(18,5))

  - UPDATE\_DATE (datetime)

  - DOWNLOAD\_DATE (datetime)

  - IMPORT\_DATE (datetime)

  - SOURCESYSTEM\_CD (varchar(50))

  - UPLOAD\_ID (int)
