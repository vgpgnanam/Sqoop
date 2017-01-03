#Sqoop

sqoop list-databases \
  --connect "jdbc:mysql://nn01.itversity.com:3306" \
  --username retail_dba \
  --password itversity

sqoop list-tables \
  --connect "jdbc:mysql://nn01.itversity.com:3306/retail_db" \
  --username retail_dba \
  --password itversity

sqoop eval \
  --connect "jdbc:mysql://nn01.itversity.com:3306/retail_db" \
  --username retail_dba \
  --password itversity \
  --query "select * from departments limit 10"

hostname -f

--as-avrodatafile	Imports data to Avro Data Files
--as-sequencefile	Imports data to SequenceFiles
--as-textfile	Imports data as plain text (default)
--as-parquetfile	Imports data to Parquet Files (from 1.4.6)

#Sqoop import-all - Default it stored as-textfile

sqoop import-all-tables \
  --connect "jdbc:mysql://nn01.itversity.com:3306/retail_db" \
  --username=retail_dba \
  --password=itversity \
  --warehouse-dir=/user/gnanaprakasam/sqoop_import_new

#Sqoop import-all - stored as-avrodatafile

sqoop import-all-tables -Dmapreduce.job.user.classpath.first=true \
  --connect "jdbc:mysql://nn01.itversity.com:3306/retail_db" \
  --username=retail_dba \
  --password=itversity \
  -m 1 \
  --as-avrodatafile \
  --warehouse-dir /user/gnanaprakasam/sqoop_import_avro

  hive -e "use gnanaprakasam; select * from departments limit 10";

  #Sqoop import-all-table into hive, compress

sqoop import-all-tables \
  --connect "jdbc:mysql://nn01.itversity.com:3306/retail_db" \
  --username retail_dba \
  --password itversity \
  --hive-import \
  --create-hive-table \
  --hive-overwrite \
  --compress \
  --compression-codec org.apache.hadoop.io.compress.SnappyCodec \
  --hive-database gnanaprakasam

#sqoop import single table, use target-dir instead of warehouse-dir, 

sqoop import \
--connect "jdbc:mysql://nn01.itversity.com:3306/retail_db" \
--username retail_dba \
--password itversity \
--table departments \
--target-dir /apps/hive/warehouse/gnanaprakasam.db/sqoop_import_New/departments_ls \
--lines-terminated-by '\n' \
--fields-terminated-by '|'

# sqoop import --as-textfile

sqoop import \
  --connect "jdbc:mysql://nn01.itversity.com:3306/retail_db" \
  --username=retail_dba \
  --password=itversity \
  --table departments \
  --as-textfile \
  --target-dir=/user/gnanaprakasam/sqoop_import/departments \
  --fields-terminated-by '|' \
  --lines-terminated-by '\n' \
  --null-string nvl \
  --null-non-string -1


# sqoop import --as-sequencefile

sqoop import \
  --connect "jdbc:mysql://nn01.itversity.com:3306/retail_db" \
  --username=retail_dba \
  --password=itversity \
  --table departments \
  --as-sequencefile \
  --target-dir=/user/gnanaprakasam/sqoop_import/department_seq

  --fields-terminated-by '\001' \
  --lines-terminated-by '\n'

# sqoop import --as-parquetfile

sqoop import \
  --connect "jdbc:mysql://nn01.itversity.com:3306/retail_db" \
  --username=retail_dba \
  --password=itversity \
  --table departments \
  --as-parquetfile \
  --target-dir=/user/gnanaprakasam/sqoop_import/departments_parquet


#sqoop import, --boundary-query & --columns

sqoop import \
  --connect "jdbc:mysql://nn01.itversity.com:3306/retail_db" \
  --username retail_dba \
  --password itversity \
  --table departments \
  --target-dir /apps/hive/warehouse/gnanaprakasam.db/sqoop_import_New/departments_ls \
  --boundary-query "select 2,8 from departments limit 1" \
  --columns department_id,department_name

#sqoop import --split-by & $CONDITIONS & --table & --query both are mutually exclusive

sqoop import \
  --connect "jdbc:mysql://nn01.itversity.com:3306/retail_db" \
  --username retail_dba \
  --password itversity \
  --query="select * from orders join order_items on orders.order_id = order_items.order_item_order_id where \$CONDITIONS" \
  --target-dir /user/gnanaprakasam/ordersJoin \
  --split-by order_id
  
#sqoop import --where & --append

sqoop import \
  --connect "jdbc:mysql://nn01.itversity.com:3306/retail_db" \
  --username retail_dba \
  --password itversity \
  --append \
  --table departments \
  --where "department_id > 7" \
  --split-by department_id \
  --target-dir /apps/hive/warehouse/gnanaprakasam.db/sqoop_import_New/departments_ls \
  --outdir java_files

