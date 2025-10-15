---
layout: post
title: FileMaker to Oracle APEX (Invoices) Part 3
author: Jose (Human)
categories: FileMaker APEX SQL
date: 2025-10-15
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

In the [previous tutorial](https://solutionscay.github.io/filemaker/apex/sql/2025/10/04/filemaker-to-apex-invoices-02.html), we built the Client List and the Client Form for the Invoice Solution. In this post, we'll clean up the Client Form and then create the Client Details page.

# Cleaning up the Client Form

## Planning

First, we'll clean the Client Form. Let's take a look at the original FileMaker Layout. We'll recreate the following elements:

1. The Status pop-up menu with the values `Active` and `Inactive`.
2. The Terms pop-up menu with the values `Net 30`, `Net 45`, and `Net 60` (so they match the data we imported in Part 1).
3. We'll group the Contact Information.
4. We'll group the Main Address.
5. We'll group the Billing Address.
6. And finally, we'll make the **SAME AS MAIN** checkbox field copy all the values from the Main Address to the Billing Address.

![](/assets/img/filemaker_to_apex_invoices/fm_clients_form_01.png)

### About the Billing Address fields

In the FileMaker Invoice Solution, the Billing Address fields use calculated values like:

```c
Case ( 
Billing Same as Main Flag ; Main Address Street ; 
Self 
)
```

Calculations in FileMaker are a nice way of auto-entering information, and in Oracle, we could do something similar with a database Trigger. However, since we want to see the Main Address information get copied over to the Billing Address immediately when we click on the **SAME AS MAIN** checkbox field, we'll use a **Dynamic Action** instead, and let the APEX **Form Processing** take care of saving the values to the table.

## Setting up the Value Lists

First, let's set up the value lists. Edit page 11.

1. Click on **P11_STATUS** and change the **Identification** **Type** to **Select List**.
2. Under **List of Values** change the **Type** to **Static Values**, and click on the box right below it.
3. On the **Static Values** pop up window, set the **Values** to `Active` and `Inactive`. Note that you have different display and return values. The **Display Value** is what the user sees on the list, and the **Return Value** is the value that APEX saves to the table.

![](/assets/img/filemaker_to_apex_invoices/client_form_05.png)

Now repeat the process for the `P11_TERMS` page item. Set the values as `Net 30`, `Net 45`, and `Net 60`.

![](/assets/img/filemaker_to_apex_invoices/client_form_06.png)

## Grouping Page Items with Regions

Now let's group the page items on this form. We'll create three groups:

1. Contact Info
2. Main Address
3. Billing Address

Right-click on the **Client Form** region and click on **Create Sub Region**. Create the following three sub-regions: `Contact Info`, `Main Address`, and `Billing Address`. Then drag and drop the corresponding page items under each region. It should end up looking like this:

![](/assets/img/filemaker_to_apex_invoices/client_form_07.png)

> APEX uses a 12-column grid system. As a FileMaker developer, I know it's hard to let go of the pixel-perfect control afforded by Layout Mode. It's a giant leap, but don't be dismayed. Let go of total control over field placement, and you will have more time to iterate and build faster.

Click on each page item and rename its **Label** to make them user-friendly.

![](/assets/img/filemaker_to_apex_invoices/client_form_08.png)

Place the postal code next to the state by switching on **Start New Row** under **Layout**. Do that for both the Main and Billing Addresses.

![](/assets/img/filemaker_to_apex_invoices/client_form_09.png)

## Configure the Same as Main checkbox

Click on **P11_BILLING_SAME_AS_MAIN_FLAG** and set the **Identification** **Type** to **Checkbox**.

![](/assets/img/filemaker_to_apex_invoices/client_form_10.png)

Then right-click on **P11_BILLING_SAME_AS_MAIN_FLAG** and click on **Create Dynamic Action**. 

1. Set the **Identification** **Name** to `CLICK_SAME_AS_MAIN`
2. Set the **When** **Event** to `Change`, the **Selection Type** to `Item(s)`, and set the item to `P11_BILLING_SAME_AS_MAIN_FLAG`.
3. Set the **Client-side Condition** **Type** to `Item = Value`, the **Item** to `P11_BILLING_SAME_AS_MAIN_FLAG`, and the **Value** to `Y`.

So when the user clicks on **P11_BILLING_SAME_AS_MAIN_FLAG** and the value changes, the **Dynamic Action** will trigger, but only if the value equals `Y` - which is the default truthy value for the checkbox.

![](/assets/img/filemaker_to_apex_invoices/client_form_12.png)

### TRUE Actions

Now right-click on the **True** node and select **Create TRUE Action**. 

On the new Action,

1. Set the **Identification** **Type** to `Execute JavaScript Code`.
2. And under **Settings**, paste the following JavaScript code:

```js
apex.item("P11_BILLING_ADDRESS_STREET").setValue(apex.item("P11_MAIN_ADDRESS_STREET").getValue());
apex.item("P11_BILLING_ADDRESS_CITY").setValue(apex.item("P11_MAIN_ADDRESS_STREET").getValue());
apex.item("P11_BILLING_ADDRESS_STATE").setValue(apex.item("P11_MAIN_ADDRESS_STATE").getValue());
apex.item("P11_BILLING_ADDRESS_POSTAL_CODE").setValue(apex.item("P11_MAIN_ADDRESS_POSTAL_CODE").getValue());
apex.item("P11_BILLING_ADDRESS_COUNTRY").setValue(apex.item("P11_MAIN_ADDRESS_COUNTRY").getValue());
```

This block uses the [item Interface](https://docs.oracle.com/en/database/oracle/apex/24.2/aexjs/item.html) from the APEX JavaScript API to set the billing page items with the values of the main address page items.

![](/assets/img/filemaker_to_apex_invoices/client_form_13.png)

> You can use vanilla JavaScript or jQuery to accomplish the same functionality. The APEX JS API makes it easier and standardized.

### FALSE Actions

Now create a **False** condition.

1. Set the **Identification** **Action** to `Set Value`.
2. And under **Affected Elements**, set the **Selection Type** to `Item(s)`, leave the **Value** blank, and set the `Item(s)` to all the billing address page items:

```txt
P11_BILLING_ADDRESS_STREET,P11_BILLING_ADDRESS_CITY,P11_BILLING_ADDRESS_STATE,P11_BILLING_ADDRESS_POSTAL_CODE,P11_BILLING_ADDRESS_COUNTRY
```
When the user clears the checkbox, APEX sets all billing address page items to empty.

![](/assets/img/filemaker_to_apex_invoices/client_form_14.png)

## Required Fields and Button Styles

Now, let's set all the client form fields to be required. You can apply properties to multiple items at once by multi-selecting them (Ctrl + Click on Windows or Command + Click on Mac). Just be careful that you don't mistakenly override the wrong property. 

There are two settings here:

1. Set the **Appearance** **Template** to `Required - Floating` - this will add a red mark on each page item to indicate the fields are required.
2. Switch on **Value Required** under **Validation** - this will not allow you to save the form if any of the page items are empty.

![](/assets/img/filemaker_to_apex_invoices/client_form_15.png)

One last detail, change the **Delete** **Template Options** **Type** to `Danger`. Now the Delete button will appear red in color.

![](/assets/img/filemaker_to_apex_invoices/client_form_16.png)

## Testing the Client Form

Now run the Client List, edit a client, and try out your new form!

![](/assets/img/filemaker_to_apex_invoices/client_form_same_as_main.gif)


# Creating the Client Details

## Planning the UI

Let's go back to our sketch to visualize what's next. We completed the Clients List and the Client Form. Next, we'll build the Client Details section, which will provide a read-only view of client information and a region for displaying client invoices.

![](/assets/img/filemaker_to_apex_invoices/invoice_clients_sketch_01.png)

Let's plan the page before we start. I drew this sketch to give you the structure of the page, and so you can see the field placement and grouping. We will use nested regions to accomplish this design.

![](/assets/img/filemaker_to_apex_invoices/client_details_03.png)

First, we'll place the **Company Name**, **Status**, and **Terms** horizontally directly under the Client Form region.

Second, we'll create the following nested sub regions:

* Contact Info
    * Contact Name
    * Phone and Email
* Address
    * Main Address
    * Billing Address

Third, we'll rearrange the page items under their corresponding regions.

And finally, we'll tweak the region's **Template Options** to hide unnecessary headers and styles.

We'll come back to the Invoices in a later lesson. Let's do it.

## Building the Page

Create a new **Blank Page**.

![](/assets/img/filemaker_to_apex_invoices/client_details_01.png)

 Set the **Page Number** to **12**, the **Name** to **Client Details**, the **Page Mode** to **Normal**, and disable **Use Navigation** - we will only access this page from the Client List.

![](/assets/img/filemaker_to_apex_invoices/client_details_02.png)

### Setting up the Form Region and Sub Regions

Create a Form region and set the Source to the `Clients` table.

![](/assets/img/filemaker_to_apex_invoices/client_details_04.png)

Now, create the nested sub-regions and assign their proper placement using the **Layout** **Start New Row** and **New Column** switches.

![](/assets/img/filemaker_to_apex_invoices/client_details_05.png)

## Moving Page Items and Label Revisions

Move all the page items to their corresponding regions and configure their placement. It should end up looking like this:

![](/assets/img/filemaker_to_apex_invoices/client_details_06.png)

Go through each region and revise the **Template Options** under **Appearance** and set them up accordingly.

| Region         | Header           | Style                 |
|----------------|------------------|-----------------------|
| Client Details | Hidden           | Remove UI Decoration  |
| Contact Info   | Visible - Default| Default               |
| Contact Name   | Hidden           | Remove UI Decoration  |
| Phone and Email| Hidden           | Remove UI Decoration  |
| Address        | Hidden           | Remove UI Decoration  |
| Main Address   | Visible - Default| Default               |
| Billing Address| Visible - Default| Default               |

Revise all the labels, make them user-friendly. And when you run the page, you should have a nice-looking layout like this:

## Add the View link to the Button List

Now let's add a link to the Client List so you can open the Client Details page.

1. Right-click on the **Client List** **Columns** node and select **Create Virtual Column**.
2. Set the **Identification** **Type** to `Link`.
3. Click on the **Link** **Target**, and on the **Link Builder**, set the **Type** to `Page in this Application`, set the **Page** to `12`, and set the item `P12_CLIENT_ID` to the value `#CLIENT_ID`.
4. Finally, set the **Link Text** to a Universal Theme icon like `<span class="fa fa-expand" aria-hidden="true"></span>`.

![](/assets/img/filemaker_to_apex_invoices/client_details_08.png)

Run the Clients List, and you should see your new link.

![](/assets/img/filemaker_to_apex_invoices/client_details_09.png)

Click on the new link to behold your new Client Details page.

![](/assets/img/filemaker_to_apex_invoices/client_details_07.png)

# Adding the New and Edit buttons to the Client Details

Let's add some buttons to edit and create new clients from the new page. The Client Form modal drawer already handles creating and editing clients, so our buttons will call that page. But we need to make one minor adjustment to the Client Form first.

When the Client Form dialog is closed, we need to pass back the affected client's primary ID so we can control the display of information on the Client Details.

Edit Page 11, navigate to **Dynamic Actions** (🔁), click on the **Close Dialog** process, and under **Settings**, set the **Items to Submit** to `P11_CLIENT_ID`.

![](/assets/img/filemaker_to_apex_invoices/client_details_10.png)

Now edit page 12.

Create two new buttons under the **Breadcrumb** region using the following settings:

| Name | Label | Slot | Buttom Template | Icon |
| `EDIT` | `Edit` | **Edit** | **Text with Icon** |  `fa-user-edit` |
| `NEW` | `New` | **Create** | **Text with Icon** |  `fa-plus-square-o` |

Then, for each button, create a **Dynamic Action** with the following settings:

| Name | Event | Selection Type | Button |
| `CLOSE_EDIT_DIALOG` | **Dialog Closed** | **Button** | EDIT |
| `NEW` | **Dialog Closed** | **Button** | NEW |

Both dynamic actions will behave the same. Create a **True Condition** for each with the following settings:

**TRUE Action 1**

* Identification
    * Action: **Set Value**
* Settings
    * Set Type: **Dialog Return Item**
    * Return Item: `P11_CLIENT_ID`
* Affected Elements
    * Selection Type: **Item(s)**
    * Item(s): `P12_CLIENT_ID`

**TRUE Action 2**

* Identification
    * Action: **Submit Page**

There's one issue with this approach. If you delete the client from the Client Form (Page 11), then **Initialize form Client Details** on the Client Details (Page 12) will throw an error because the record no longer exists. To prevent this, create a **Pre-Rendering** -> **Before Header** Branch with the following settings:

* Identification
    * Name: `CLIENT_DELETED`
* Behavior
    * Type: **Page or URL (Redirect)**
    * Target: `10`
* Server-Side Condition
    * Type: **No Rows returned**
    * SQL Query: `select 1 from clients where client_id = :P12_CLIENT_ID`

If the client no longer exists by the time APEX starts loading the page, it will redirect the user back to the Client List.

Here's the entire process.

1. When the Client Form is closed (whether it's opened from the Edit or New buttons), the corresponding **Dynamic Action** will trigger.
2. The **Set Value** TRUE Action fires up, receiving the affected primary ID (CLIENT_ID) and setting it in the P12_CLIENT_ID page item.
3. The **Submit Page** TRUE Action fires up next and submits the page
4. The page reloads, and the **CLIENT_DELETED** branch checks of the client ID returned by the Client Form exist in the database. If it does not exist, APEX redirects the user to the Clients List.
5. If the client ID still exists, the **Initialize form Client Details** process runs and sets the Client Form region with the latest values from the table.

![](/assets/img/filemaker_to_apex_invoices/client_details_11.png)

Great. Now run your Clients List, click on the **View** icon, and try out your new Client Details page! Create new clients, edit and delete them.

![](/assets/img/filemaker_to_apex_invoices/client_details_test.gif)

# Conclusion

Excellent work. I hope you are getting the hang of Oracle APEX and starting to see its benefits. In the next session, we'll create the Products List and the Product Form. Following this, we'll add the Invoices List and the Invoice Form, where you can add Line Items. And we'll wrap it up by adding an Invoices List region to the Client Details.