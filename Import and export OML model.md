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
Step3: Drop, then build a model
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
Step4: View model in databases
```
SELECT MODEL_NAME, MINING_FUNCTION, ALGORITHM, CREATION_DATE 
FROM   ALL_MINING_MODELS 
WHERE  MODEL_NAME='GLM_MOD';
```
Step5: Drop, Create temporary table model export
```
BEGIN EXECUTE IMMEDIATE 'DROP TABLE MODEL_EXPORT';  -- drop table if it exists
EXCEPTION WHEN OTHERS THEN NULL; END;
/

CREATE TABLE MODEL_EXPORT(MY_MODEL BLOB)
```
Step6: Export the model to the table
```
DECLARE
  BLOB_MODEL BLOB;
BEGIN
  DBMS_LOB.CREATETEMPORARY(BLOB_MODEL, FALSE);
  DBMS_DATA_MINING.EXPORT_SERMODEL(BLOB_MODEL, 'GLM_MOD');
  INSERT INTO MODEL_EXPORT values (BLOB_MODEL);   -- serialized model BLOB saved in table MODEL_EXPORT
  commit;
  DBMS_LOB.FREETEMPORARY(BLOB_MODEL);
END;
```
```
-- Confirm the model was exported by looking at the length of the BLOB
SELECT LENGTH(MY_MODEL) AS MODEL_LEN from MODEL_EXPORT
```
Step7: Create a procedure to write the model to a serialized BLOB 
Define the `write_serialized_model procedure` to export a model as a serialized BLOB to a directory in the database environment
```
CREATE OR REPLACE PROCEDURE write_serialized_model (v_blob blob, dirName varchar2, filName varchar2) IS
    v_output utl_file.file_type;
    v_amt binary_integer := 2000;
    v_raw raw(2000);
    v_pos binary_integer := 1;
    v_len binary_integer;
BEGIN
    v_output := utl_file.fopen(dirName, filName, 'wb', 32760);
    v_len := dbms_lob.getlength(v_blob);
WHILE (v_pos <= v_len) loop
  IF (v_len - v_pos + 1) < 2000 then
    v_amt := v_len - v_pos + 1;
  END IF;
  dbms_lob.read(v_blob,v_amt,v_pos,v_raw);
  utl_file.put_raw(v_output, v_raw);
  utl_file.fflush(v_output);
  v_pos := v_pos + v_amt;
 END LOOP;
 utl_file.fclose(v_output);
END;
```

Step8: Export a serialized model from an Oracle Database and import to another Oracle Database
1.Export the model to a serialized BLOB from Oracle Database to the database server Operating System.
  a. Define the `write_serialized_model` procedure under “Prerequisites” to export a model as a serialized BLOB to a directory.
  b. Define the directory where the serialized model will be exported in SQL.
  c. Export the model using `DBMS_DATA_MINING.EXPORT_SERMODEL`.

2. Import the serialized model to the target Oracle Database
  a. Download the serialized model to the target database server.
  b. In SQL, define the directory from which the serialized model will be imported.
  c. Import the model using `DBMS_DATA_MINING.IMPORT_SERMODEL`.

`note` In this example, the name of the in-database model is GLM_MOD and the name of the serialized model is GLM_MOD.mod.

The following commands will run in a SQL prompt on the database server or in a SQL Developer instance connected to the database server as the OML user:

1. Export the model to a file on the database server
```
----------------------------------------------------
-- Define directory where serialized model is saved
----------------------------------------------------
CREATE OR REPLACE DIRECTORY MYDIR AS '/home/oracle';       

DECLARE
  MODNAME VARCHAR2(100) := 'GLM_MOD';
  SER_MODNAME VARCHAR2(100);
  BLOB_MODEL BLOB;
BEGIN
  DBMS_LOB.CREATETEMPORARY(BLOB_MODEL, FALSE);
  SELECT CONCAT(MODNAME,'.mod') INTO SER_MODNAME from dual;
  ---------------------------
  -- export serialized model 
  ---------------------------
  DBMS_DATA_MINING.EXPORT_SERMODEL(BLOB_MODEL, 'GLM_MOD'); 
  ---------------------------------------------------
  -- convenience function to write a model to a BLOB
  ---------------------------------------------------
  write_serialized_model(BLOB_MODEL, 'MYDIR', SER_MODNAME);   
  DBMS_LOB.FREETEMPORARY(BLOB_MODEL);
END;
/
```
2. Copy the serialized model to the target database server and import the model
```
---------------------------
-- Drop table if it exists
---------------------------
BEGIN EXECUTE IMMEDIATE 'DROP TABLE MODEL_IMPORT';           
EXCEPTION WHEN OTHERS THEN NULL; END;
/

---------------------------
-- Drop model if it exists
---------------------------
BEGIN DBMS_DATA_MINING.DROP_MODEL('GLM_MOD');             
EXCEPTION WHEN OTHERS THEN NULL; END;
/

----------------------------------------------------
-- Define directory where serialized model is saved
----------------------------------------------------
CREATE OR REPLACE DIRECTORY MYDIR AS '/home/oracle'; 

-----------------------------------------------
-- Create table to temporarily store the model
-----------------------------------------------
CREATE TABLE MODEL_IMPORT(MY_MODEL BLOB)                     

DECLARE
  BLOB_MODEL BLOB;
BEGIN
  INSERT INTO MODEL_IMPORT values (BFILENAME('MYDIR','GLM_MOD.mod'));
  commit;
  SELECT MY_MODEL INTO BLOB_MODEL FROM MODEL_IMPORT;
  --------------------------
  -- Import serialized model
  --------------------------
  DBMS_DATA_MINING.IMPORT_SERMODEL(BLOB_MODEL, 'GLM_MOD');  
END;
/

Test the model:
-----------------------------
-- Verify model was imported
 ----------------------------
SELECT * FROM USER_MINING_MODELS WHERE MODEL_NAME='GLM_MOD';    
-----------------------------
-- Model description
 ----------------------------
SELECT MODEL_NAME, MINING_FUNCTION, ALGORITHM, CREATION_DATE FROM USER_MINING_MODELS WHERE MODEL_NAME='GLM_MOD';  
-----------------------------
-- Model settings
 ----------------------------
SELECT SETTING_NAME, SETTING_VALUE FROM ALL_MINING_MODEL_SETTINGS WHERE MODEL_NAME='GLM_MOD'; 
-----------------------------
-- Perform scoring
 ----------------------------
SELECT CID, ROUND(PREDICTION(GLM_MOD USING *), 1)  
AS PREDICTED_MPG, MPG AS ACTUAL_MPG FROM CARS ORDER BY CID;
```
## Clean up

Drop tables (optional)
```
BEGIN EXECUTE IMMEDIATE 'DROP TABLE CARS';          -- drop table CARS if it exists
EXCEPTION WHEN OTHERS THEN NULL; END;
/
BEGIN EXECUTE IMMEDIATE 'DROP TABLE MODEL_EXPORT';  -- drop table MODEL_EXPORT if it exists
EXCEPTION WHEN OTHERS THEN NULL; END;
/
BEGIN EXECUTE IMMEDIATE 'DROP TABLE MODEL_IMPORT';  -- drop table MODEL_IMPORT if it exists
EXCEPTION WHEN OTHERS THEN NULL; END;
/
```
Drop model(optional)
```
BEGIN DBMS_DATA_MINING.DROP_MODEL('GLM_MOD'); -- drop the model if it exists
EXCEPTION WHEN OTHERS THEN NULL; END;
```


