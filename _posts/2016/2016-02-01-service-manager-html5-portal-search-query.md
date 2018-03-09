---
id: 435
title: 'Service Manager HTML5 Portal: Search Query!'
date: 2016-02-01T18:50:03+00:00
author: Ryan Ephgrave
layout: post
guid: http://www.ephingadmin.com/?p=435
permalink: /service-manager-html5-portal-search-query/
categories:
  - HTML5 Portal
  - SCSM
tags:
  - HTML5
  - SCSM
  - Service Manager Portal
---
One of the issues I have with the HTML5 portal is there isn’t a great way to do search with the query box. You can add a text box above the query box as required to do your searches, but the way it is laid out isn’t that great:

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/02/image.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/02/image_thumb.png" alt="image" width="644" height="252" border="0" /></a>

As you can see, it isn’t exactly user friendly! I decided I wanted to get rid of that additional text (Query Search) and change the Refresh button to Search if I’m doing a query search. I found it wasn’t that hard to do! If you read my last post (<a href="http://www.ephingadmin.com/service-manager-html5-portal-larger-description-text-box/" target="_blank">Larger Description Text Box</a>) you’ll be a little familar with the concept of modifying the view based on the names of Request prompts. I decided to target this based on the name QuerySearch, since I doubt I’m going to be using that anywhere else.

First off, where should I place this in the else if loop in MakeForm.cshtml? I’ll want to put it right before the InstancePicker else if line. That will create a query, and I want mine to be created before the query is created.

Now that I know where I’ll put it, I want to start with the code! First off, I want to only run this if the name is QuerySearch, so I’ll use this line:

 
<pre class="lang:c# decode:true " >else if (item["Prompt"].ToString().Equals("QuerySearch"))</pre> 


Then, I want it to create the query without the label. Here is the generic query code:

 
<pre class="lang:c# decode:true " >&lt;div class="row query_picker_heading"&gt;
	&lt;span class="title" data-required="@item["Optional"].ToString()"&gt;@item["Prompt"].ToString()&lt;/span&gt;
	@if (item["Metadata"].ToString() != "")
	{
		&lt;span class="depend_text"&gt;@Resources.SelfServicePortalResources.QueryPickerMeta @item["Metadata"].ToString()&lt;/span&gt;
	}
&lt;/div&gt;

&lt;button type="button" class="queryPickButton btn" data-idsource="@item["PathSend"].ToString()"&gt;@Resources.SelfServicePortalResources.Refresh&lt;/button&gt;
&lt;div class="error-text"&gt;@ErrorResults.Find(m =&gt; m.MemberNames.ElementAt(0).Equals(item["PathSend"].ToString()))&lt;/div&gt;
&lt;div class="queryResult" id="@item["PathSend"].ToString()" data-required="@item["Optional"].ToString()"&gt;
	&lt;table class="query_table cell-border"&gt;
		&lt;thead&gt;
			&lt;tr&gt;
				&lt;th&gt;&lt;/th&gt;
				@foreach (string column in item["DisplayColumns"] as List&lt;string&gt;)
				{
					&lt;th&gt;@column&lt;/th&gt;
				}

			&lt;/tr&gt;
		&lt;/thead&gt;
		&lt;tbody&gt;&lt;/tbody&gt;
    &lt;/table&gt;
&lt;/div&gt;</pre> 


I notice the first &lt;div is “query_picker_heading”, so I bet I just have to remove that to make it skip creating the header! I remove that and now have this code:

 
<pre class="lang:c# decode:true " >&lt;button type="button" class="queryPickButton btn" data-idsource="@item["PathSend"].ToString()"&gt;@Resources.SelfServicePortalResources.Refresh&lt;/button&gt;
&lt;div class="error-text"&gt;@ErrorResults.Find(m =&gt; m.MemberNames.ElementAt(0).Equals(item["PathSend"].ToString()))&lt;/div&gt;
&lt;div class="queryResult" id="@item["PathSend"].ToString()" data-required="@item["Optional"].ToString()"&gt;
	&lt;table class="query_table cell-border"&gt;
		&lt;thead&gt;
			&lt;tr&gt;
				&lt;th&gt;&lt;/th&gt;
				@foreach (string column in item["DisplayColumns"] as List&lt;string&gt;)
				{
					&lt;th&gt;@column&lt;/th&gt;
				}

			&lt;/tr&gt;
		&lt;/thead&gt;
		&lt;tbody&gt;&lt;/tbody&gt;
    &lt;/table&gt;
&lt;/div&gt;</pre> 


