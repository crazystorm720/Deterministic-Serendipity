Here's the **strictly object-layer README.md** for your manicurist YAML system:

```markdown
# Manicurist Object Repository

## Data Schema Specification

### 1. `polish.yml` (Physical Inventory)
```yaml
# Example Object
items:
  - identifier: "OPI-123-BB"  # Required
    name: "OPI Bubble Bath"    # Required
    type: "classic"            # Required
    quantity: 1                # Required (current physical count)
    metadata:                  # Optional
      purchase_date: "2023-11-01"
```

### 2. `service.yml` (Service Catalog)
```yaml
# Example Object
offerings:
  - code: "CM30"               # Required (unique)
    name: "Classic Manicure"   # Required
    base_price: 25.00          # Required
    requires:                  # Optional
      - "classic_polish"
```

### 3. `client.yml` (Preference Records)
```yaml
# Example Object
profiles:
  - client_id: "CL-001"        # Required
    restrictions:              # Optional
      - "acetone_allergy"
    preferences:               # Optional
      - "square_shape"
```

## Validation Rules

1. **Immutable Fields**:
   - `identifier` (polish)
   - `code` (service)
   - `client_id` (client)

2. **Required Fields**:
   ```bash
   # Validation command example
   yamllint -d relaxed *.yml
   ```

3. **No Business Logic**:
   - These files contain only state representations
   - No derived fields or calculations

## Maintenance Protocol

1. **Object Creation**:
   ```bash
   # Add new polish (example)
   echo "  - identifier: \"NEW-001\"" >> polish.yml
   echo "    name: \"New Color\"" >> polish.yml
   echo "    type: \"gel\"" >> polish.yml
   echo "    quantity: 1" >> polish.yml
   ```

2. **Object Updates**:
   - Only modify `quantity` or `metadata` in polish.yml
   - Never modify `code` or `base_price` after creation

3. **Object Deletion**:
   - Remove entire entries when retired
   - Never nullify required fields

## Versioning
- YAML 1.2 specification
- UTF-8 encoding required
- No trailing whitespace allowed

```

This README:
1. Treats the YAML files as pure data contracts
2. Documents schema without workflow assumptions
3. Provides validation requirements
4. Enforces object-layer purity

---

Here’s a **strict object-definition table** for your manicurist YAML system, treating each entity as a discrete data container with no workflow contamination:

---

### **Manicurist Object Schema**  

| Object File  | Required Fields               | Optional Fields       | Immutable Fields      | Data Type Rules                  |
|--------------|-------------------------------|-----------------------|-----------------------|----------------------------------|
| **polish.yml** | `identifier` (string)<br>`name` (string)<br>`type` ("classic"\|"gel"\|"dip")<br>`quantity` (integer ≥0) | `metadata` (key-value pairs) | `identifier`          | `type` must match enum values    |
| **service.yml** | `code` (string)<br>`name` (string)<br>`base_price` (float ≥0) | `requires` (list of material categories) | `code`                | `base_price` must have 2 decimal places |
| **client.yml** | `client_id` (string)          | `restrictions` (list of allergy/condition tags)<br>`preferences` (list of style tags) | `client_id`           | Tags must be lowercase snake_case |

---

### **Field Definitions**  

#### **polish.yml**  
- `identifier`: Vendor SKU or custom ID (e.g., `"OPI-123-BB"`)  
- `quantity`: **Physical** bottles in kit (no projections)  
- `metadata`: Free-form (e.g., `purchase_date: "2023-11-01"`)  

#### **service.yml**  
- `code`: Machine-readable ID (e.g., `"CM30"` for 30min Classic Manicure)  
- `requires`: Material categories (e.g., `["classic_polish", "base_coat"]`)  

#### **client.yml**  
- `restrictions`: Safety tags only (e.g., `["acetone_allergy"]`)  
- `preferences`: Non-critical tags (e.g., `["square_shape", "no_file_shaping"]`)  

---

### **Object Interaction Rules**  

1. **No Cross-References**  
   - Never link objects (e.g., no `service` in `client.yml`)  
   - Never calculate fields (e.g., no `total_value` in `polish.yml`)  

2. **Validation Command**  
   ```bash
   # Verify schema compliance
   yamllint -d "{extends: relaxed, rules: {trailing-spaces: disable}}" *.yml
   ```

3. **Example Valid Objects**  
   ```yaml
   # polish.yml
   items:
     - identifier: "GEL-456-TC"
       name: "Gel Top Coat"
       type: "gel"
       quantity: 2

   # service.yml
   offerings:
     - code: "GR15"
       name: "Gel Removal"
       base_price: 10.00

   # client.yml
   profiles:
     - client_id: "CL-2023-001"
       restrictions: ["acetone_allergy"]
   ```

---

### **Change Control**  
| Operation    | Allowed On                | Example                  |
|-------------|---------------------------|--------------------------|
| **Create**  | Any object with all required fields | `Add new polish entry` |
| **Update**  | Only `quantity` and `metadata` (polish.yml)<br>Only `preferences` (client.yml) | `Update polish quantity from 2 → 1` |
| **Delete**  | Full objects only          | `Remove discontinued polish` |

---

This maintains **pure object-layer integrity** with:  
- No inventory management logic  
- No appointment scheduling traces  
- No derived/calculated fields  

Need to add an `equipment.yml` object type while preserving these constraints?

Would you like to add:
1. Schema diagram (Mermaid.js)
2. Example validation script
3. Change control procedures?
