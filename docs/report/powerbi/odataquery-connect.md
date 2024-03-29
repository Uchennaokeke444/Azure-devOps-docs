---
title: Connect with data by using OData queries
titleSuffix: Azure DevOps
description: Learn how to write and test OData queries for use in Power BI integration.
ms.subservice: azure-devops-analytics
ms.author: chcomley
author: chcomley
ms.topic: tutorial
monikerRange: '>= azure-devops-2019'
ms.date: 12/09/2022
---

# Connect with data by using Power BI and OData queries

[!INCLUDE [version-gt-eq-2019](../../includes/version-gt-eq-2019.md)]

Using OData queries is the recommended approach for pulling data into Power BI. OData (Open Data Protocol) is an ISO/IEC approved, OASIS standard that defines best practices for building and consuming REST APIs. For more information, see [OData documentation](/odata/).

To get started quickly, check out the [Overview of sample reports that use OData queries](sample-odata-overview.md). For information about other approaches, see [Power BI integration overview](overview.md). 

Power BI can run OData queries, which can return a filtered or aggregated set of data to Power BI. OData queries have two advantages: 
* All filtering is done server-side. Only the data you need is returned, which leads to shorter refresh times.
* You can pre-aggregate data server-side. An OData query can carry out aggregations such as work item rollup and build failure rates. The aggregations are accomplished server-side, and only the aggregate values are returned to Power BI. With pre-aggregation, you can carry out aggregations across large data sets, without needing to pull all the detail data into Power BI.

In this article, learn how to:

> [!div class="checklist"]
> * Write and test OData queries
> * Run an OData query from Power BI

[!INCLUDE [prerequisites-simple](../includes/analytics-prerequisites-simple.md)]

## Use Visual Studio Code to write and test OData queries

