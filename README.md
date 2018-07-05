# com.castsoftware.uc.ost

What are the Smell Tests?
Smell Tests are a collection of tests to be executed after Transaction Configuration is completed. There are two specific smell test checks plus other checks useful for a second level of investigation.
The detail for all these checks are explained below. 

How does it connect?
Smell tests are now re written as a User Community Extension and will run automatically if the extension has been installed.

How to Operate?
Installing the extension on your schema was run the smell tests and the executionalry report will be available in the reports section of the AIP

AIP versions supported?
8.3 and above


Smell Tests
Empty Transaction Ratio
This ratio is calculated by comparing non-empty transactions with total transaction. The ratio should be as high as possible, however not all entry points contribute to beginning of transaction hence having some empty transactions is normal. Classic example includes some VB 6 (or vb.net) controls created for the purpose of templates and controls inheriting them contribute to transaction entry points. Considering such scenario, it may be fair to say the expected ratio % to be more than 90% however the possibility of 20% empty transactions would not be uncommon. 
Concerned should be raised if the ratio falls below 70% and investigated to validate the cause. The ratio is calculated directly from TCC tables.
RAG Status

RED >=30%
AMBER >=10% and <30% of Empty Transactions
GREEN >0% and <10% of Empty Transactions
Compute Value 1: Count of Empty Transactions
Compute Value 2: Count of All Transactions
Exclusions and Considerations
None

Artifact Coverage
The below actions are covered in the coverage (The query is completely re-written in OST 3.3) 

objects participating in a transaction
parents of objects participating in a transaction
callees of objects participating in a transaction--level1
callees of objects participating in a transaction--level2
parents of callees of objects participating in a transaction--level1
parents of callees of objects participating in a transaction--level2
remaining child objects of the parent of the paticipating object of a transaction
remaining child objects of the parents of callees of objects participating in a transaction--level1
remaining child objects of the parents of callees of objects participating in a transaction--level2
 

This check identifies the total count of artifacts per type for the application and the count of those artifact types associated to transactions. An aggregate percentage of transactions artifacts against total artifacts gives the percentage coverage. 

The technology artifacts taken into consideration and the RAG status parameters are shown below:
RAG Status

RED (0 - 30% coverage)
AMBER (30 to 50% coverage)
GREEN for more than 50% coverage of classes
Compute Value 1: Artifacts Count in Transactions
Compute Value 2: Artifact Count
Technology Artifacts

Progress Program
Cobol Program
C++ Class
VB.NET Class
ColdFusion Fuse Action
VB MDI Form
SHELL Program,
ColdFusion Template
C/C++ File
C# Class
Java Class
VB Module
Power Center Session


Exclusions and Considerations

All external artifacts are excluded from the calculations. (Artifacts with property value set to 1 in CTT_OBJECT_PROPERTY table)
Certain object types which are never expected to be part of transactions and on the reverse there are certain object types which are the key artifact types which can be considered for evaluation of the artifact coverage. This approach gives objective answer for observation and measurement.
 

These limits are maintained in table core_smelltest_threshholds. This table contains the different object types per technology which will be considered for coverage evaluation and range also is maintained which determines the RED,AMBER or GREEN range.
 

For object type "Methods" the calculation is considering the coverage of its parent classes instead, hence if any 1 method of a class is found associated to a transaction, the class is taken as consideration as part of the transaction. That way the granularity is reduced but a more definitive result is obtained.
 

Different object types can have different thresholds to remain in GREEN, reason being one would expect such artifacts to always remain in transaction. Any new object type added to the table or updates in the thresholds is immediately reflected in the results of the views.


 

Other Tests
Calibration Kit Applied
This check executes a Postgres query on the local schema that returns the package names in the table sys_package_version to know if there is package by name "com.castsoftware.uc.transactioncalibrationkit" present and its corresponding RAG colour code based on the below thresholds: 
GREEN: If package by name com.castsoftware.uc.transactioncalibrationkit is present 
RED: If package by name com.castsoftware.uc.transactioncalibrationkit is not present
Compute Value 1: Not Applicable
Compute Value 2: Not Applicable

FP to Class Ratio
This ratio is calculated between total functions points and total number of classes/programs. Classes are considered for technologies like J2EE/.Net where Programs are considered for technologies like Mainframe. Typically counts would show 3 function points per class/program.
RAG Status

RED for FP/Class ratio is >=0 and <1 OR >=5
AMBER for FP/Class ratio is >=1 and <2 OR >=4 and <5
GREEN for FP/Class ratio is >=2 and <4
Compute Value 1: Count of Classes
Compute Value 2: Count of FP
Exclusions and Considerations
None

DLM Reviewed
This check executes a Postgres query on the local schema that returns the Total DLM's as compute_value2, New DLM's as compute_value1, the Percentage difference between them and its corresponding RAG colour code based on the below thresholds:
GREEN: No Additional DLM's
RED: New DLM's present
Compute Value 1: New DLM's
Compute Value 2: Total DLM's
Exclusions and Considerations
None

