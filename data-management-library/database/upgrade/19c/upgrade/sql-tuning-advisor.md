# Query Your Data

## Introduction

*Describe the lab in one or two sentences, for example:* This lab walks you through the steps to ...

Estimated Lab Time: n minutes

### About Product/Technology
Enter background information here..

### Objectives

*List objectives for the lab - if this is the intro lab, list objectives for the workshop*

In this lab, you will:
* Objective 1
* Objective 2
* Objective 3

### Prerequisites

*Use this section to describe any prerequisites, including Oracle Cloud accounts, set up requirements, etc.*

* An Oracle Free Tier, Always Free, Paid or LiveLabs Cloud Account
* Item no 2 with url - [URL Text](https://www.oracle.com).

*This is the "fold" - below items are collapsed by default*

## **STEP 1**: title

In the previous section you did fixed plans with SQL Plan Management. But lets see what else could be done and ask the SQL Tuning Advisor (STA).

You’ll pass the SQL Tuning Set from the “Load” exercise where you captured the HammerDB workload directly from Cursor Cache to the SQL Tuning Advisor and check the results.
Analyze the SQL Tuning Set and generate recommendations

A complete script is provided: sta_cc.sql.
It will:

    Generate a tuning task with the SQL Tuning Set STS_CaptureCursorCache
    Run a tuning task where the SQL Tuning Advisor simulates the execution
    Generate a result report in TEXT format
    Generate statements to implement the findings

. upgr19
cd /home/oracle/scripts
sqlplus / as sysdba

@/home/oracle/scripts/sta_cc.sql

It will take 30 seconds – check the output by scrolling up.

At first, the findings, here I display just the first two findings for a COUNT statement on the CUSTOMER table:

-------------------------------------------------------------------------------
Object ID  : 3
Schema Name: TPCC
SQL ID	   : 7m5h0wf6stq0q
SQL Text   : SELECT COUNT(C_ID) FROM CUSTOMER WHERE C_LAST = :B3 AND C_D_ID =
	     :B2 AND C_W_ID = :B1

-------------------------------------------------------------------------------
FINDINGS SECTION (2 findings)
-------------------------------------------------------------------------------

1- Index Finding (see explain plans section below)
--------------------------------------------------
  The execution plan of this statement can be improved by creating one or more
  indices.

  Recommendation (estimated benefit: 97.95%)
  ------------------------------------------
  - Consider running the Access Advisor to improve the physical schema design
    or creating the recommended index.
    create index TPCC.IDX$$_00D60001 on TPCC.CUSTOMER("C_LAST","C_D_ID","C_W_ID
    ");

  Rationale
  ---------
    Creating the recommended indices significantly improves the execution plan
    of this statement. However, it might be preferable to run "Access Advisor"
    using a representative SQL workload as opposed to a single statement. This
    will allow to get comprehensive index recommendations which takes into
    account index maintenance overhead and additional space consumption.

2- Alternative Plan Finding
---------------------------
  Some alternative execution plans for this statement were found by searching
  the system's real-time and historical performance data.

  The following table lists these plans ranked by their average elapsed time.
  See section "ALTERNATIVE PLANS SECTION" for detailed information on each
  plan.

  id plan hash	last seen	     elapsed (s)  origin	  note
  -- ---------- -------------------- ------------ --------------- ----------------
   1 3642382161  2020-02-20/22:12:33	    0.002 AWR		  original plan
   2 1075826057  2020-02-21/10:00:03	    0.004 AWR

  Information
  -----------
  - The Original Plan appears to have the best performance, based on the
    elapsed time per execution.  However, if you know that one alternative
    plan is better than the Original Plan, you can create a SQL plan baseline
    for it. This will instruct the Oracle optimizer to pick it over any other
    choices in the future.
    execute dbms_sqltune.create_sql_plan_baseline(task_name =>
	    'STA_UPGRADE_TO_19C_CC', object_id => 3, owner_name => 'SYS',
	    plan_hash_value => xxxxxxxx);
-----------------------------------------------------------------------------------

You see that the SQL Tuning Advisor interacts with SQL Plan Management from the previous exercise as well.

When you scroll to the end, you will find the implementation section:

-- Script generated by DBMS_SQLTUNE package, advisor framework --
-- Use this script to implement some of the recommendations    --
-- made by the SQL tuning advisor.			       --
--							       --
-- NOTE: this script may need to be edited for your system     --
--	 (index names, privileges, etc) before it is executed. --
-----------------------------------------------------------------
execute dbms_stats.gather_table_stats(ownname => 'TPCC', tabname => 'HISTORY', estimate_percent => DBMS_STATS.AUTO_SAMPLE_SIZE, method_opt => 'FOR ALL COLUMNS SIZE AUTO');
execute dbms_stats.gather_table_stats(ownname => 'TPCC', tabname => 'ORDERS', estimate_percent => DBMS_STATS.AUTO_SAMPLE_SIZE, method_opt => 'FOR ALL COLUMNS SIZE AUTO');
execute dbms_stats.gather_table_stats(ownname => 'TPCC', tabname => 'ORDER_LINE', estimate_percent => DBMS_STATS.AUTO_SAMPLE_SIZE, method_opt => 'FOR ALL COLUMNS SIZE AUTO');
create index TPCC.IDX$$_00D60001 on TPCC.CUSTOMER("C_LAST","C_D_ID","C_W_ID");
create index TPCC.IDX$$_00D60002 on TPCC.CUSTOMER("C_LAST","C_D_ID","C_W_ID");
create index TPCC.IDX$$_00D60003 on TPCC.ORDERS("O_C_ID","O_D_ID","O_W_ID");
create index TPCC.IDX$$_00D60004 on TPCC.CUSTOMER("C_LAST","C_D_ID","C_W_ID");
create index TPCC.IDX$$_00D60004 on TPCC.CUSTOMER("C_LAST","C_D_ID","C_W_ID");
create index TPCC.IDX$$_00D60004 on TPCC.CUSTOMER("C_LAST","C_D_ID","C_W_ID");
create index TPCC.IDX$$_00D60004 on TPCC.CUSTOMER("C_LAST","C_D_ID","C_W_ID");

First of all, remove the duplicate recommendations (you won’t need 3 identical indexes with different names on TPCC.CUSTOMER for sure) marked in BLUE.
Fix everything

This is an exercise. Please don’t do this in a real environment without proper verification. But let us implement all the recommendations and see what happens.

Execute all the recommendations from the Advisor:

execute dbms_stats.gather_table_stats(ownname => ‘TPCC’, tabname => ‘HISTORY’, estimate_percent => DBMS_STATS.AUTO_SAMPLE_SIZE, method_opt => ‘FOR ALL COLUMNS SIZE AUTO’);
execute dbms_stats.gather_table_stats(ownname => ‘TPCC’, tabname => ‘ORDERS’, estimate_percent => DBMS_STATS.AUTO_SAMPLE_SIZE, method_opt => ‘FOR ALL COLUMNS SIZE AUTO’);
execute dbms_stats.gather_table_stats(ownname => ‘TPCC’, tabname => ‘ORDER_LINE’, estimate_percent => DBMS_STATS.AUTO_SAMPLE_SIZE, method_opt => ‘FOR ALL COLUMNS SIZE AUTO’);
create index TPCC.IDX$$_00D60001 on TPCC.CUSTOMER(“C_LAST”,”C_D_ID”,”C_W_ID”);
create index TPCC.IDX$$_00D60003 on TPCC.ORDERS(“O_C_ID”,”O_D_ID”,”O_W_ID”);

Wait a bit until all statements have been executed.

Afterwards repeat the SQL Performance Analyzer runs from the previous exersize and verify the results:

@/home/oracle/scripts/spa_cpu.sql
@/home/oracle/scripts/spa_report_cpu.sql
@/home/oracle/scripts/spa_elapsed.sql
@/home/oracle/scripts/spa_report_elapsed.sql
exit

Compare the two resulting reports again – and compare them to the examples from the previous run.

cd /home/oracle/scripts
firefox compare_spa_* &

It should look similar to the ones below:

More isn’t always better 😉 Be careful just implementing recommendations. Test and verify them step by step.

You may now [proceed to the next lab](#next).

## Learn More

*(optional - include links to docs, white papers, blogs, etc)*

* [URL text 1](http://docs.oracle.com)
* [URL text 2](http://docs.oracle.com)

## Acknowledgements
* **Author** - <Name, Title, Group>
* **Contributors** -  <Name, Group> -- optional
* **Last Updated By/Date** - <Name, Group, Month Year>
* **Workshop (or Lab) Expiry Date** - <Month Year> -- optional, use this when you are using a Pre-Authorized Request (PAR) URL to an object in Oracle Object Store.

## Need Help?
Please submit feedback or ask for help using our [LiveLabs Support Forum](https://community.oracle.com/tech/developers/categories/livelabsdiscussions). Please click the **Log In** button and login using your Oracle Account. Click the **Ask A Question** button to the left to start a *New Discussion* or *Ask a Question*.  Please include your workshop name and lab name.  You can also include screenshots and attach files.  Engage directly with the author of the workshop.

If you do not have an Oracle Account, click [here](https://profile.oracle.com/myprofile/account/create-account.jspx) to create one.
