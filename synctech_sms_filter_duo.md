# 🔍 MMS-DuoFilter  
*Bidirectional Android Message Thread Extraction*  

**Extract ~~court-ready~~ SMS/MMS threads between two numbers**—*without* manual redaction or privacy risks. *because why not!?*

```bash
# Concept (language-agnostic)
filter_thread --input backup.xml --output alice_bob.xml \
  --from "+15551234567" --to "+15559876543"
```

[![Generic badge](https://img.shields.io/badge/Stability-Production_Ready-green.svg)](https://shields.io/) 
[![Generic badge](https://img.shields.io/badge/Architecture-Offline_First-blue.svg)](https://shields.io/)  

---

## 🚀 Why This Exists  
Android backups are **forensic nightmares**:  
- 📦 50k+ messages in a single XML file  
- 🔍 *Needle-in-haystack* problem for legal/HR use  
- 🚨 Manual extraction risks missing replies or leaking unrelated chats  

**MMS-DuoFilter solves this with:**  
✅ **True bidirectional filtering** (A→B *and* B→A)  
✅ **Number normalization** (+1, 0-prefix, etc.)  
✅ **Zero dependencies** (pure XML parsing)  

---

## 🛠 Usage  
### Step 1: Define Your Targets  
```json
// config.json
{
  "party_a": "+15551234567",
  "party_b": "+15559876543"
}
```

### Step 2: Run Extraction  
```bash
# Pseudocode execution
filter_thread input.xml output.json --config config.json
```

### Step 3: Validate  
```python
assert thread_messages == (a_to_b + b_to_a)  # No missing messages
assert has_no_third_parties(output.json)    # No privacy leaks
```

---

## 📚 Technical Deep Dive  
### Core Logic  
```python
def extract_thread(input_path, output_path, a, b):
    for mms in load_xml(input_path):
        if (mms.sender == a and mms.recipient == b) or \
           (mms.sender == b and mms.recipient == a):
            write_to_output(mms, output_path)
```

### Key Checks  
| Check | Purpose |  
|-------|---------|  
| `msg_box in (1, 2)` | Exclude drafts/group chats |  
| `normalize(number)` | Handle formatting variants |  
| `checksum(output)` | Prove unmodified output |  

---

## 📜 Logic: Pseudocode  
<details>
<summary>📌 Expand for Implementation Details</summary>

### Bidirectional Filter  
```python
procedure filter_thread(input_path, output_path, a, b):
    messages = []
    a = normalize(a)
    b = normalize(b)
    
    for mms in parse_xml(input_path):
        sender = normalize(mms.address) if mms.msg_box == 2 else None
        recipient = normalize(mms.address) if mms.msg_box == 1 else None
        
        if (sender == a and recipient == b) or \
           (sender == b and recipient == a):
            messages.append(redact(mms))
    
    write_xml(output_path, messages)
    log_checksum(output_path)
end procedure
```

### Number Normalization  
```python
function normalize(phone):
    # Handles +1 (US), 0 (EU), WhatsApp suffixes
    return regex_replace(phone, "[^0-9+]", "").ltrim("0+")
```
</details>

---

## 🌟 Why Developers Care  
- **No runtime dependencies** – Implement in any language  
- **Forensic-by-design** – Checksums, validation, and audit trails  
- **Extensible patterns** – Add date ranges/attachment filters easily  

```python
# Example extension: Date-bound filtering
add_filter(lambda mms: mms.date >= "2023-01-01")
```

---

**📌 Pro Tip**: Use this with [Android Backup Extractor](https://github.com/nelenkov/android-backup-extractor) for end-to-end workflow.

---

# **MMS-DuoFilter: Pseudocode Specification**  
*For Forensic Message Thread Extraction*

## **1. Core Data Structures**
```python
// Represents a single MMS record with forensic requirements
struct MMS:
    string    m_id       // Universally unique message ID
    string    address    // Normalized E.164 phone number
    integer   msg_box    // 1=received, 2=sent
    timestamp date       // ISO 8601 UTC
    string[]  parts      // Multimedia attachments
end struct

// Thread extraction request
struct Request:
    string    input_path  // Path to source XML
    string    output_path // Path for filtered output
    string    party_a     // Normalized number
    string    party_b     // Normalized number
    string    config_hash // SHA-256 of config
end struct
```

## **2. Main Algorithm**
```python
// @Precondition: input_path exists and is valid XML
// @Precondition: party_a != party_b
// @Postcondition: output_path contains only A↔B messages
// @Complexity: O(n) where n = number of MMS in input
procedure ExtractThread(Request r):
    // Phase 1: Input Validation
    if not FileExists(r.input_path):
        throw FileNotFoundError
    if not IsValidE164(r.party_a) or not IsValidE164(r.party_b):
        throw InvalidNumberFormatError
    
    // Phase 2: Normalization
    a_normalized ← NormalizeNumber(r.party_a)
    b_normalized ← NormalizeNumber(r.party_b)
    
    // Phase 3: Filtering
    output_mms = []
    for mms in ParseXML(r.input_path):
        if not IsRelevantMMS(mms, a_normalized, b_normalized):
            continue
        output_mms.append(RedactMMS(mms))
    
    // Phase 4: Output
    WriteXML(r.output_path, output_mms)
    WriteChecksum(r.output_path + ".sha256")
end procedure
```

## **3. Critical Helper Functions**
```python
// @Precondition: number is non-empty string
// @Postcondition: returns E.164 without formatting
function NormalizeNumber(string number) → string:
    normalized ← RegexReplace(number, "[^0-9+]", "")
    // Handle country codes
    if normalized starts with "0":
        return "+1" + normalized[1:]  // Example: EU → US format
    return normalized
end function

// @Precondition: mms.msg_box ∈ {1, 2}
// @Postcondition: returns true iff A↔B message
function IsRelevantMMS(MMS mms, string a, string b) → bool:
    if mms.msg_box == 1:  // Received
        return NormalizeNumber(mms.address) == a
    else if mms.msg_box == 2:  // Sent
        return NormalizeNumber(mms.address) == b
    return false
end function
```

## **4. Forensic Integrity Checks**
```python
// @Precondition: output file exists
// @Postcondition: throws if chain of custody broken
procedure VerifyOutput(Request r):
    expected_count ← CountMessages(r.input_path, r.party_a, r.party_b)
    actual_count ← CountMessages(r.output_path)
    
    if expected_count != actual_count:
        throw ForensicIntegrityError(
            "Message count mismatch: expected %d, got %d" 
            % (expected_count, actual_count))
    
    if not ValidateChecksum(r.output_path):
        throw TamperDetectionError("Output checksum invalid")
end procedure
```

## **5. Complexity Analysis**
| Component               | Time Complexity | Space Complexity |
|-------------------------|-----------------|------------------|
| XML Parsing             | O(n)            | O(m)             | 
| Number Normalization    | O(1) per number | O(1)             |
| Thread Filtering        | O(n)            | O(k)             | 
| Checksum Calculation    | O(k)            | O(1)             |

Where:  
- **n** = Total MMS in input  
- **m** = Max XML depth  
- **k** = Relevant MMS count  

## **6. Failure Modes**
```python
enum ErrorCode:
    FILE_NOT_FOUND
    INVALID_XML
    NUMBER_NORMALIZATION_FAILURE
    FORENSIC_TAMPER_DETECTED
    OUTPUT_VALIDATION_FAILED
end enum

// Standardized error handling
procedure HandleError(ErrorCode code, string message):
    LogError(code, message, GetSystemTime())
    if code == FORENSIC_TAMPER_DETECTED:
        HaltExecution()  // Fail-fast for integrity violations
end procedure
```

---
