# MMS-DuoFilter: Precise SMS/MMS Thread Extraction

## Overview

Android SMS/MMS backups are **massive** - containing thousands of messages, dozens of participants, and gigabytes of media. When you only need the **communication thread between two specific individuals**, manual processing is:

- Time-consuming (endless scrolling)
- Error-prone (missed messages)
- Risky (potential data leaks)

MMS-DuoFilter solves this with **one command** - producing court-ready XML containing only the MMS traffic exchanged between two specified numbers.

## Key Features

- **Precision filtering**: Isolates exact message threads between two parties
- **Court-ready output**: Clean XML with only relevant metadata
- **Privacy-focused**: Runs locally with no data transmission
- **Format-aware**: Handles E.164, national, and raw number formats
- **Downstream compatible**: Preserves original XML schema for toolchain integration

## Installation

```bash
# From PyPI
pip install mms-duofilter

# From source
pip install git+https://github.com/your-org/mms-duofilter.git
```

## Usage Examples

### Basic Command Line

```bash
mms-duofilter \
  full-backup.xml \
  filtered-output.xml \
  --number-a +15551234567 \
  --number-b +15559876543
```

### Configuration File Approach

Create `config.json`:
```json
{
  "number_a": "+15551234567",
  "number_b": "+15559876543"
}
```

Then run:
```bash
mms-duofilter input.xml output.xml --config config.json
```

## Use Cases

### Legal Applications
- Isolate conversations between attorneys and clients
- Produce evidentiary records for litigation

### Corporate Scenarios
- HR investigations of employee communications
- Compliance audits of specific message threads

### Personal Use
- Extract important family conversations before device migration
- Create clean archives of meaningful message histories

## Technical Implementation

### Filtering Logic

1. Processes standard Android SMS/MMS backup XML
2. Retains only `<mms>` elements where:
   - `msg_box` is 1 (received) or 2 (sent)
   - `address` matches either specified number
3. Preserves only essential attributes:
   - `m_id`
   - `address`
   - `msg_box`

### Algorithm Pseudocode

```python
def filter_mms(input_path, output_path, number_a, number_b):
    validate_input_file_exists(input_path)
    root = parse_xml(input_path)
    new_root = create_output_structure(root)
    
    for mms in root.find_all(".//mms"):
        if not valid_msg_box(mms): continue
        if not involves_target_numbers(mms, number_a, number_b): continue
        
        filtered = create_filtered_element(mms)
        new_root.append(filtered)
    
    write_output(output_path, new_root)
```

### Word of advice learn pseudocode!!!

```
Function Main:
    # Step 1 – Acquire parameters
    numberA   ← Read number A via CLI flag or environment variable
    numberB   ← Read number B via CLI flag or environment variable
    inputPath ← Read CLI positional argument 1
    outputPath ← Read CLI positional argument 2

    # Step 2 – Pre-conditions
    If FileExists(inputPath) = False then
        Print "Input file not found"
        Exit with error code 1
    End If

    # Step 3 – Execute filter
    Call FilterMMSBetweenNumbers(inputPath, outputPath, numberA, numberB)

    # Step 4 – Post-condition
    Print "Filtered XML written to", outputPath
End Function

Procedure FilterMMSBetweenNumbers(inputPath: Path,
                                  outputPath: Path,
                                  numberA: String,
                                  numberB: String):
    Create Directory(outputPath.parent) if not exists

    root ← XMLParse(inputPath).getRootElement()
    newRoot ← CreateElement(root.tag, root.attributes)

    For each mmsElement in root.query(".//mms"):
        msgBox ← mmsElement.getAttribute("msg_box")
        address ← mmsElement.getAttribute("address").strip()

        If msgBox not in {"1", "2"} then Continue
        If address not in {numberA, numberB} then Continue

        filteredElement ← CreateElement("mms")
        For each key in ["m_id", "address", "msg_box"]:
            value ← mmsElement.getAttribute(key)
            If value is not null then
                filteredElement.setAttribute(key, value)
            End If
        End For

        Append filteredElement to newRoot
    End For

    Write XML(newRoot) to outputPath with encoding "utf-8" and XML declaration
End Procedure
```
