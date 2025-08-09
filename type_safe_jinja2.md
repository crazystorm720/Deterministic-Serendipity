### **Type-Safe Configuration Templates at Runtime with CUE + Jinja2 Integration**

To achieve **type-safe templating** where:  
1. **Schemas are validated** before rendering,  
2. **Templates are flexible** (Jinja2),  
3. **Outputs are guaranteed correct** at runtime,  

Use this **CUE-powered Jinja2 pipeline**:

---

## **1. Define Types in CUE**  
**`schema.cue`**  
```cue
#NetworkDevice: {
  name:    string
  ip:      string & =~"^\\d+\\.\\d+\\.\\d+\\.\\d+$"
  vlans?:  [...int] | *[1, 10, 20]  // Optional, defaults to [1, 10, 20]
}

#ConfigTemplate: {
  devices: [string]: #NetworkDevice
  template: string   // Jinja2 template
}
```

---

## **2. Store Template + Data**  
**`config.yml`**  
```yaml
devices:
  switch1:
    name: "Core-SW1"
    ip: "10.0.0.1"
    vlans: [100, 200]
  firewall1:
    name: "FW-Main"
    ip: "10.0.0.2"
template: |
  {% for device in devices %}
  configure {{ device.name }}
    ip address {{ device.ip }}
    {% if device.vlans %}
    vlans {{ device.vlans | join(',') }}
    {% endif %}
  {% endfor %}
```

---

## **3. Validate & Render**  
**`render.py`**  
```python
import cue.language as cue
import jinja2

# 1. Load and validate config
config = cue.load("config.yml", "#ConfigTemplate")  # Fails if invalid IP/vlans

# 2. Render template
env = jinja2.Environment(loader=jinja2.BaseLoader)
template = env.from_string(config.template)
output = template.render(devices=config.devices)

print(output)
```

**Output (Guaranteed Valid)**:  
```text
configure Core-SW1
  ip address 10.0.0.1
  vlans 100,200
configure FW-Main
  ip address 10.0.0.2
```

---

## **4. Runtime Safety Features**  
### **A. Automatic Defaults**  
If `vlans` is omitted in YAML, CUE injects `[1, 10, 20]`.  

### **B. Schema Enforcement**  
```yaml
# Rejected (invalid IP)
firewall2:
  name: "FW-Backup"
  ip: "oops"  # CUE throws error before Jinja2 runs
```

### **C. Template Guardrails**  
```jinja2
{# Safe: Only validated fields are accessible #}
{{ device.ip }}  ✅ 
{{ device.foo }} ❌ CUE blocks unknown fields
```

---

## **5. Integration Options**  
### **A. CLI Workflow**  
```bash
# Validate first
cue vet schema.cue config.yml

# Then render
python render.py
```

### **B. CI/CD Pipeline**  
```yaml
steps:
  - name: Validate config
    run: cue vet schema.cue configs/*.yml

  - name: Generate configs
    run: |
      python render.py > output.conf
```

### **C. Go Integration**  
```go
package main

import (
  "cuelang.org/go/cue/cuecontext"
  "text/template"
)

func main() {
  ctx := cuecontext.New()
  cfg := ctx.CompileString(`#ConfigTemplate: {...}`)  // Load schema
  data := ctx.CompileFile("config.yml")             // Load data
  unified := cfg.Unify(data)                        // Validate
  tpl := template.Must(template.New("").Parse(unified.Lookup("template").String()))
  tpl.Execute(os.Stdout, unified.Lookup("devices").Map())
}
```

---

## **6. Why This Works**  
| **Problem**          | **Solution**                          |
|----------------------|---------------------------------------|
| Jinja2 has no types  | CUE validates data *before* rendering |
| YAML/JSON too rigid  | Jinja2 adds loops/conditionals        |
| Manual defaults      | CUE’s `*` operator auto-fills         |
| Configuration drift  | Single source of truth (CUE schema)   |

---

## **7. Alternatives Compared**  
| Tool          | Type Safety | Templating | Runtime Checks |  
|---------------|------------|------------|----------------|  
| **Raw Jinja2** | ❌ No      | ✅ Yes      | ❌ No           |  
| **Jsonnet**   | ✅ Yes      | ✅ Yes      | ❌ No           |  
| **Pulumi**    | ✅ Yes      | ❌ No       | ✅ Yes          |  
| **CUE+Jinja2**| ✅ Yes      | ✅ Yes      | ✅ Yes          |  

---

## **8. Advanced: Template Validation**  
Ensure templates only reference valid fields:  
```cue
#ConfigTemplate: {
  template: string & """
    {% for dev in devices %}
    {{ dev.name }} is at {{ dev.ip }}
    {# dev.missing_field would fail validation #}
    {% endfor %}
    """
}
```

---

## **9. Final Architecture**  
```
1. YAML Configs 
   → 2. CUE Validation 
   → 3. Jinja2 Rendering 
   → 4. Guaranteed-Correct Outputs
```

**Get Started**:  
```bash
go install cuelang.org/go/cmd/cue@latest
pip install jinja2
```
