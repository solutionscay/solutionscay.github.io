---
layout: post
title: FileMaker to Oracle APEX (Invoices) Part 1
author: Jose (Human)
categories: FileMaker APEX SQL
---

<style>
img {
  display: block;
  max-width: 100%;
  height: auto;
  margin: 2rem auto;
  padding: 10px;
  background-color: #fff;
  border: 1px solid #e0e0e0;
  border-radius: 12px;
  box-shadow:
    0 1px 2px rgba(0, 0, 0, 0.04),
    0 4px 12px rgba(0, 0, 0, 0.06);
  transition: box-shadow 0.3s ease, transform 0.3s ease;
}
</style>

# Introduction

The first step in converting your FileMaker solution to Oracle APEX is to migrate the database. For this tutorial, I am using the Invoice Solutions from the Claris Marketplace. You can download it here: [https://marketplace.claris.com/detail/2000.html](https://marketplace.claris.com/detail/2000.html).

We are going to migrate the following five tables:

1. Clients
2. Staff
3. Products
4. Invoices
5. Line Items

# Understanding the Data Model

From the Relationships Graph, we see that one Client and Staff can have one or many Invoices. Each Invoice can have one or many Line Items. And each Line Item has one Product. Pretty simple model.

![](/assets/img/filemaker_to_apex_invoices/fm_invoices_erd.png)

# Generating Quick SQL from FileMaker

To quickly get you started, we are going to use a feature in Oracle APEX called [Quick SQL](https://docs.oracle.com/en/database/oracle/apex/24.2/aeutl/using-quick-sql.html#GUID-21EE36C2-F814-48C0-90EA-7D464E9014FD). It is basically a shorthand form of writing the SQL Data Definitions for your tables, and it will save you a lot of time.

First, create new layouts for each of the five tables with the fields we will migrate to Oracle. Do not include primary keys or foreign keys, calculation or summary fields, or creation and modification usernames and timestamps. We will let Quick SQL create these columns for us.

![](/assets/img/filemaker_to_apex_invoices/filemaker_field_layout_01.png)

Then, run the following Calculation from the Data Viewer:

```c
Let ( [
    fields = FieldNames ( Get ( FileName ) ; Get ( LayoutName ) );
    field_count = ValueCount ( fields )
];
    // Output the table name
    Substitute(Lower( Get ( LayoutTableName ) ); " "; "_") & ¶ & 
    // Loop to build the fields list
    While (
        // Initialize the fields list
        [counter = 1 ; result = "   " ] ;
        counter ≤ field_count ; [ 
        result = result & 
        // Get the field name
        Lower(Substitute(GetValue(fields;counter);" ";"_")) & " " & 
        // Append the field type
        Substitute( 
            Lower(MiddleWords ( FieldType(Get(FileName);GetValue(fields;counter)); 2 ; 1)) ;
            // Convert FileMaker types to Oracle
            ["text";"vc225"];["container";"blob"]
        ) & 
        // Separate with return
        "¶   " ; 
        // Increase the counter
        counter = counter + 1 ];
        // Boom
        result
    )
)
```
The Calculation will produce the Quick SQL text for the Layout's table with all the fields placed on the Layout:

![](/assets/img/filemaker_to_apex_invoices/filemaker_custom_function_quicksql.png)

Copy each output to a text file so it's easier to import everything into Quick SQL.

Please note we are defaulting all text fields to `Varchar2(225)`. In a real scenario, we would take the Quick SQL output and modify the lengths accordingly. However, for this tutorial, I will leave them as 225.

Now log in to the Oracle APEX App Builder and navigate to SQL Workshop -> Utilities -> Quick SQL.

![](/assets/img/filemaker_to_apex_invoices/quicksql_menu.png)

# Configuring Quick SQL Settings in Oracle APEX

Before we get started, let's configure some Settings.

![](/assets/img/filemaker_to_apex_invoices/quicksql_settings.png)

Switch on **Add Primary Key**, use SYS_GUID for the Population Method, and switch on **Prefix primary keys with table name.**

![](/assets/img/filemaker_to_apex_invoices/quicksql_settings_01.png)

We prefix the primary keys with the table name, so for example, we will get `client_id`, `staff_id`, `invoice_id`, etc. This naming convention makes it easier to write SQL queries than if all primary keys were named `id` (or `PrimaryKey`).

Next, check **Include Audit Columns**:

![](/assets/img/filemaker_to_apex_invoices/quicksql_settings_02.png)

You can name your created/updated audit columns however you like. I personally like `created_on`, `created_by`, `updated_on`, and `updated_by`.

![](/assets/img/filemaker_to_apex_invoices/quicksql_settings_03.png)

# Tweaking the Quick SQL

Now paste the Quick SQL produced by the FileMaker calculation. Take a moment to add the foreign keys and fix any naming issues.

![](/assets/img/filemaker_to_apex_invoices/quicksql_01.png)

For example, since one Invoice will have one Client and one Staff, we need to add the foreign keys `client_id` and `staff_id`. You do not have to specify the data type; Quick SQL will automatically generate the columns with the proper data types and Constraints.

Also, rename the Invoices `date` field to `invoice_date`. Date is a reserved name in SQL.

Here is the full [Quick SQL file](/assets/source/filemaker_to_apex_invoices/fm_invoice_quicksql.txt).

When done, click **Review and Run**. 

# Running the Quick SQL Script

On the next screen, click **Create** to save the SQL DDL Script. Name it **FM Invoices**. Then click **Run** when you are ready.

![](/assets/img/filemaker_to_apex_invoices/sql_01.png)

The next screen is a confirmation prompt. Click **Run Now.**

![](/assets/img/filemaker_to_apex_invoices/sql_02.png)

The last screen displays a summary of all the tables and objects that have been created. And if you have any errors, you will see them here.

![](/assets/img/filemaker_to_apex_invoices/sql_03.png)

Finally, navigate to SQL Workshop -> Object Browser to view all your newly created Tables, Indexes, and Triggers.

![](/assets/img/filemaker_to_apex_invoices/object_browser_01.png)

In the following tutorial, I will guide you through the process of creating an APEX app using these new tables.