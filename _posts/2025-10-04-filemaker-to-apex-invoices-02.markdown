---
layout: post
title: FileMaker to Oracle APEX (Invoices) Part 2
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

# Planning the UI

Before creating the application in APEX, let's sketch a rough plan for the UI. 

We'll start with Clients.

The List View is straightforward. We'll copy it as closely as possible.

![](/assets/img/filemaker_to_apex_invoices/fm_clients_list.png)

For the Form View, I'll remove the Master Detail functionality. Instead, we'll use a more common pattern where you can create and edit records using a Modal Form, and we'll have a separate full-page view for Client Details.

![](/assets/img/filemaker_to_apex_invoices/fm_clients_form.png)

> APEX provides a Master Detail component using **Interactive Grid** that we can use to create the Client Form View, similar to FileMaker. But I would rather spend our time now learning more basic components and patterns.

So, instead of two pages, we're creating three.

1. Clients List  
2. Clients Form (Modal Drawer)
3. Client Details  

![](/assets/img/filemaker_to_apex_invoices/invoice_clients_sketch.png)

We'll build the Client List and Client Form in this tutorial, and we'll revisit Client Details later.

Let's do it. 

# Creating the Clients List

Log in to the App Builder and click **Create**.

![](/assets/img/filemaker_to_apex_invoices/create_new_app.png)

Click **Create a New App**.

![](/assets/img/filemaker_to_apex_invoices/create_new_app_01.png)

Name the application **Invoice Solution** and leave the ID alone.

![](/assets/img/filemaker_to_apex_invoices/create_new_app_02.png)

Now you've got a new app. Click **Create Page**.

![](/assets/img/filemaker_to_apex_invoices/create_new_page_01.png)

Let's create the Client Form first, so when we make the list, we can immediately hook up the **Create Client** and **Edit Client** buttons. Select the **Form** Component and click **Next**.

![](/assets/img/filemaker_to_apex_invoices/client_form.png)

> In APEX, the **Form** component provides out-of-the-box CRUD functionality.

Set the **Page Number** to **11**, the **Name** to **Client Form**, and the **Page Mode** to **Drawer**. For the **Data Source**, set the **Table/View Name** to `CLIENTS`, and click **Next**.

![](/assets/img/filemaker_to_apex_invoices/client_form_01.png)

APEX will recognize your Primary Key Column as `CLIENT_ID`, leave it as is, and click **Create Page**.

![](/assets/img/filemaker_to_apex_invoices/client_form_02.png)

Expand the **Client Form** to see all the columns. The only change we'll make right now is to set the Audit Columns (`created_on`, `created_on`, `created_on`, and `created_on`) **Identification Type** to **Hidden**. These columns are auto-entered by a Trigger, so there's no need to expose them in the form.

![](/assets/img/filemaker_to_apex_invoices/client_form_03.png)

# Creating the Client Form

Let's create the next page. Click on the **+** drop-down button on the upper right and then click **Page**  

![](/assets/img/filemaker_to_apex_invoices/client_form_04.png)

This time, select the **Classic Report** Component and click **Next**.

![](/assets/img/filemaker_to_apex_invoices/clients_classic_report.png)

> In APEX, a **Classic Report** provides list report functionality. It displays each record as a row, includes out-of-the-box functionality such as sorting and grouping, and is highly customizable.

Set the **Page Number** to **10**, the **Name** to **Clients List**, and the **Data Source Table/View Name** to `CLIENTS`. Leave the Navigation alone, but feel free to change the page icon. Click **Create Page**.

![](/assets/img/filemaker_to_apex_invoices/clients_classic_report_01.png)

Now let's add a button to open the **Client Form** Page so we can start creating records.

1. Right-click on the Breadcrumb region and click **Create Button**.
2. Set the **Button Name** to `NEW_CLIENT` and the Label to **New Client**.
3. Set the **Slot** to **Create**.
4. Set the **Behavior Action** to **Redirect to Page in this Application**, and click on the **Target** button below.
5. On the **Link Builder - Target** window, set the Page to 11 (Client Form). Click **Ok** to dismiss the window.

![](/assets/img/filemaker_to_apex_invoices/clients_classic_report_02.png)

# Refreshing the Client List with a Dynamic Action

Let's add a Dynamic Action to refresh the Client List after creating a new record from the Client Form Drawer.

Click on the Dynamic Action pane (⚡). Right-click on **Dialog Closed** and click on **Create new Dynamic Action**. Set the **Identification Name** to `NEW_CLIENT_CLOSED`. Set the **When Event** to **Dialog Closed**, the **Selection Type** to **Button**, and select the **NEW_CLIENT** button from the drop-down list.

![](/assets/img/filemaker_to_apex_invoices/clients_classic_report_03.png)

Then, click on the **True** Action and set the **Identification Action** to **Refresh**. Set the **Selection Type** under **Affected Elements** to **Region**, and select the **Clients List** region from the drop-down list.

![](/assets/img/filemaker_to_apex_invoices/clients_classic_report_04.png)

Now, for the big reveal, run the application by clicking the green Play button located in the upper-right corner.

![](/assets/img/filemaker_to_apex_invoices/clients_classic_report_05.png)

# Creating the first record

Beautiful! Click on the **New Client** button to test the Client Form.

![](/assets/img/filemaker_to_apex_invoices/clients_classic_report_10.png)

Fill the form and click the **Create** button.

![](/assets/img/filemaker_to_apex_invoices/clients_classic_report_11.png)

Congratulations, you signed up your first client!

