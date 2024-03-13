# OML Export and Import  Serialized Models
## Serialized model format
The serialized model format was introduced in Oracle Database 18c as a lightweight approach to support scoring. DBMS_DATA_MINING.EXPORT_SERMODEL exports a single model to a serialized BLOB so it can be imported and scored in a separate Oracle Machine Learning (OML) database instance or to OML Services. DBMS_DATA_MINING.IMPORT_SERMODEL takes the serialized BLOB and creates the model in the target database.

As a result of building models, each model has a set of model detail views that provide information about the model such as model statistics for evaluation. These model detail views can be queried by the user. With serialized models, only the model data and metadata required for scoring are available in the serialized model, so it is more compact and transfers faster to the target environment than dump files produced by the DBMS_DATA_MINING.EXPORT_MODEL procedure.

If full model details are required, use the DBMS_DATA_MINING.EXPORT_MODEL and DBMS_DATA_MINING.IMPORT_MODEL procedures. Serialized model export support only models that produce scores. Specifically, it does not support Attribute Importance, Association Rules, Exponential Smoothing, or O-Cluster (although O-Cluster does support scoring). Use DBMS_DATA_MINING.EXPORT_MODEL to export these models and scenarios when the full model details are required.

## Deployment Scenarios
The same machine learning algorithms are available in `both Autonomous Database and on-premises.` Users have the option to build models on premises and deploy to the cloud, and vice versa.

Serialized models can also be deployed to OML Services, which provides REST endpoints hosted on Autonomous Database. These endpoints enable storing OML models and creating scoring endpoints. The REST API for OML Services supports both OML models and ONNX format models, and enables cognitive text functionality.

In this notebook we demonstrate the following scenarios using DBMS_DATA_MINING.EXPORT_SERMODEL and DBMS_DATA_MINING.IMPORT_SERMODEL:

- Migrating models between two schemas in the same database
- Migrating models between two separate database instances
  - Autonomous Database to Autonomous Database
  - Autonomous Database to Oracle Database
  - Oracle Database to Autonomous Database
  - Oracle Database to Oracle Database
