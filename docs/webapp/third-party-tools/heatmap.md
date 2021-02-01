# Heatmap

InterMine makes use of canvasXpress [heatmap](http://www.canvasxpress.org/api/heatmap_graphs.html) to visualize gene expression data.

[CanvasXpress](http://www.canvasxpress.org/) is a JavaScript library based on the `<canvas>` tag implemented in HTML5. It is written by Isaac Neuhausi.

Hierarchical and k-Means clustering algorithms and zoom in/out functionality have been implemented within the heatmap.

## An example in modMine

A specific heatmap application can be referred in [modMine](http://intermine.modencode.org/). It visualizes fly expression data \([example](http://intermine.modencode.org/query/bagDetails.do?scope=global&bagName=example)\) generated from [modENCODE project](http://www.modencode.org/).

The raw data is parsed and converted to InterMine objects. In a Struts controller, the expression data will be fetched by running a InterMine path query and parsed to JSON string. The JSON string will be sent to a JSP page by a http request to feed into heatmap.

### Expression data source

[FlyExpressionScoreConverter](https://github.com/intermine/intermine/blob/master/bio/sources/modmine/fly-expression-score/main/src/org/intermine/bio/dataconversion/FlyExpressionScoreConverter.java) is a specific data converter for modENCODE fly expression data. The class is located at `bio/sources/modmine/fly-expression-score`. Any other similar expression data conversion tasks can take the data source as a reference.

Exprssion data type is an extension of InterMine core model. It is addressed in `modmine/dbmodel/resources/modencode-metadata_additions.xml`

```markup
# modmine/dbmodel/resources/modencode-metadata_additions.xml
<class name="GeneExpressionScore" is-interface="true">
<attribute name="score" type="java.lang.Double" />
    <reference name="gene" referenced-type="Gene" reverse-reference="expressionScores" />
    <reference name="cellLine" referenced-type="CellLine" />
    <reference name="developmentalStage" referenced-type="DevelopmentalStage" />
    <reference name="submission" referenced-type="Submission" />
    <reference name="organism" referenced-type="Organism" />
</class>
```

A better practice would be to add the model extension to a source specific additions.xml under a source directory.

### Controller

The controller class [HeatMapController](https://github.com/modENCODE-DCC/modmine/blob/master/modmine/webapp/src/org/modmine/web/HeatMapController.java) is a component of [Struts MVC framework](https://struts.apache.org/). It holds the logic to process user requests, and selects a proper webpage for the user.

In HeatMapController, a query is run to fetch expression scores from database \(ref method `queryExpressionScore`\), then the results are parsed to JSON string \(ref method `getJSONString`\) and set in the request \(ref method `findExpression`\).

Struts config:

```markup
# modmine/webapp/resources/struts-config-model.xml 
<action path="/initHeatMap"
type="org.modmine.web.HeatMapController" />

<action path="/heatMap" forward="heatMap.page" />


#  modmine/webapp/resources/tiles-defs-model.xml
<definition name="heatMap.tile" path="/model/heatMap.jsp"
    controllerUrl="/initHeatMap.do"/>

<definition name="heatMap.page" extends="layout.template">
    <put name="body" value="heatMap.tile"/>
    <put name="pageName" value="heatMap"/>
</definition>
```

### Web page

[heatMap.jsp](https://github.com/modENCODE-DCC/modmine/blob/master/modmine/webapp/resources/webapp/model/heatMap.jsp) displays heatmap. canvasXpress object takes expression JSON string and other parameters in to create a heatmap \(in modMine, we have two separate heatmaps for cell line and developmental stage respectively\). jQuery was used to adjust page layout.

## Further development

A modern way of creating widget like heatmap would be using InterMine webservices framework to query and generate JSON strings and embed heatmap on any web page. To [learn more...](http://github.com/intermine/intermine-embedding-examples)

An alternative library would be [D3.js](http://d3js.org/), an example of heatmap can be found [here](http://www.larsko.org/v/mpte/). In [ThaleMine](https://apps.araport.org/thalemine/begin.do) there is a D3 implementation \(see any gene list report page, [for example](https://apps.araport.org/thalemine/bagDetails.do?scope=all&bagName=Demo+1+-+Sucrose+Transporters+List), [code](https://github.com/intermine/CDN/blob/master/js/intermine/expression/1.0.3/expression.js). However canvasXpress is particularly designed to display genomics data, D3 is for a broader use.