Lastly, I want the button to say Search instead of Refresh. I notice it sets Refresh with the code “@Resources.SelfServicePortalResources.Refresh” which points to the global resources in SelfServicePortal\App_GlobalResources. This means the language of the button wll change based on the language of the portal. If you check in the files, you’ll notice there is an option for “@Resources.SelfServicePortalResources.Search” so I’ll change that to search!

 
<pre class="lang:c# decode:true " >&lt;button type="button" class="queryPickButton btn" data-idsource="@item["PathSend"].ToString()"&gt;@Resources.SelfServicePortalResources.Search&lt;/button&gt;
&lt;div class="error-text"&gt;@ErrorResults.Find(m =&gt; m.MemberNames.ElementAt(0).Equals(item["PathSend"].ToString()))&lt;/div&gt;
&lt;div class="queryResult" id="@item["PathSend"].ToString()" data-required="@item["Optional"].ToString()"&gt;
	&lt;table class="query_table cell-border"&gt;
		&lt;thead&gt;
			&lt;tr&gt;
				&lt;th&gt;&lt;/th&gt;
				@foreach (string column in item["DisplayColumns"] as List&lt;string&gt;)
				{
					&lt;th&gt;@column&lt;/th&gt;
				}

			&lt;/tr&gt;
		&lt;/thead&gt;
		&lt;tbody&gt;&lt;/tbody&gt;
    &lt;/table&gt;
&lt;/div&gt;</pre> 


Now, let’s put this code in our MakeForm.cshtml file, right above the standard query picker and see if it works!

 
<pre class="lang:c# decode:true " >else if (item["Prompt"].ToString().Equals("QuerySearch"))
{
	&lt;button type="button" class="queryPickButton btn" data-idsource="@item["PathSend"].ToString()"&gt;@Resources.SelfServicePortalResources.Search&lt;/button&gt;
	&lt;div class="error-text"&gt;@ErrorResults.Find(m =&gt; m.MemberNames.ElementAt(0).Equals(item["PathSend"].ToString()))&lt;/div&gt;
	&lt;div class="queryResult" id="@item["PathSend"].ToString()" data-required="@item["Optional"].ToString()"&gt;
		&lt;table class="query_table cell-border"&gt;
			&lt;thead&gt;
				&lt;tr&gt;
					&lt;th&gt;&lt;/th&gt;
					@foreach (string column in item["DisplayColumns"] as List&lt;string&gt;)
					{
						&lt;th&gt;@column&lt;/th&gt;
					}

				&lt;/tr&gt;
			&lt;/thead&gt;
			&lt;tbody&gt;&lt;/tbody&gt;
		&lt;/table&gt;
	&lt;/div&gt;
}
else if (item["Type"].ToString().Equals("InstancePicker"))
{
	&lt;div class="row query_picker_heading"&gt;
		&lt;span class="title" data-required="@item["Optional"].ToString()"&gt;@item["Prompt"].ToString()&lt;/span&gt;
		@if (item["Metadata"].ToString() != "")
		{
			&lt;span class="depend_text"&gt;@Resources.SelfServicePortalResources.QueryPickerMeta @item["Metadata"].ToString()&lt;/span&gt;
		}
	&lt;/div&gt;

	&lt;button type="button" class="queryPickButton btn" data-idsource="@item["PathSend"].ToString()"&gt;@Resources.SelfServicePortalResources.Refresh&lt;/button&gt;
	&lt;div class="error-text"&gt;@ErrorResults.Find(m =&gt; m.MemberNames.ElementAt(0).Equals(item["PathSend"].ToString()))&lt;/div&gt;
	&lt;div class="queryResult" id="@item["PathSend"].ToString()" data-required="@item["Optional"].ToString()"&gt;
		&lt;table class="query_table cell-border"&gt;
			&lt;thead&gt;
				&lt;tr&gt;
					&lt;th&gt;&lt;/th&gt;
					@foreach (string column in item["DisplayColumns"] as List&lt;string&gt;)
					{
						&lt;th&gt;@column&lt;/th&gt;
					}

				&lt;/tr&gt;
			&lt;/thead&gt;
			&lt;tbody&gt;&lt;/tbody&gt;
		&lt;/table&gt;
	&lt;/div&gt;
}</pre> 


So, after making these changes I recycle the IIS pool and view the results!

<a href="http://www.ephingadmin.com/wp-content/uploads/2016/02/image-1.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="http://www.ephingadmin.com/wp-content/uploads/2016/02/image_thumb-1.png" alt="image" width="644" height="242" border="0" /></a>