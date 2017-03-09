
pe:gridview and related tags(pe:pager, pe:bindingblock) are used for displaying a long list of templated data with paging. Such as friends list, application list, etc. 
A recurring task in software development is to display tabular data. With the GridView control, you can display, edit, and delete data 
from many different kinds of data sources, including databases, XML files, NPL tables and business objects that expose data.

Using the GridView you can:
   * Automatically bind to and display data from a data source control.
   * Select, sort, page through, edit, and delete data from a data source control.
Additionally, you can customize the appearance and behavior of the GridView control by:
   * Specifying custom columns and styles.
   * Utilizing templates to create custom user interface (UI) elements.

Grid view sample code: 

```xml
<pe:gridview style="margin:10px" name="gvwTableTest" CellPadding="5"  AllowPaging="True" DefaultNodeHeight = "20" pagesize="10"
        DataSource='<%={{username="LiXizhi", Company="ParaEngine"}, {username="Andy", Company="ParaEngine"}, {username="Gandhi", Company="PE"}}%>'
        DataBound = "gvwTableTest_DataBound()" OnPageIndexChanging="gvwTableTest_PageIndexChanging()" OnPageIndexChanged="gvwTableTest_PageIndexChanged()">
	<Columns>
		Row index is <%=Eval("index")%><a href='<%="profile?name="..Eval("username")%>'>User Name is <%=Eval("username")%></a>Company: <%=Eval("Company")%>
	</Columns>
	<EmptyDataTemplate>
		<b>NO MATCHING USER IS FOUND 没有找到符合要求的用户</b>
	</EmptyDataTemplate>
	<FetchingDataTemplate>
		<b>Please wait while fetching data</b>
	</FetchingDataTemplate>
	<PagerSettings Position="TopAndBottom" height="26" PreviousPageText="" NextPageText=""/>
	<!--<PagerTemplate><form><input type="button" name="pre" value="previous page"/><input type="button" name="next" value="next page"/><label name="page" style="height:18px;margin:4px"/></form></PagerTemplate>-->
</pe:gridview>
```

### Data Binding with the GridView Control
The GridView control provides you with two choices for binding to data:
   * Data binding using the DataSourceID property, which allows you to bind the GridView control to a data source control. 
This is a simple approach because it allows the GridView control to take advantage of the capabilities of the data 
source control and provide built-in functionality for sorting, paging, and updating. 
   * Data binding using the DataSource property, which allows you to bind to various objects, including NPL array table, NPL function(nRowIndex) (see below). 
This approach requires you to write code for any additional functionality such as paging, and updating. 
However, using NPL function is the recommended way if your data is dynamically and asynchronously loaded such as from a web server. 

If you bind to an data source `function(nRowIndex) end`, the function must return value with following rules: 
   * if nRowIndex is number, it returns the fields in a table of the given row index. 
   * if nRowIndex is nil, it returns totalItems. One can return nil as totalItems, which tells that control that data is not available, possibly still fetching from web services.

The following is pseudo code of your datasource function. 
```lua
MyPage.ds = nil;
function MyPage.DS_Func_MyRemote(index) 
	if(MyPage.ds) then
		-- if we already have local data, provide it
		if(not index) then
			-- return the number of items
			return #(MyPage.ds); 
		else
			return MyPage.ds[index];
		end
	else
		-- data is not available, we will return nil and fetch the data. This will only be called once.
		-- we will use a timer to simulate remote call. 
		commonlib.TimerManager.SetTimeout(
			function()  
				MyPage.ds = { {data=1} , {data=2} };
				-- when data is available refresh the page again or just call DataBind method of the control.
				MyPage:RefreshPage();
			end, 1000); 
	end
end
```

### Formatting Data Display in the GridView Control
You can specify the layout of the GridView control's rows by using any other MCML tags inside the Columns node as shown in the grid view example above. 
Inside Columns node, use embedded script block with Eval() or XPath() method to extract data from columns of the current row. 
| ItemsPerLine | it is a pe:gridview property which controls how many items to show on one line. Default value is 1. One can set it to a bigger value to show 
| LineWidth | width of a line in pixel. it is usually same as the grid view width. |
| LineAlign | it can only be "center". if it is "center" and ItemsPerLine is lager than 1, the columns will be centered, especially for the last row. LineWidth must also be specified.  |
| ClickThrough | wheather mouse event will leak to 3d scene. default to false  |
| | a grid of databind items, where each items is customizable via the columns property. |
| RememberScrollPos | true to remember scroll position within current page. |
| ScrollBarTrackWidth | |
| VerticalScrollBarWidth | |
| RememberLastPage | true to remember last opened page during page refresh |
| AllowPaging | true to allow paging |
| ClickThrough | whether mouse event will leak to 3d scene. default to false  |
| PagerTemplate.AutoHidePager | boolean. |
| FitHeight | if true, the height of the control will best contain the data nodes. css.min-height can also be specified. |

### GridView Paging Functionality
A pager row is displayed in a GridView control when the paging feature is enabled (when the AllowPaging property is set to true). 
The pager row contains the controls that allow the user to navigate to the different pages in the control. 
Instead of using the built-in pager row user interface (UI), you can define your own UI by using the PagerTemplate property.

To specify a custom template for the pager row, first place PagerTemplate followed by a form tags between the opening and closing tags of the GridView control. 
You can then list the contents of the template between the opening and closing PagerTemplate tags. If any buttons are named "pre" and "next" in the content, they are automatically
bounded to command action previous page and next page. If any text or label are named "page", it will be used to display current page number/page count. Here is an example
```
	<PagerTemplate><form><input type="button" name="pre" value="previous page"/><input type="button" name="next" value="next page"/><label name="page" style="height:18px;margin:4px"/></form></PagerTemplate>
```

one can also apply css style to PageSettings, like below.
```
	<PagerSettings style="margin-left:300px" Position="Bottom"/>
```

### GridView Events
You can customize the functionality of the GridView control by handling events. The GridView control provides events that occur both before and after a navigation. 
   * PageIndexChanging(gridviewName, pageindex): Occurs when a pager button is clicked, but before the GridView control performs the paging operation. This event is often handled to cancel the paging operation. 
   * PageIndexChanged(gridviewName, pageindex): Occurs when a pager button is clicked, but after the GridView control performs the paging operation. This event is commonly handled when you need to perform a task after the user navigates to a different page in the control. 
   * DataBound(gridviewName, datasource): Occurs after the GridView control has finished binding to the data source. 

use the lib:
-------------------------------------------------------
NPL.load("(gl)script/kids/3DMapSystemApp/mcml/pe_gridview.lua");
-------------------------------------------------------