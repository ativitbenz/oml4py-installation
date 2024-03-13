# OML Export and Import  Serialized Models
## Serialized model format
The serialized model format was introduced in Oracle Database 18c as a lightweight approach to support scoring. `DBMS_DATA_MINING.EXPORT_SERMODEL` exports a single model to a serialized BLOB so it can be imported and scored in a separate Oracle Machine Learning (OML) database instance or to OML Services. `DBMS_DATA_MINING.IMPORT_SERMODEL` takes the serialized BLOB and creates the model in the target database.

As a result of building models, each model has a set of model detail views that provide information about the model such as model statistics for evaluation. These model detail views can be queried by the user. With serialized models, only the model data and metadata required for scoring are available in the serialized model, so it is more compact and transfers faster to the target environment than dump files produced by the `DBMS_DATA_MINING.EXPORT_MODEL` procedure.

If full model details are required, use the `DBMS_DATA_MINING.EXPORT_MODEL` and `DBMS_DATA_MINING.IMPORT_MODEL` procedures. Serialized model export support only models that produce scores. Specifically, it does not support Attribute Importance, Association Rules, Exponential Smoothing, or O-Cluster (although O-Cluster does support scoring). Use `DBMS_DATA_MINING.EXPORT_MODEL` to export these models and scenarios when the full model details are required.

## Deployment Scenarios
The same machine learning algorithms are available in `both Autonomous Database and on-premises.` Users have the option to build models on premises and deploy to the cloud, and vice versa.

Serialized models can also be deployed to OML Services, which provides REST endpoints hosted on Autonomous Database. These endpoints enable storing OML models and creating scoring endpoints. The REST API for OML Services supports both OML models and ONNX format models, and enables cognitive text functionality.

In this notebook, we demonstrate the following scenarios using DBMS_DATA_MINING.EXPORT_SERMODEL and DBMS_DATA_MINING.IMPORT_SERMODEL:

- Migrating models between two schemas in the same database
- Migrating models between two separate database instances
  - Autonomous Database to Autonomous Database
  - Autonomous Database to Oracle Database
  - Oracle Database to Autonomous Database
  - Oracle Database to Oracle Database
 
Download Dataset from: `https://gist.github.com/noamross/e5d3e859aa0c794be10b` 

In this scenario, we use separate databases for the development and testing of the machine learning model and ultimately production. Here, data scientists build models in one environment, and when the machine learning model produces satisfactory results, it can be exported from the development database and imported into the production database.

Step1: Import dataset using SQL developer software and granting all privileges to a user is generally `not` recommended due to security concerns, as it provides the user with full control over the database.
```
GRANT ALL PRIVILEGES TO username;
```
Step2: Generated index columns for training data
```
-- GENERATED INDEX COLOUMNS
ALTER TABLE cars
ADD (CID NUMBER GENERATED ALWAYS AS IDENTITY);
```
Step3: Build model
```
BEGIN DBMS_DATA_MINING.DROP_MODEL('GLM_MOD');
EXCEPTION WHEN OTHERS THEN NULL; END;
/

DECLARE
  v_setlst DBMS_DATA_MINING.SETTING_LIST;
    
BEGIN
  v_setlst('PREP_AUTO') := 'ON';
  v_setlst('ALGO_NAME') := 'ALGO_GENERALIZED_LINEAR_MODEL';
  v_setlst('GLMS_DIAGNOSTICS_TABLE_NAME') := 'GLMR_DIAG';
    
  DBMS_DATA_MINING.CREATE_MODEL2(
    model_name          => 'GLM_MOD',
    mining_function     => 'REGRESSION',
    data_query          => 'SELECT * FROM CARS',
    set_list            =>  v_setlst,
    case_id_column_name => 'CID',
    target_column_name  => 'MPG');
END;
/
```
Step4: Search model to created
```
SELECT MODEL_NAME, MINING_FUNCTION, ALGORITHM, CREATION_DATE 
FROM   ALL_MINING_MODELS 
WHERE  MODEL_NAME='GLM_MOD';
```
Step5: 