TFP/DFP Ratio
This ratio is calculated directly from the value of transaction function point count and data function point count. A balanced application with associated database is expected to follow a ratio of 2:1 (i.e. for 2 transaction function points 1 data function points is expected.)
Compute Value 1: DFP count
Compute Value 2: TFP count
RAG Status

RED for TFP/DFP ratio is >=0 and <1 OR >=5
AMBER for TFP/DFP ratio is >=1 and <2 OR >=4 and <5
GREEN for TFP/DFP ratio is >=2 and <4
Exclusions and Considerations
None

Smell Test Queries

The schema name for the "set search_path =" should be manually updated to reflect the correct name. At execution time the correct schema names are dynamically updated in the relevant queries.

Empty Transaction Ratio(local):
set search_path = test_local; 
select cast(tb_all_tr.tr_count as text) as compute_value2, cast(tb_empty_tr.tr_count as text) as compute_value1 , 
cast( ((tb_empty_tr.tr_count * 100)/ tb_all_tr.tr_count) as text) as percentage_ratio,
case when (((tb_empty_tr.tr_count * 100)/ tb_all_tr.tr_count) >= 0 and ((tb_empty_tr.tr_count * 100)/ tb_all_tr.tr_count) < 10) then 'GREEN'
when (((tb_empty_tr.tr_count * 100)/ tb_all_tr.tr_count) >=10 and ((tb_empty_tr.tr_count * 100)/ tb_all_tr.tr_count) < 30) then 'AMBER'
else 'RED'
end as rag
from
(select count(cob.object_name) as tr_count
from dss_transaction dtr, cdt_objects cob
where dtr.form_id = cob.object_id
and dtr.cal_mergeroot_id = 0 
and dtr.cal_flags not in ( 8, 10, 126, 128,136, 138, 256, 258 ) 
) tb_all_tr,
(select count(cob.object_name) as tr_count from dss_transaction dtr, cdt_objects cob 
where dtr.form_id = cob.object_id
and dtr.cal_mergeroot_id = 0 
and dtr.cal_flags not in ( 8, 10, 126, 128,136, 138, 256, 258 ) 
and DTR.tf_ex=0 ) tb_empty_tr;



Artifact Coverage(local):
set search_path = test_local;

--percentage Artifact coverage--
select total_count,covered_count,cast((covered_count*100/total_count)as integer)||'%' as percent_covered,
CASE when cast((covered_count*100/total_count)as integer) <= 30 THEN 'RED'
when (cast((covered_count*100/total_count)as integer) <= 50
and cast((covered_count*100/total_count)as integer) > 30) THEN 'AMBER'
when cast((covered_count*100/total_count)as integer) > 50 THEN 'GREEN' end as color
from
(
select count(distinct cdt.object_id) as total_count from cdt_objects cdt, ctt_object_applications ctt where cdt.object_id=ctt.object_id and ctt.properties<>1 and
cdt.object_type_str in ('Progress Program','Cobol Program','C++ Class','VB.NET Class','ColdFusion Fuse Action','VB MDI Form','SHELL Program','ColdFusion Template','C/C++ File','C# Class','Java Class','VB Module','Session')) total,
(
select count(distinct cdt.object_id) as covered_count from cdt_objects cdt, ctt_object_applications ctt where cdt.object_id=ctt.object_id and ctt.properties<>1 and ctt.object_id in
--(1)objects participating in a transaction
(select object_id from cdt_objects where object_id in (
select child_id from dss_transactiondetails)
--(2)parents of objects participating in a transaction
union
select parent_id from ctt_object_parents where object_id in (
select child_id from dss_transactiondetails)
--(3)callees of objects participating in a transaction--level1
union
--(3a)callees of objects participating in a transaction--level2
select called_id from ctv_links where caller_id in
(select child_id from dss_transactiondetails)
union
select called_id from ctv_links where caller_id in(
select called_id from ctv_links where caller_id in
(select child_id from dss_transactiondetails))
--(4)parents of callees of objects participating in a transaction--level1
union
select parent_id from ctt_object_parents where object_id in (
select called_id from ctv_links where caller_id in (select child_id from dss_transactiondetails))
--(4a)parents of callees of objects participating in a transaction--level2
union
select parent_id from ctt_object_parents where object_id in (
select called_id from ctv_links where caller_id in(
select called_id from ctv_links where caller_id in
(select child_id from dss_transactiondetails)))
--(5)remaining child objects of the parent of the paticipating object of a transaction
union
select object_id from ctt_object_parents where parent_id in (
select parent_id from ctt_object_parents where object_id in (
select child_id from dss_transactiondetails))
--(6)remaining child objects of the parents of callees of objects participating in a transaction--level1
union
select object_id from ctt_object_parents where parent_id in (
select parent_id from ctt_object_parents where object_id in (
select called_id from ctv_links where caller_id in (select child_id from dss_transactiondetails)))
--(6a)remaining child objects of the parents of callees of objects participating in a transaction--level2
union
select object_id from ctt_object_parents where parent_id in (
select parent_id from ctt_object_parents where object_id in (
select called_id from ctv_links where caller_id in(
select called_id from ctv_links where caller_id in
(select child_id from dss_transactiondetails)))))
and cdt.object_type_str in ('Progress Program','Cobol Program','C++ Class','VB.NET Class','ColdFusion Fuse Action','VB MDI Form','SHELL Program','ColdFusion Template','C/C++ File','C# Class','Java Class','VB Module','Session'))covered

