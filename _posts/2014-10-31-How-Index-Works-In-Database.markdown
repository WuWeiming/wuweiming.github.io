---
layout: post
title: How Index Works In DataBase
category: Coding
tags: github 
year: 2014
month: 10
day: 31
published: true
summary: An introduction to how index works in database
image: post_one.jpg
---

<div class="row">	
	<div class="span9 columns">
		<h2>Why is it needed?<h2>
		<p>When data is stored on disk based storage devices, it is stored as blocks of data. These blocks are accessed in their entirety, making them the atomic disk access operation. Disk blocks are structured in much the same way as linked lists; both contain a section for data, a pointer to the location of the next node (or block), and both need not be stored contiguously.<p>

		<p>Due to the fact that a number of records can only be sorted on one field, we can state that searching on a field that isn’t sorted requires a Linear Search which requires N/2 block accesses (on average), where N is the number of blocks that the table spans. If that field is a non-key field (i.e. doesn’t contain unique entries) then the entire table space must be searched at N block accesses.<p>

		<p>Whereas with a sorted field, a Binary Search may be used, this has log2 N block accesses. Also since the data is sorted given a non-key field, the rest of the table doesn’t need to be searched for duplicate values, once a higher value is found. Thus the performance increase is substantial.<p>

		<h2>What is indexing?<h2>

		<p>Indexing is a way of sorting a number of records on multiple fields. Creating an index on a field in a table creates another data structure which holds the field value, and pointer to the record it relates to. This index structure is then sorted, allowing Binary Searches to be performed on it.<p>

		<p>The downside to indexing is that these indexes require additional space on the disk, since the indexes are stored together in a table using the MyISAM engine, this file can quickly reach the size limits of the underlying file system if many fields within the same table are indexed.<p>

		<h2>How does it work?<h2>

		<p>Firstly, let’s outline a sample database table schema;</p>

		<table>
			<tr>
				<th>Field name</th>
				<th>Data type</th>     
				<th>Size on disk</th>
			</tr>
			<tr>
				<td>id (Primary key)</td> 
				<td>Unsigned INT</td>   
				<td>4 bytes</td>
			</tr>
			<tr>
				<td>firstName</td>
				<td>Char(50)</td>
				<td>50 bytes</td>
			</tr>
			<tr>
				<td>lastName</td>
				<td>Char(50)</td>
				<td>50 bytes</td>
			</tr>
			<tr>
				<td>emailAddress</td>
				<td>Char(100)</td>
				<td>1000 bytes</td>
			</tr>
	    </table>

		<p>Note: char was used in place of varchar to allow for an accurate size on disk value. This sample database contains five million rows, and is unindexed. The performance of several queries will now be analyzed. These are a query using the id (a sorted key field) and one using the firstName (a non-key unsorted field).</p>

		<h3>Example 1<h3>

		<p>Given our sample database of r = 5,000,000 records of a fixed size giving a record length of R = 204 bytes and they are stored in a table using the MyISAM engine which is using the default block size B = 1,024 bytes. The blocking factor of the table would be bfr = (B/R) = 1024/204 = 5 records per disk block. The total number of blocks required to hold the table is N = (r/bfr) = 5000000/5 = 1,000,000 blocks.</p>

		<p>A linear search on the id field would require an average of N/2 = 500,000 block accesses to find a value given that the id field is a key field. But since the id field is also sorted a binary search can be conducted requiring an average of log2 1000000 = 19.93 = 20 block accesses. Instantly we can see this is a drastic improvement.</p>

		<p>Now the firstName field is neither sorted, so a binary search is impossible, nor are the values unique, and thus the table will require searching to the end for an exact N = 1,000,000 block accesses. It is this situation that indexing aims to correct.</p>

		<p>Given that an index record contains only the indexed field and a pointer to the original record, it stands to reason that it will be smaller than the multi-field record that it points to. So the index itself requires fewer disk blocks that the original table, which therefore requires fewer block accesses to iterate through. The schema for an index on the firstName field is outlined below;</p>

		<table>
			<tr>
				<th>Field name</th>
				<th>Data type</th>     
				<th>Size on disk</th>
			</tr>
			<tr>
				<td>firstName</td>
				<td>Char(50)</td>
				<td>50 bytes</td>
			</tr>
			<tr>
				<td>(record pointer)</td>
				<td>Special</td>
				<td>4 bytes</td>
			</tr>
	    </table>
		<p>Note: Pointers in MySQL are 2, 3, 4 or 5 bytes in length depending on the size of the table.</p>

		<h3>Example 2</h3>

		<p>Given our sample database of r = 5,000,000 records with an index record length of R = 54 bytes and using the default block size B = 1,024 bytes. The blocking factor of the index would be bfr = (B/R) = 1024/54 = 18 records per disk block. The total number of blocks required to hold the table is N = (r/bfr) = 5000000/18 = 277,778 blocks.</p>

		<p>Now a search using the firstName field can utilise the index to increase performance. This allows for a binary search of the index with an average of log2 277778 = 18.08 = 19 block accesses. To find the address of the actual record, which requires a further block access to read, bringing the total to 19 + 1 = 20 block accesses, a far cry from the 277,778 block accesses required by the non-indexed table.</p>

		<h2>When should it be used?<h2>

		<p>Given that creating an index requires additional disk space (277,778 blocks extra from the above example), and that too many indexes can cause issues arising from the file systems size limits, careful thought must be used to select the correct fields to index.</p>

		<p>Since indexes are only used to speed up the searching for a matching field within the records, it stands to reason that indexing fields used only for output would be simply a waste of disk space and processing time when doing an insert or delete operation, and thus should be avoided. Also given the nature of a binary search, the cardinality or uniqueness of the data is important. Indexing on a field with a cardinality of 2 would split the data in half, whereas a cardinality of 1,000 would return approximately 1,000 records. With such a low cardinality the effectiveness is reduced to a linear sort, and the query optimizer will avoid using the index if the cardinality is less than 30% of the record number, effectively making the index a waste of space.</p>
	</div>
