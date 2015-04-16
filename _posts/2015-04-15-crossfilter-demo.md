---
title: "Crossfilter Demo: Starring Crossfilter, Featuring Dimensional Charting, with Cameo Appearances by GeoJson and rawgit"
author: "Robert Piccone"
output: html_document
layout: post
description: Crossfilter / Dimensional Charting Demo 
tags: assignments
---

 <a href="../../../../assets/rap125/blscrossfiltermap.html">The Charts</a>
 
 
 <a href="http://www.bls.gov">BLS</a>:
 
The HTML/Javascript Code:

<pre><code>
<meta http-equiv="content-type" content="text/html; charset=UTF8"> 

<script type="text/javascript" src="https://cdn.rawgit.com/square/crossfilter/master/crossfilter.js"></script>
<script type="text/javascript" src="https://cdn.rawgit.com/mbostock/d3/master/d3.js"></script>
<script type="text/javascript" src="https://cdn.rawgit.com/dc-js/dc.js/master/dc.js"></script>

<link rel="stylesheet" type="text/css" href="https://cdn.rawgit.com/dc-js/dc.js/master/dc.css" media="screen" /> 


<div align=center id="chart-bar-ymvals"> <h1>Annual National Compensation Survey</h1> </div>

<div align=center id="us-chart"><h2>Totals By State</h2> </div>

<div float=left align=center id="chart-ring-state"><h2>Proportion By State</h2></div>
<script>
/*

National Compensation Survey
<div id="dc-table-graph"></div>
http://www.bls.gov/help/hlpforma.htm#EC
	Series ID   NCU5306633300003
	Positions       Value           Field Name
	1-2             NC              Prefix
	3               U               Seasonal Adjustment Code 
	4-5             53              State Code
	6-9             0663            Area Code
	10-14           33000           Occupation Code
	15-16           03              Level Code
*/

//http://eric.clst.org/wupl/Stuff/gz_2010_us_040_00_500k.json
d3.json("./gz_2010_us_040_00_500k.json", function(error, statesJson){
d3.tsv("./nc.state", function(stateData) {
d3.tsv("./nc.data.1.AllData", function(data) {
function print_filter(filter){
	var f=eval(filter);
	if (typeof(f.length) != "undefined") {}else{}
	if (typeof(f.top) != "undefined") {f=f.top(Infinity);}else{}
	if (typeof(f.dimension) != "undefined") {f=f.dimension(function(d) { return "";}).top(Infinity);}else{}
	console.log(filter+"("+f.length+") = "+JSON.stringify(f).replace("[","[\n\t").replace(/}\,/g,"},\n\t").replace("]","\n]"));
} 

  // Run the data through crossfilter 
	var states = crossfilter(stateData);
	var stCodeDim = states.dimension(function(d) {return d.state_code;});
	//print_filter("stateData");  
	var facts = crossfilter(data);
	var parseDate = d3.time.format("%Y").parse;

	data.forEach(function(d) {
		//d.prefix= d.series_id.substr(0,2);
		//d.yearmo = parseDate(d.year + "-" + d.period.substr(1,2));
		d.seasonal_ac= d.series_id.substr(2,1);
		d.state_cd= +d.series_id.substr(3,2);
		stCodeDim.filter(d.series_id.substr(3,2));
		d.stateName=stCodeDim.top(1)[0].state_name;
		stCodeDim.filterAll();
		d.area_cd= d.series_id.substr(5,4);		
		d.occupational_cd= d.series_id.substr(9,5);		
		d.level_cd= d.series_id.substr(14,2);		
		d.year= parseDate(d.year);
		//d.period= +d.period.substr(1,2);
		d.value= +d.value.trim().substr(1);
	});
//print_filter("data"); 

 // Create the dc.js chart objects & link to div
  //var dataTable = dc.dataTable("#dc-table-graph");

	var stateDim  = facts.dimension(function(d) {return d.state_cd;});
	stateDim.filter(0); 
	facts.remove();
	stateDim.filterAll();
	stateDim.filter([56,80]);
	facts.remove();
	stateDim.filterAll();
	
	var sacDim = facts.dimension(function(d) {return d.seasonal_ac;});
	sacDim.filter("S");
	facts.remove();
	sacDim.filterAll();

	var stNameDim  = facts.dimension(function(d) {return d.stateName;});
	var stName_total = stNameDim.group().reduceSum(function(d) {return d.value;});
	var state_total = stateDim.group().reduceSum(function(d) {return d.value;});
	var stNameDim2  = facts.dimension(function(d) {return d.stateName;});
	var stName_total2 = stNameDim2.group().reduceSum(function(d) {return d.value;});
			  
 // Create  dimension
  	var dateDim = facts.dimension(function (d) {return d.year;});
	var ymtotal = dateDim.group().reduceSum(function(d) {return d.value;}); 
	var minDate = dateDim.bottom(1)[0].year;
	var maxDate = dateDim.top(1)[0].year;

	
 // Setup the charts
var ymValsChart  = dc.barChart("#chart-bar-ymvals"); 

ymValsChart
	.width(800).height(300)
	.dimension(dateDim)
	.group(ymtotal)
	.x(d3.time.scale().domain([minDate,maxDate])) 
	.elasticX(true)
	.elasticY(true)
	.centerBar(true)
	.margins({top: 10, right: 50, bottom: 30, left: 40})
	.renderTitle(false)
	.yAxisLabel("", 100)
	.xUnits(function(){return 10;});
	
	//.yAxisLabel("Total")  
	;
	
var stateRingChart   = dc.pieChart("#chart-ring-state");
stateRingChart
    .width(250).height(250)
    .dimension(stNameDim2)
    .group(stName_total2)
    .innerRadius(30); 

var mapChart = dc.geoChoroplethChart("#us-chart");
mapChart
	.width(1000).height(500)
    .dimension(stNameDim)
	.group(stName_total)
	.colors(d3.scale.linear().domain([0,330000]).range(["pink","red"]))
	.overlayGeoJson(statesJson.features, "state", function(d) {
    return d.properties.NAME;});


	
dc.renderAll(); 
});
});
});
</script>
</code></pre>