Other Test Queries

The schema name for the "set search_path =" should be manually updated to reflect the correct name. At execution time the correct schema names are dynamically updated in the relevant queries.

Calibration Kit Applied(local):

set search_path = test_local;
Select cast(count(*)as text) as compute_value1, cast(0 as text) as compute_value2, cast(0 as text) as compute_value3, cast(0 as text) as compute_value4,cast(0 as text) as percentage_ratio
,case when (select count * from sys_package_version where package_name like '%com.castsoftware.uc.transactioncalibrationkit%') > 0 then 'GREEN' else 'RED' end as rag
from sys_package_version where package_name like '%com.castsoftware.uc.transactioncalibrationkit%'

 

FP to Class Ratio(Central):

set search_path = test_central;
select cast(Programs_Class as text) as compute_value1, cast(FP as text) as compute_value2, cast(fp_to_pgmclass_ratio as text) as percentage_ratio,
case when fp_to_pgmclass_ratio >=0 and fp_to_pgmclass_ratio < 1 then 'RED'
when fp_to_pgmclass_ratio >=1 and fp_to_pgmclass_ratio < 2 then 'AMBER'
when fp_to_pgmclass_ratio >=2 and fp_to_pgmclass_ratio < 4 then 'GREEN'
when fp_to_pgmclass_ratio >=4 and fp_to_pgmclass_ratio < 5 then 'AMBER'
when fp_to_pgmclass_ratio >=5 then 'RED'
end as rag
from(
select pgm_classes.pgm_count Programs_Class, fp_count.fp FP, 
round(
CASE
WHEN pgm_classes.pgm_count = 0::numeric THEN 0::numeric
ELSE fp_count.fp / pgm_classes.pgm_count
END, 2) AS fp_to_pgmclass_ratio 
from 
(select sum(metric_num_value) as pgm_count
from 
dss_metric_results t1,
dss_metric_types t2, 
dss_objects t3
where
t1.metric_id = t2.metric_id
and t1.object_id = t3.object_id
and t3.object_type_id = -102 
and t1.snapshot_id = (select max(snapshot_id) from dss_snapshots)
and t1.metric_id in ( 10155, 10156)) as pgm_classes,
(select sum(metric_num_value) as fp from 
dss_metric_results t1,
dss_metric_types t2, 
dss_objects t3
where
t1.metric_id = t2.metric_id
and t1.object_id = t3.object_id
and t3.object_type_id = -102 
and t1.snapshot_id = (select max(snapshot_id) from dss_snapshots)
and t1.metric_id in ( 10203, 10204)) as fp_count 
) as dataTable

 

DLM Reviewed(local):
set search_path = test_local;
select cast((select count* from acc where prop = 1) as text) as compute_value1, cast((select count * from acc where prop = 0) as text) as compute_value2, cast((select count * from acc where prop = 65537) as text) as compute_value3,cast((select count * from acc where prop = 32769) as text) as compute_value4,cast( (((select count * from acc where prop <> 1) * 100)/ (select count * from acc)) as text) as percentage_ratio, case when (select count * from acc where prop = 1) > 0 then 'RED' else 'GREEN' end as rag 




TFP/DFP Ratio(local):

set search_path = test_local;
select cast(TFP.TFP as text) as compute_value2, cast(DFP.DFP as text) as compute_value1, cast(TFP/DFP as text) as percentage_ratio, 
case when TFP/DFP >=0 and TFP/DFP < 1 then 'RED'
when TFP/DFP >=1 and TFP/DFP < 2 then 'AMBER'
when TFP/DFP >=2 and TFP/DFP < 4 then 'GREEN'
when TFP/DFP >=4 and TFP/DFP < 5 then 'AMBER'
when TFP/DFP >=5 then 'RED'
end as rag
from (
select sum(DTR.tf_ex) as TFP
from dss_transaction dtr, cdt_objects cob
where dtr.form_id = cob.object_id
and dtr.cal_mergeroot_id = 0 
and dtr.cal_flags not in ( 8, 10, 126, 128,136, 138, 256, 258 ) 
) as TFP,
( 
select sum(dtf.ilf_ex) as DFP
from dss_datafunction dtf, cdt_objects cob
where dtf.maintable_id = cob.object_id 
and dtf.cal_flags in (0,2)
) as DFP;
 
