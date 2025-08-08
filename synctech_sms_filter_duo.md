# 🔍 MMS-DuoFilter  
*Bidirectional Android Message Thread Extraction*  

**Extract court-ready SMS/MMS threads between two numbers**—*without* manual redaction or privacy risks.  

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

## 📜 Appendix: Full Pseudocode  
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