</div> 

<div class="row">	
	<div class="span9 column">
			<p class="pull-right">{% if page.previous.url %} <a href="{{page.previous.url}}" title="Previous Post: {{page.previous.title}}"><i class="icon-chevron-left"></i></a> 	{% endif %}   {% if page.next.url %} 	<a href="{{page.next.url}}" title="Next Post: {{page.next.title}}"><i class="icon-chevron-right"></i></a> 	{% endif %} </p>  
	</div>
</div>

<div class="row">	
    <div class="span9 columns">    
		<h2>Comments Section</h2>
	    <p>Feel free to comment on the post but keep it clean and on topic.</p>	
		<div id="disqus_thread"></div>
		<script type="text/javascript">
			/* * * CONFIGURATION VARIABLES: EDIT BEFORE PASTING INTO YOUR WEBPAGE * * */
			var disqus_shortname = 'ericjones'; // required: replace example with your forum shortname
			var disqus_identifier = '/blog/How-Index-Works-In-DataBase';
			var disqus_url = '/blog/How-Index-Works-In-DataBase';
			
			/* * * DON'T EDIT BELOW THIS LINE * * */
			(function() {
				var dsq = document.createElement('script'); dsq.type = 'text/javascript'; dsq.async = true;
				dsq.src = 'http://' + disqus_shortname + '.disqus.com/embed.js';
				(document.getElementsByTagName('head')[0] || document.getElementsByTagName('body')[0]).appendChild(dsq);
			})();
		</script>
		<noscript>Please enable JavaScript to view the <a href="http://disqus.com/?ref_noscript">comments powered by Disqus.</a></noscript>
		<a href="http://disqus.com" class="dsq-brlink">blog comments powered by <span class="logo-disqus">Disqus</span></a>
	</div>
</div>

<!-- Twitter -->
<script>!function(d,s,id){var js,fjs=d.getElementsByTagName(s)[0];if(!d.getElementById(id)){js=d.createElement(s);js.id=id;js.src="//platform.twitter.com/widgets.js";fjs.parentNode.insertBefore(js,fjs);}}(document,"script","twitter-wjs");</script>

<!-- Google + -->
<script type="text/javascript">
  (function() {
    var po = document.createElement('script'); po.type = 'text/javascript'; po.async = true;
    po.src = 'https://apis.google.com/js/plusone.js';
    var s = document.getElementsByTagName('script')[0]; s.parentNode.insertBefore(po, s);
  })();
</script>