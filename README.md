# Query Examples
Remember to change fields: index, sourcetype, L_bluemixServiceName, etc

### 1. [Percentage of rows - Not as precise time frame](https://answers.splunk.com/answers/611632/calculate-percentage-in-every-row-adding-two-searc.html)
<pre>
index="[insert_index_here]" sourcetype="bluemix:rtr" L_bluemixServiceName="[insert_service_name_here]" L_status!=2* L_reqURLpath=/health 
| rangemap field=L_responseTimeSec "1) <0.5 sec"=0-0.5 "2) 0.5 to 1 sec"=0.5-1 "3) 1 to 3 sec"=1-3 "4) 3 to 5 sec"=3-5 "5) 5 to 10 sec"=5-10 "6) 10 to 30 sec"=10-30 "7) 30 to 60 sec"=30-60 "8) 60 to 120 sec"=60-120 default="9) >120 sec" 
| stats  values(L_status), values(L_bluemixServiceName), values(L_route), values(L_reqURLpath), count as "transactions" by range 
| eventstats sum(transactions) as total
| eval percentage= (transactions/total)*100
</pre>

### 2. Transaction per Status - Using more precise time frame
<pre>
index="[insert_index_here]" L_bluemixServiceName="[insert_service_name_here]" sourcetype="bluemix:RTR" L_status=4* 
| rangemap field=L_responseTimeSec "1) <0.5 sec"=0-0.5 "2) 0.5 to 1 sec"=0.5-1 "3) 1 to 3 sec"=1-3 "4) 3 to 5 sec"=3-5 "5) 5 to 10 sec"=5-10 "6) 10 to 20 sec"=10-20 "7) 20 to 30 sec"=20-30 "8) 30 to 60 sec"=30-60 "9) 60 to 120 sec"=60-120 default="10) >120 sec" 
| stats values(L_bluemixServiceName), values(L_route), count as "Number of Transactions" by range, L_status, L_reqURLpath

![text](https://github.com/jgong1993/splunk_Q/blob/master/GH_Pics/Transaction%20per%20Status.PNG)
</pre>

### 3. Distribution of Instances
<pre>
index=bluemixapps_* sourcetype=bluemix:RTR 
| eval convertedTime = strptime(date_month + " " + date_mday + " " + date_year, "%B %d %Y")
| eval time = strftime(convertedTime, "%B %d %Y")
| table time, L_appIndex, L_status
| stats count by L_appIndex L_status time 
| stats list(count) as Num_Of_Statuses by time, L_status, L_appIndex 
| eventstats count(L_appIndex) as NumOfInstances, sum(Num_Of_Statuses) as total by time, L_status
| eval distribution_per_instance = round((100/NumOfInstances),2)
| eval perc = round((Num_Of_Statuses/total)*100,4)
| eventstats perc75(L_appIndex) perc25(L_appIndex) by time, L_status
| fields - total, NumOfInstances
</pre>

### 4. Group By
<pre>
index="[insert_index_here]" sourcetype="bluemix:rtr" L_reqURLpath!="/health" 
| rangemap field=L_responseTimeSec "1) <0.5 sec"=0-0.5 "2) 0.5 to 1 sec"=0.5-1 "3) 1 to 3 sec"=1-3 "4) 3 to 5 sec"=3-5 "5) 5 to 10 sec"=5-10 "6) 10 to 60 sec"=10-60 "7) 60 to 120 sec"=60-120 default="8) >120 sec" 
| chart count by L_status,L_bluemixServiceName
</pre>

### 5. Aggregation
<pre>
index="[insert_index_here]" sourcetype="bluemix:RTR" L_bluemixServiceName="[insert_service_name_here]" L_status!=2*
| eval convertedTime = strptime(date_month + " " + date_mday + " " + date_year, "%B %d %Y")
| eval time = strftime(convertedTime, "%B %d %Y")
| stats avg(L_responseTimeSec), median(L_responseTimeSec), min(L_responseTimeSec), max(L_responseTimeSec)   by L_bluemixServiceName , time
</pre>

### 6. Get aggregation for count
<pre>
index="[insert_index_here]" sourcetype=bluemix:rtr L_reqURLpath=/health  L_bluemixServiceName="[insert_service_name_here]"
| eval convertedTime = strptime(date_month + " " + date_mday + " " + date_year, "%B %d %Y")
| eval time = strftime(convertedTime, "%B %d %Y")
| stats count(L_bluemixServiceName) as countService by L_bluemixServiceName, time, L_status, L_appIndex
| eventstats mean(countService) as meanService by time, L_status
| eval meanService = round(meanService, 2)
| eventstats stdev(countService) as stdService by time, L_status
| eval stdService = round(stdService, 2)
| eventstats median(countService) as medianService by time, L_status
| eval medianService = round(medianService, 2)
| eventstats max(countService) as maxService by time, L_status
| eventstats min(countService) as minService by time, L_status
| fields L_bluemixServiceName, time, L_status, L_appIndex, countService, meanService, stdService, medianService, maxService, minService
</pre>

### 7. Volume of Traffic
<pre>
index="[insert_index_here]" sourcetype="bluemix:RTR" L_bluemixServiceName="[insert_service_name_here]"
| eval convertedTime = strptime(date_month + " " + date_mday + " " + date_year, "%B %d %Y")
| eval time = strftime(convertedTime, "%B %d %Y")
| stats count(L_bluemixServiceName) as volume  avg(volume) , median(volume), stdev(volume), min(volume), max(volume)   by L_bluemixServiceName
</pre>


### 8. Count of L_status vs Time
<pre>
index="[insert_index_here]" sourcetype="bluemix:RTR"  L_status=4* OR L_status=5*  L_routeEnvKP=preprod L_reqURLpath!="/health"  L_bluemixServiceName!="[insert_service_name_here]" 
| timechart  span=1h limit=20 count by  L_status
</pre>

# Functions
### [Associate](https://docs.splunk.com/Documentation/Splunk/7.2.4/SearchReference/Associate)  
	• "Identify correlations between fields using entropy based on field's values"

### [Correlate](https://docs.splunk.com/Documentation/Splunk/7.2.4/SearchReference/Correlate)
	• "Does not identify contents of the fields"
	• "Represents the percentage of times that the two fields exist in the same events"

# Random Notes 
<pre>
host=myhost myfield=A OR myfield=B myotherfield=C
is equivalent to
host=myhost AND ( myfield=A OR myfield=B ) AND myotherfield=C
</pre>


# Format
<pre>

</pre>