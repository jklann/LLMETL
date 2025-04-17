Objective: Develop two Python programs to manage a mapping between PATIENT\_NUM and a patient identifier, using the i2b2 PATIENT\_MAPPING table structure. See the attached CSV file defining the PATIENT\_MAPPING table and its expected values. 

## First Program
The first program will take an input CSV with potentially many columns and it will output a CSV of a PATIENT\_MAPPING table, mapping a user-specified identifier column to a unique PATIENT\_NUM.

It should also accept an optional existing PATIENT_MAPPING CSV table, to which it will add the patients in the input CSV, without duplicating existing PATIENT_NUMs. Mapping already in the existing PATIENT_MAPPING CSV table should not be added a second time.

It should also accept a parameter that will be put in the PATIENT\_IDE\_SOURCE field.

The program will also output a second CSV that looks like an OBSERVATION\_FACT table, with one fact per patient, in which the PATIENT\_NUM is the PATIENT\_NUM from the mapping table and the CONCEPT\_CD has the form: "PATIENT\_MAP:<Value of PATIENT\_IDE\_SOURCE>"

### Observation_fact columns

- The complete list of output columns for OBSERVATION_FACT is provided below. Those labeled PK should never be empty, and ‘@’ is a good default value. Others can have null values:

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


## Second Program

The second program should be a module that can be loaded in another Python program. It will load a PATIENT\_MAPPING CSV output by the first program and then functions will allow a lookup of PATIENT\_NUM from PATIENT\_IDE and vice-versa.

**[UPLOAD THE PATIENT\_MAPPING\_DEFINITION.CSV FILE IN THE PROMPT]**
