---
layout: post
published: true
title: OWA Toolkit Based File Upload
date: '2018-10-05'
---
<script >
window.addEventListener('load', function() {
    
  var config = {
    startOnLoad: true,
    theme:"forest",
    flowchart:{
        useMaxWidth:false,
        htmlLabels:true,
        curve: "basis" 

    }
};
    mermaid.initialize(config);
});


</script> 
## OWA Toolkit File upload

The logic in 18.3 changed to better accomidate OWA based applications that are not apex based.  It's probably easiest to explan with a flow chart so here the current logic as of ORDS 18.3+.
 
{% mermaid %}
  graph TD
    A[ Read Defaults.xml ]-->B{ Contains owa.docTable };
    B--Yes-->D{ table name is <br/> FLOWS_FILES.WWV_FLOW_FILE_OBJECTS$ }
    B--No-->C;
    C{ contains apex.docTable }--Yes-->D;
    C--No-->E{ Check Apex 5+ };
    D--Yes-->E;
    E--Yes-->F[ Use apex_util.set_blob ]
    D--No-->G[ Use specified Table];
    E--No-->G;
{% endmermaid %}

## How this was rendered

I found  Mermaid.JS found here: [https://mermaidjs.github.io]() this is pretty terse syntax to get a simple flow chart out.  Here's the syntax for the above flowchart.

```
  graph TD
    A[ Read Defaults.xml ]-->B{ Contains owa.docTable };
    B--Yes-->D{ table name is <br/> FLOWS_FILES.WWV_FLOW_FILE_OBJECTS$ }
    B--No-->C;
    C{ contains apex.docTable }--Yes-->D;
    C--No-->E{ Check Apex 5+ };
    D--Yes-->E;
    E--Yes-->F[ Use apex_util.set_blob ]
    D--No-->G[ Use specified Table];
    E--No-->G;
```
