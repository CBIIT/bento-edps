# bento-edps: Extended Definition Properties

This is the common repository for defining CRDC Extended Definition Properties
(EDPs) using Model Definition Format ([MDF](https://cbiit.github.io/bento-mdf/mdf.html)) files 
(located in [model-desc](/model-desc)).

EDPs allow for the creation of custom standardized permissible value (PV) sets that can be 
used across CRDC data commons models. These are created to supplement PV sets that are
maintained by [caDSR](https://cadsr.cancer.gov).

EDPs are defined here and processed by the Model Database ([MDB](https://github.com/CBIIT/bento-mdb)) automated update scripts to persist these lists in the MDB. EDP PV sets can be accessed for data validation using the EDP endpoints of the Simple Terminology Service ([STS](https://github.com/CBIIT/bento-sts-fastapi)) at https://sts.cancer.gov/v2.

# Working with EDPs

## How EDPs flow into the MDB

1. EDP definitions are committed to this repo, under [`model-desc`](/model-desc).
2. A weekly scheduled check in [bento-mdb](https://github.com/CBIIT/bento-mdb) polls this repo for changes.
3. When a change is detected, bento-mdb generates a changelog that creates or updates the EDP term, its value_set, and all PV terms in the MDB graph:

    `(edp:term)-[:specifies_value_set]->(vs:value_set)-[:has_term]->(pv:term)`

4. That changelog is applied to the CloudOne dev MDB, then to FNL.
5. Once in the MDB, EDP PV sets can be queried for data validation via the EDP endpoints of the Simple Terminology Service (STS) at https://sts.cancer.gov/v2.

## Repository layout

```text
bento-edps/
├── github/
│   ├── model-test-and-deploy.yml/
├── model-desc/
│   ├── edp-props.yml 
│   └── terms/
│       ├── obib-terms.yml
├── README.md
└── pyproject.toml
```

- **`edp-props.yml`** declares each EDP itself — its identity (`Term`) and the list of permissible values it currently contains (`Enum`).
- **`terms/*.yml`** files per edp provide richer definitions (codes, versions, definitions) for the individual PVs referenced in `Enum`, keyed by value. Splitting these out keeps `edp-props.yml` readable and lets each external vocabulary's term metadata live in its own file as the library grows.

## Available EDPs
| EDP Handle         | Origin  |  Code   | Description                       | Terms file                |
|:-------------------|:--------|:--------|:----------------------------------|:--------------------------|
| `obib_terms_valueset`  | CRDC   | CRDC0002  | Standardized permissible values (from OBIB) describing biological specimens   | `terms/obib-terms.yml`  |
*(Add a row here each time a new EDP is introduced.)*

## EDP definition format
Each EDP is declared as a `PropDefinitions` entry in `edp-props.yml`:
```yaml
Nodes: null
Relationships: null 
PropDefinitions:
  <edp_vs_handle>:
    Desc: <human-readable description of what this EDP represents>
    Ext: true            # Required. Marks this property as an EDP.
    Term:                # Required. Identifies the EDP itself.
      Origin: <EDP origin>  # Approved origin authority for the EDP identifier, eg, CRDC (can be anything)
      Code: "<EDP code>" # Unique identifier for this EDP, eg, CRDC0001
      Version: "<version>" # eg, "1"
      Value: <display name for the EDP>
      Definition: <definition of the EDP>
    Enum:                # Required. The current list of permissible values.
      - "<value 1>"
      - "<value 2>"
      ...
```

## Term definitions file format 
Each value in an EDP's Enum list is enriched with metadata in the corresponding `terms/*.yml` file:

```yaml
Terms:
  <value matching an Enum entry>:
    Origin: <source vocabulary, e.g. OBIB>
    Code: "<source code, e.g. OBIB:0000070>"
    Version: "<source version>"
    Value: <value — should match the Enum entry exactly>
    Definition: "<definition of this term>"
    ...
```

## Referencing an EDP from your model (MDF)
To have a property in your own model's MDF draw its permissible values from an EDP defined here, rather than maintaining the value list yourself, reference the EDP's Term identity (Origin + Code + Version) from your property definition. Sample below. 

```yaml
Handle: Sample
Version: 0.0.0
Nodes:
  program:
    Props:
      - program_name
Relationships: {}
PropDefinitions:
  program_name:
    Desc: |
      The name of the program
    Ext: true       # indicator that an EDP and its corresponding valueset is being referenced
    Term:           # section to point to the details of the right EDP required out of the ones available 
      Origin: CRDC      
      Code: CRDC00001
      Value: NCI Program Names
      Version: "1"
      Definition: Sample EDP for program names.
    Enum:   
      - term_1
      - term_2
```
Once wired up, the MDB update pipeline recognizes that the property's value set is EDP-backed and will simply link the property's CDE to the shared EDP value_set in the MDB using a :specifies_value_set relationship. 

## Adding a new EDP 

- Add a new entry under `PropDefinitions` in `edp-props.yml` following the format above. Choose a clear, unique <edp_handle> and Code.
- Create a new terms/<source>-terms.yml file with term definitions for each value in your Enum list.
- Open a PR. Once merged to main, the weekly poll in bento-mdb will pick up the change automatically and propagate it to the MDB on the next run.


## Updating an existing EDP 

- **Adding values:** append to the Enum list (and add a corresponding entry in the terms file). These will be created in the MDB on the next scheduled update.
- **Removing values**: remove from the Enum list. (Note: automatic deletion of removed values from the MDB is not yet implemented — this is tracked separately. Until that lands, removing a value here will stop new references to it but will not retroactively detach it in the MDB.)

## Questions/Issues

For questions about EDP usage in a specific model, or to report a vocabulary that should become an EDP, contact the bento-mdb team or open an issue in this repository for discussion. 