The easiest way to write and test OData is to use [Visual Studio Code](https://aka.ms/vscode) with the [OData extension](https://marketplace.visualstudio.com/items?itemName=stansw.vscode-odata). Visual Studio Code is a free code editor available on Windows, Mac, and Linux. The OData extension provides syntax highlighting and other functions that are useful for writing and testing queries. 

### Install Visual Studio Code and OData extension
 
1. Install [Visual Studio Code](https://aka.ms/vscode).

2. Open Visual Studio Code, select **Extensions**, and then search for *odata*. In the results list, select **vscode-odata**, and then install it.

3. Create and save an OData file in Visual Studio Code, for example, `filename.odata`. Name it whatever you want, but it must have a `.odata` extension to enable the OData extension functionality.

### Write the OData query

1. Write the OData query. For example queries, review the [Overview of sample reports using OData queries](sample-odata-overview.md). 

The following query returns the top 10 work items under a specific area path. Replace {organization}, {project}, and {area path} with your values.

```
https://analytics.dev.azure.com/{organization}/{project}/_odata/v3.0-preview/WorkItems?
    $select=WorkItemId,Title,WorkItemType,State,CreatedDate
    &$filter=startswith(Area/AreaPath,'{area path}')
    &$orderby=CreatedDate desc
    &$top=10
``` 
To query across projects, omit `/{project}` entirely. 

For more information, see [OData query quick reference](../extend-analytics/quick-ref.md). 

After you write the query in Visual Studio Code, you should see syntax highlighting.

![Visual Studio Code OData extension, syntax highlighting example.](media/odataquery-syntaxhighlighting.png)

### Test the OData query

1. To test the OData query, place your cursor anywhere in the query text and select **View** > **Command Palette**. 
2. In the search box, enter **odata** to bring up all the OData commands.

   :::image type="content" source="media/odataquery-commandpallette.png" alt-text="Screenshot of Visual Studio Code OData extension, Command Palette.":::

3. Select **OData: Open**. This action combines the multiline query into a one-line URL and opens it in your default browser. 

- The OData query result set is in JSON format. To view the results, install the JSON Formatter extension for your browser. Several options are available for both Chrome and Microsoft Edge.

   :::image type="content" source="media/odataquery-jsonoutput.png" alt-text="Screenshot of Visual Studio Code OData extension, JSON output.":::

- If the query has an error, the Analytics service returns an error in JSON format. For example, this error states that the query selected a field that doesn't exist.

   :::image type="content" source="media/odataquery-jsonerror.png" alt-text="Screenshot of Visual Studio Code OData extension, JSON error.":::

Once you verify that the query works correctly, then you can run it from Power BI.

## Run the OData query from Power BI

### Combine the multiline OData query into a single-line query

Before you use the query in Power BI, you must convert the multiline OData query into a single-line query. The simplest way to do so is to use [Visual Studio Code](https://aka.ms/vscode) with the [OData extension](https://marketplace.visualstudio.com/items?itemName=stansw.vscode-odata) and use the **OData: Combine** command.

> [!NOTE]
> In your *filename.odata* file, you might want to first create a copy of the multiline query text and then run **OData: Combine** on the copy. Do this because there's no way to convert the single-line query back to a readable multiline query. 

1. In Visual Studio Code, place your query anywhere in the query text, and then select **View** > **Command Palette**. In the search box, enter **odata** and then, in the results list, select **OData: Combine**.

The multiline query gets converted into a single-line query.

:::image type="content" source="media/odataquery-combineto1line.png" alt-text="Screenshot of Visual Studio Code OData extension, Combine to single-line query.":::

2. Copy the entire line for use in the next section.

### Run the query from Power BI

1. Select **Get Data** > **OData feed**. For more information, see [Create a Power BI report with an OData query](create-quick-report-odataq.md).

   :::image type="content" source="media/ODataQuery.png" alt-text="Screenshot of Power BI OData feed command.":::

2. In the **OData feed** window, in the **URL** box, paste the OData query that you copied in the preceding section, and then select **OK**.

   :::image type="content" source="media/odataquery-powerbi-odatafeed.png" alt-text=".":::

   Power BI displays a preview page.

   :::image type="content" source="media/odataquery-powerbi-preview.png" alt-text="Screenshot of Power BI OData Feed Data Preview page.":::

### Specify query options

1. Select **Edit** on the preview page to open the Power Query Editor.

   :::image type="content" source="media/odataquery-powerbi-queryeditor.png" alt-text="Screenshot of Power BI - OData Feed - Power Query Editor.":::

2. Select **Advanced Editor** in the ribbon.

   :::image type="content" source="media/AdvancedEditor.png" alt-text="Screenshot of Power BI - OData Feed - Select Advanced Editor.":::

3. Scroll horizontally to view the `[Implementation="2.0"]` parameter in the **Query** pane.

   :::image type="content" source="media/odataquery-powerbi-advancededitor1.png" alt-text="Screenshot of Power BI OData Feed Advanced Editor - scrolled right.":::

4. Replace `[Implementation="2.0"]` with the following string:

   `[Implementation="2.0",OmitValues = ODataOmitValues.Nulls,ODataVersion = 4]` 

   :::image type="content" source="media/odataquery-powerbi-advancededitor2.png" alt-text="Screenshot of replacement string.":::

> [!NOTE]
> To prevent throttling errors, do the following actions:
> - Instruct Power BI to reference OData v4.
> - Instruct the Analytics service to omit any values that are null, which improves query performance. 
> Power Query attempts to resolve null values as errors by generating an additional query for every null value it encounters. This can result in thousands of queries, which quickly exceeds your usage threshold, beyond which your user account gets throttled.

The following action is required for Power BI to successfully run an OData query against the Azure DevOps Analytics Service.

5. Select **OK** to close the Advanced Editor and return to the Power BI Power Query Editor. You can use Power Query Editor to perform the following optional actions:  
   - Rename the "Query1" query as something more specific
   - Transform columns to a specific type. Power BI autodetects the type, but you might want to convert column to a specific data type 
   - Add computed columns
   - Remove columns
   - Expand columns into specific fields

### Create a report by using the data

Select **Close & Apply** to save your settings and pull the data into Power BI. Once the data refreshes, you can create a report as you would normally in Power BI.

:::image type="content" source="media/transform-data/powerbi-close-apply.png" alt-text="Screenshot of Power BI Close and Apply button.":::

## Related articles

- [Sample Power BI Reports by using OData queries](sample-odata-overview.md)
- [Data available from Analytics](data-available-in-analytics.md)
- [Grant permissions for access to Analytics](./analytics-security.md)
- [Power BI integration overview](overview.md)