![](/assets/img/filemaker_to_apex_invoices/clients_classic_report_12.png)

# Customizing the Clients List

Now, let's clean up the Client List so it resembles the FileMaker version.

Go back to the **Clients List** region and change the **Source** **Type** to **SQL Query**. This method of setting up the data source is a powerful feature because it provides complete control over the query that powers the report region. You'll see in the next step.

![](/assets/img/filemaker_to_apex_invoices/clients_classic_report_13.png)

Do not remove the `CLIENT_ID`; it is set to **Hidden** and APEX will not display it on the report. However, we need it so we can pass it to the **Client Form** drawer when editing a record. Remove all the columns we don't need. Concatenate the `FIRST_NAME` with the `LAST_NAME` to display the client's full name.

![](/assets/img/filemaker_to_apex_invoices/clients_classic_report_14.png)

> In Oracle SQL, you concatenate strings using double pipes (`||`) instead of the FileMaker `&` operator. And you declare strings within single quotes instead of double quotes.

Here is the SQL code, in case you would like to copy it.

```sql
select 
  CLIENT_ID,
  NAME,
  CONTACT_FIRST_NAME || ' ' || CONTACT_LAST_NAME as contact,
  CONTACT_PHONE,
  STATUS,
  MAIN_ADDRESS_CITY,
  MAIN_ADDRESS_STATE
from CLIENTS
```
Now, modify all the column **Heading** label names so they match the names used in the FileMaker Layout.

Click on each column and revise the **Heading**.

![](/assets/img/filemaker_to_apex_invoices/clients_classic_report_15.png)

# Adding the Edit button

One last thing here, let's add an edit button to each client row so the user can modify the records.

1. Right-click on columns and click on **Create Virtual Column**.
2. Set the **Identification** **Type** to **Link**.
3. Click on the **Target** button. Set the **Target** **Page** to **11**. Under **Set Items**, set `P11_CLIENT_ID` to `#CLIENT_ID#`, so when clicking on the Link, APEX will pass the value of the `client_id` column into the Client Form `P11_CLIENT_ID` Page Item, which in turn will initialize the Form region and load the client data.
4. Set the link text `<span class="fa fa-edit" aria-hidden="true"></span>`. You can find all the APEX icons [here](https://oracleapex.com/ords/r/apex_pm/ut/icons).

![](/assets/img/filemaker_to_apex_invoices/clients_classic_report_16.png)

Let's duplicate the `NEW_CLIENT_CLOSED` Dynamic Action and name it **EDIT_CLIENT_CLOSED**. Change the **When** **Event** to **Dialog Closed**, the **Selection Type** to **Region**, and the **Region** to **Clients List**.

![](/assets/img/filemaker_to_apex_invoices/clients_classic_report_17.png)

Now rerun the app and behold the beauty of the Universal Theme!

![](/assets/img/filemaker_to_apex_invoices/clients_classic_report_18.png)

# Inserting Sample Data

Let's get some data in there. Go to your LLM of choice and prompt it something like this:
> 
> Given this Quick SQL definition:
> 
> ```
> clients
>   name vc225
>   contact_first_name vc225
>   contact_last_name vc225
>   contact_phone vc225
>   contact_email vc225
>   status vc225
>   main_address_street vc225
>   main_address_city vc225
>   main_address_state vc225
>   main_address_postal_code vc225
>   billing_same_as_main_flag vc225
>   billing_address_street vc225
>   billing_address_city vc225
>   billing_address_state vc225
>   billing_address_postal_code vc225
>   billing_address_country vc225
>   main_address_country vc225
>   terms vc225
> ```
> 
> Write an Oracle INSERT statement for 20 clients. For the values, use SELECT statements from DUAL and UNION ALL. The company names are IT companies that develop FileMaker and Oracle APEX solutions.

Here is my copy: [client_data.txt](/assets/source/filemaker_to_apex_invoices/client_data.txt).

Go to SQL Workshop -> SQL Commands and paste the SQL statement there, then click on **Run**.

![](/assets/img/filemaker_to_apex_invoices/clients_data.png)

Congratulations! You went from one client to 21 in a matter of seconds. The power of AI is astounding.

![](/assets/img/filemaker_to_apex_invoices/clients_classic_report_19.png)

# Enhancing with Smart Filters

Now let's wrap up this page by adding a **Smart Filters** region with a Search field and a Status chip.

### Search bar

1. Right-click on the **Breadcrumb** region and click **Create Sub Region**.
2. Click on the **New** region and change the **Type** to **Smart Filters**, and the **Slot** to **Search Field and Smart Filters**. 
3. Then cllick on the **Template Options** under **Appearance**, and set the **Header** to **Hidden**, and the **Style** to **Remove UI Decoration**.

![](/assets/img/filemaker_to_apex_invoices/clients_classic_report_20.png)

### Status Chip

1. Right-click on **Filters** and click on **Create Filter**
2. Under **Identification**, set **Name** to `P10_STATUS` and the **Type** to **Checkbox Group**.
3. Under **List of Values**, set the **Type** to **Distinct Values**

The filter uses the **Identification** **Name** to determine the name of the column. If you scroll down the **Filter** pane, you will see under **Source** that APEX automatically sets the **Database Column** to `STATUS`.

![](/assets/img/filemaker_to_apex_invoices/clients_classic_report_21.png)

Run the page and try it out! Perform a search. Check out the Status filter chip.

![](/assets/img/filemaker_to_apex_invoices/clients_classic_report_25.png)

# Conclusion

In the next session, we'll clean up the Client Form and add the Client Details page.