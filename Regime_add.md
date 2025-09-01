# Best Practice for Adding Jurisdictions in DRR

## 1. Introduction
Over time, the DRR community has experimented with different methods of digitising jurisdictional requirements.  
In 2025, a major refactoring of the rules was completed to maximise re-use of common elements.  
This enables contributors to reduce duplication and improve maintainability when adding new jurisdictions.  

This document sets out the best practices for incorporating new jurisdictions into DRR, making use of the latest functionality in the RUNE DSL and shared common types.  

## 2. Type Structuring

### 2.1 Guideline
The latest RUNE DSL allows labels, ruleReferences, and regulatoryReferences to be attached directly to both simple and complex types.  

### 2.2 Rules
- ✅ Do attach regulatory metadata directly to types.  
- ✅ Do follow the standard order:  
  1. Labels  
  2. RegulatoryReferences  
  3. RuleReferences  

- ❌ Do not use empty ruleReferences to capture regulatoryReferences.  
- ❌ Do not rely on rule source (rule source will be deprecated).  

### 2.3 Examples

```
override attribute boolean (0..1)
        [label "Test Label"]
        [regulatoryReference Jurisdiction table "1" dataElement "13" field "Attribute"
            provision "Example Provision"]
        [ruleReference AttributeRule]
```

```
override attribute Type (0..1)
        [ruleReference empty]
```

```
override attribute ComplexType (0..1)
        [label for nestedAttribute "Test Label"]
        [regulatoryReference for nestedAttribute Jurisdiction table "2" dataElement "55" field "Nested Attribute"
            provision "Example Provision"]
        [ruleReference for nestedAttribute NestedAttributeRule]
```
---

## 3. Adding New Mappings for Jurisdictions

### 3.1 Guideline
All new jurisdiction-specific mappings should leverage the `CommonTransactionReport` and `CriticalDataElement` types introduced in the 2025 refactoring.  

### 3.2 Rules
- ✅ Do add attributes to `CommonTransactionReport` if they are required by more than one jurisdiction.  
- ✅ Do relax type restrictions at the common level if necessary, then reapply stricter type restrictions at the jurisdiction level.  
- ✅ Do reuse existing ruleReferences in the common namespace.  

- ❌ Do not duplicate logic that already exists in `CommonTransactionReport`.  

### 3.3 Examples
 
 ```
 type NewJuristionTransactionReport extends common.EMIRTransactionReport:
    [rootType]
    override exisitingAttribute Type (1..1)
        [label "Test Label"]
        [regulatoryReference Jurisdiction table "1" dataElement "13" field "Attribute"
            provision "Example Provision"]
        [ruleReference AttributeRule]
    newAttribute
        [label "Test Label"]
        [regulatoryReference Jurisdiction table "1" dataElement "13" field "Attribute"
            provision "Example Provision"]
        [ruleReference AttributeRule]
```
---

## 4. Writing Rules

### 4.1 Guideline
When adding rules for attributes already defined in `CommonTransactionReport`:  

- ✅ Make use of the existing common ruleReferences if applicable.  
- ✅ Clearly write code so jurisdiction specific divergences are easy to identify and maintain.  

- ❌ Do not write rule references from scratch if we have existing common ruleReferences.  

### 4.2 Examples
```
reporting rule JuristionSpecificRule from TransactionReportInstruction: <"Cleared">
    filter IsAllowableAction
    then extract
        if common.CommonRule = value
        then extract juristion
        else common.CommonRule
```

---

## 5. Reference Model: ESMA EMIR
The ESMA EMIR regime is a model implementation of these practices. It demonstrates:  

- ✅ Direct use of ruleReferences on types.  
- ✅ Rule namespaces free from unnecessary regulatory information.  
- ✅ Shared attributes defined centrally in `CommonTransactionReport`.  
- ✅ Rules all use common rules with divergences clear.  

👉 Future jurisdictions should use ESMA EMIR as the benchmark for implementation consistency.  
