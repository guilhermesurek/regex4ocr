# regex4ocr

A simple library to plug regular expression models (Document Regex Models) to parse your favorite OCR string output and extract important fields
from the OCR picture.

## Built with

* Python 3.7.1 (but the library supports any Python 3.x);
* PyYAML for yml parsing;
* Tox and Pytest for testing.

## Document Regexp Model (DRM)

The DRM are yml files which describes the desired documents with many regular expressions. Those are used to extract the document data in order to transform the OCR results into a structured Python dict.

An example is as follows:

```yml
identifiers:
  - id_regexp_1
  - id_regexp_2
  - id_regexp_3
fields:
  key1: regexp1
  key2: regexp2
options:
  lowercase: true
  remove_whitespace: false
  force_ascii: true
  replace:
    - ['replace_regex_1', 'substituion_str_1']
    - ['replace_regex_2', 'substituion_str_2']
table:
  header: table header regex
  line_start: table row beginning regex 
  footer: table footer regex
```

### DRM (Document Regexp Model) fields

In order for the OCR string of a picture to be parsed properly, a regexp expressions that model the given document picture must be set in a YML file. The following fields are allowed to model the OCR document data string:

```identifiers```: list of regexps that defines that this DRM should be used for the given OCR string. ALL regexp identifiers must match or the DRM will not be used for the given document. When using the ```regex4ocr.parse```function, a folder path to the folder which contains the yml DRMs is passed. Hence, if a DRM does not match the OCR document because not all the identifiers regexp were found in the OCR string, the next DRM in the folder will be tested until one is found.

```fields```: defines key and regexp pairs. The regexps will be matched against the OCR string in order to extract the desired data. The key names will be used in the final Python dictionary.
```options```: defines optional pre processing of the OCR string. Replaces in the OCR string can be performed, the OCR string can be lowercased, whitespace can be removed and non ascii characerts can be coerced to be closest ascii match, e.g, ```ç``` is cast to ```c```.

```table```: defines tabular data to be extract. Such data is very common on purchase receipts where the table header marks the beginning of the table such.
```header```: regexp which marks the header of the table data in the OCR string.
```line_start```: regexp that matches the beginning of EVERY new line of the table. The rows fields in the final dictionary requires this field.
```footer```: regexp that matches the end of the table data. This is used to stop trying to parse the table rows.

## Transform OCR images into structured data

This library allows one to convert the OCR result string to the following structured format:

```python
{
    "fields": {
        "user_defined_field_1": "result_1",
        "user_defined_field_2": "result_2",
    },
    "table": {
        "header": "table header",
        "all_rows": "row 1 result row 2 result",
        "rows": ["row 1 result", "row 2 result"],
        "footer": "table footer"
    }
}
```

## Installing and using

To install the library, just run pip install:

```bash
pip install regex4ocr
```

To use it, just import the library:

```python
import regex4ocr  # use the regex4ocr.parse function
from regex4ocr import parse  # import the parse function directly
```

### Parse function

This OCR string parsing library is based on one function: ```parse```. Use it as follows:

```python
from regex4ocr import parse

# ocr string is the result string generated by your OCR system (Google Vision, etc.)
# drms_folder_path is the os folder path to the folder which contains the yml DRM models
parse(ocr_string, drms_folder_path)
```

## Getting ready with local development

In a system with Python pip, install the dev requirements:

```bash
pip install -r requirements_dev.txt
```

To locally use the main parse function, create a Python module at the root of the project and import the parse function:

```python
import regex4ocr

regex4ocr.parse(ocr_result_string, folder_with_drms)
```

## Testing

Regex4ocr has been tested under Python 3.x environments with tox and pytest. In order to run such tests:

```bash
tox  # executes all tests for Python 3.0, 3.3, 3.5 and 3.7
```

To execute the tests for a single environment (faster testing):

```bash
tox -e 3.7
```

## Code Linting

This project uses the following linters:

* pylint;
* flake8;
* pep8.

Also, the code is formatted with the Black formatter (https://github.com/ambv/black). Some Black configuration can be found
at the ```pyproject.toml``` file.

## OCR and DRM example:

Given the following DRM which exists at a folder named ```drms```:

```yml
identifiers:
  - cupom fiscal
fields:
  cnpj: 'cnpj:\s*(\d{2}\.\d{3}\.\d{3}\/\d{4}-\d{2})'
  coo: 'coo:\s*(\d{6})'
  date: '\d{2}\/\d{2}\/\d{4}\s*\d{2}:\d{2}:\d{2}'
options:
  lowercase: true
  remove_whitespace: false
  force_ascii: true
  replace:
    - ['c00', 'coo']
    - ['c10', 'coo']
table:
  header: (item|iten)\s+codigo.*vl.*(?=\n)
  line_start: \n\d*.+\d+
  footer: total\s*r\$
```

And given the OCR string result from a picture (such as Google Vision, Tesseract OCR, etc.), import the
regex4ocr library and use its parse function in order to extract structured data (Python dict) from it:


```python
import regex4ocr  # import the library

# stores the OCR string result generated by a computer vision package (Google Vision, etc.)
ocr_result = """
cnpj: 11.123.456/0001-99
ie:111.111.111. 111
im: 123456-7
2570972078 17:54: ttccf 3045759 **** c00:047621
cupom fiscal
iten codigo descricao qid un vl unit r$ ) st vl item(r$)
17273 breit grossa -7mts" bunx373 ft 288 026
2 $17 pedra 1 (ht) 2unx84 694 f1
169 38g
003 515 cimento votoran todas as obras 50 kg
cred)
boun x 26.489 f1
794,676
total r$
1.247.07
cheque
1.247.09
troco r$
"""

# insert the path to a folder where the DRM (yml files) are stored
drms_folder_path = "./my_drms"

# generate structured data (JSON-like) from the unstructured OCR string
extracted_data = regex4ocr.parse(ocr_result, drms_folder_path)

# prints the result
print(extracted_data)
```

Here's the final result:

```python
{
    "fields": {
        "cnpj": "11.123.456/0001-99",
        "coo": "047621",
    },
    "table": {
        "header": "iten codigo descricao qid un vl unit r$ ) st vl item(r$)",
        "all_rows": "\n17273 breit grossa -7mts bunx373 ft 288 026\n2 $17 pedra 1 (ht) 2unx84 694 f1\n169 38g\n003 515 cimento votoran todas as obras 50 kg\ncred)\nboun x 26.489 f1\n794,676\n",
        "rows": ["17273 breit grossa -7mts bunx373 ft 288 026", "2 $17 pedra 1 (ht) 2unx84 694 f1", "169 38g", "003 515 cimento votoran todas as obras 50 kg cred) boun x 26.489 f1794,676"],
        "footer": "total r$"
    }
}
```