# Create Hive table

create table departments(
department_id int,
department_name string)
row format delimited fields terminated by '|'
stored as textfile;

# sqoop import to already existing Hive table

sqoop import \
  --connect "jdbc:mysql://nn01.itversity.com:3306/retail_db" \
  --username retail_dba \
  --password itversity \
  --table departments \
  --hive-home /apps/hive/warehouse/gnanaprakasam.db \
  --hive-import \
  --hive-overwrite \
  --hive-table departments\
  --hive-database gnanaprakasam \
  --outdir java_files \
  --fields-terminated-by '|' \
  --lines-terminated-by '\n'

# Incremental Load


sqoop import \
  --connect "jdbc:mysql://nn01.itversity.com:3306/retail_db" \
  --username=retail_dba \
  --password=itversity \
  --table departments \
  --target-dir /apps/hive/warehouse/gnanaprakasam.db/departments \
  --append \
  --fields-terminated-by '|' \
  --lines-terminated-by '\n' \
  --check-column "department_id" \
  --incremental append \
  --last-value 7 \
  --outdir java_files

# Initial sqoop import

sqoop import \
  --connect "jdbc:mysql://nn01.itversity.com:3306/retail_db" \
  --username retail_dba \
  --password itversity \
  --table departments \
  --target-dir /user/gnanaprakasam/sqoop_merge/departments \
  --outdir java_file \
  --as-textfile \
  --where "department_id <=5"

sqoop import \
  --connect "jdbc:mysql://nn01.itversity.com:3306/retail_db" \
  --username retail_dba \
  --password itversity \
  --table departments \
  --append \
  --target-dir /user/gnanaprakasam/sqoop_merge/departments_delta \
  --outdir java_file \
  --as-textfile \
  --where "department_id > 5"

sqoop merge \
  --merge-key department_id \
  --new-data hdfs://nn01.itversity.com:8020/user/gnanaprakasam/sqoop_merge/departments_delta \
  --onto /user/gnanaprakasam/sqoop_merge/departments \
  --target-dir /user/gnanaprakasam/sqoop_merge/departments_stage \
  --class-name departments \
  --jar-file /tmp/sqoop-gnanaprakasam/compile/7538867c50efb265fc588863cc75a1e9/departments.jar

hadoop fs -rm -R /user/gnanaprakasam/sqoop_merge/departments
hadoop fs -mv /user/gnanaprakasam/sqoop_merge/departments_stage /user/gnanaprakasam/sqoop_merge/departments

# sqoop export

sqoop export -Dmapreduce.job.user.classpath.first=true \

sqoop export \
  --connect "jdbc:mysql://nn01.itversity.com:3306/retail_export" \
  --username retail_dba \
  --password itversity \
  --table department_export_gnana \
  --export-dir /user/gnanaprakasam/sqoop_import/departments \
  --input-fields-terminated-by '|' \
  --input-lines-terminated-by '\n' \
  --update-key department_id \
  --update-mode allowinsert \
  --input-null-string nvl \
  --input-null-non-string -1


sqoop export \
  --connect "jdbc:mysql://nn01.itversity.com:3306/retail_export" \
  --username retail_dba \
  --password itversity \
  --table department_export_gnana \
  --export-dir /user/gnanaprakasam/sqoop_import/department_seq \
  --class-name departments \
  --jar-file /tmp/sqoop-gnanaprakasam/compile/11547b2c1058549abd66d240a4601ffb/departments.jar
  
  --input-fields-terminated-by '\001' \
  --input-lines-terminated-by '\n'

sqoop export -Dmapreduce.job.user.classpath.first=true \
  --connect "jdbc:mysql://nn01.itversity.com:3306/retail_export" \
  --username retail_dba \
  --password itversity \
  --table department_export_gnana \
  --export-dir /user/gnanaprakasam/sqoop_import/departments_parquet \
  --class-name codegen_departments \
  --jar-file /tmp/sqoop-gnanaprakasam/compile/185c0d253525db603f67af4605afb2c9/codegen_departments.jar


In mysql 
CREATE TABLE departments_ls AS SELECT * FROM retail_db.departments WHERE 1=2;

sqoop export \
--connect "jdbc:mysql://nn01.itversity.com:3306/retail_db" \
--username retail_dba \
--password itversity \
--table departments_ls \
--export-dir /apps/hive/warehouse/gnanaprakasam.db/sqoop_import/departments_ls/* \
--input-lines-terminated-by '\n' \
--input-fields-terminated-by '|'

sqoop list-tables \
  --connect "jdbc:mysql://nn01.itversity.com:3306/retail_db" \
  --username retail_dba \
  --password-file hdfs://nn01.itversity.com:8020/user/gnanaprakasam/.password

  --password itversity
