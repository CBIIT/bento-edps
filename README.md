# bento-edps: Extended Definition Properties

This is the common repository for defining CRDC Extended Definition Properties
(EDPs) using Model Definition Format ([MDF](https://cbiit.github.io/bento-mdf/mdf.html)) files 
(located in [model-desc](/model-desc)).

EDPs allow for the creation of custom standardized permissible value (PV) sets that can be 
used across CRDC data commons models. These are created to supplement PV sets that are
maintained by [caDSR](https://cadsr.cancer.gov).

EDPs are defined here and processed by the Model Database ([MDB](https://github.com/CBIIT/bento-mdb)) automated update scripts
to persist these lists in the MDB. EDP PV sets can be accessed for data validation using
the EDP endpoints of the Simple Terminology Service ([STS](https://github.com/CBIIT/bento-sts-fastapi)) at https://sts.cancer.gov/v2.

