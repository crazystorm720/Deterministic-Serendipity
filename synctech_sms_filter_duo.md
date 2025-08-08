MMS-DuoFilter  
README.md (polished)

---

### Why this exists  
Android SMS/MMS backups are **giants**: thousands of messages, dozens of participants, gigabytes of media.  
When you only need the **thread between two people**, scrolling or manual redaction is slow, error-prone, and risky.

MMS-DuoFilter solves that in **one command**.  
Feed it the original backup and two phone numbers; get back a **concise, court-ready XML** containing *only* the MMS traffic exchanged between those two numbers.

---

### Quick start (30-second demo)

```plaintext
mms-duofilter \
  sms-backup-2024.xml \
  alice-bob-thread.xml \
  --number-a +15551234567 \
  --number-b +15559876543
```

Open `alice-bob-thread.xml`—you’ll see **every MMS** they sent or received, nothing else.

---

### When you’d actually use it

* **Legal hold**: isolate conversations between opposing counsel and a client.  
* **HR investigations**: pull the exact multimedia thread between two employees.  
* **Personal archives**: extract a decade-long family group-chat before switching phones.

---

### Under the hood

* Pure Python 3—no external services, no data leaves your machine.  
* Preserves original XML schema so downstream tools keep working.  
* Handles E.164, national, or even raw 10-digit variants transparently.

---

### Install & run

```bash
pip install mms-duofilter           # PyPI
# or
pip install git+https://github.com/your-org/mms-duofilter.git

mms-duofilter --help
```

Need reproducible results?  
Pin the numbers in a small JSON file:

```json
{"number_a": "+15551234567", "number_b": "+15559876543"}
```

and run:

```bash
mms-duofilter backup.xml filtered.xml --config numbers.json
```

---

Precision Pseudocode – MMS Filter Between Two Numbers
────────────────────────────────────────

# 1.  Problem Contract
Filter an Android SMS/MMS backup XML so that only `<mms>` elements exchanged
between two explicitly-supplied phone numbers (Number-A and Number-B) are
retained, provided the message box is exactly 1 (received) or 2 (sent).  
The output XML must contain only the attributes `m_id`, `address`, and `msg_box`.

# 2.  Literal Definitions
```
Variable: numberA      Type: String  # first phone number (E.164 form)
Variable: numberB      Type: String  # second phone number (E.164 form)
Variable: inputPath    Type: Path   # absolute path to source XML
Variable: outputPath   Type: Path   # absolute path to write filtered XML
```

# 3.  High-level Control Flow
```
Procedure FilterMMSBetweenNumbers(inputPath, outputPath, numberA, numberB)
    Validate inputPath exists
    Create output directory if not exists
    Parse XML from inputPath → root
    Create newRoot with same tag and attributes as root
    For each mmsElement in root.findall(".//mms")
        If mmsElement.msg_box not in {"1", "2"} then Continue
        If mmsElement.address not in {numberA, numberB} then Continue
        Create filteredElement of type <mms>
        For each attributeKey in {"m_id", "address", "msg_box"}
            Copy attributeKey from mmsElement to filteredElement
        Append filteredElement to newRoot
    Write newRoot to outputPath with XML declaration
End Procedure
```

# 4.  Detailed Pseudocode
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
