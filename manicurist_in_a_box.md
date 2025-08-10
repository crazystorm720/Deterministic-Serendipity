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

Would you like to add:
1. Schema diagram (Mermaid.js)
2. Example validation script
3. Change control procedures?
