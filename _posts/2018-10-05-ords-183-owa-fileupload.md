---
layout: post
published: true
title: OWA Toolkit Based File Upload
date: '2018-10-05'
---

<script src="https://unpkg.com/mermaid@8.0.0-rc.1/dist/mermaid.js"></script>

<script>
    
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


</script> 
## OWA Toolkit File upload

The logic in 18.3 changed to better accomidate OWA based applications that are not apex based.  It's probably easiest to explan with a flow chart so here the current logic as of ORDS 18.3+.
 
  <div class="mermaid">
  graph TD
    A[ Read Defaults.xml ]-->B{ Contains owa.docTable };
    B--Yes-->D{ table name is <br/> FLOWS_FILES.WWV_FLOW_FILE_OBJECTS$ }
    B--No-->C;
    C{ contains apex.docTable }--Yes-->D;
    C--No-->E{ Check Apex 5+ };
    D--Yes-->E;
    D--No-->G;
    E--Yes-->F[ Use apex_util.set_blob ]
    E--No and DO NOT have tablename-->H[ ERROR ];
    E--No and have tablename-->G[ Use specified table name ];
  </div>

## Great but how was this flow chart made.

I found  Mermaid.JS found here: [https://mermaidjs.github.io]() this is pretty terse syntax to get a simple flow chart out.  Here's the syntax for the above flowchart.

```
  graph TD
    A[ Read Defaults.xml ]-->B{ Contains owa.docTable };
    B--Yes-->D{ table name is <br/> FLOWS_FILES.WWV_FLOW_FILE_OBJECTS$ }
    B--No-->C;
    C{ contains apex.docTable }--Yes-->D;
    C--No-->E{ Check Apex 5+ };
    D--Yes-->E;
    D--No-->G;
    E--Yes-->F[ Use apex_util.set_blob ]
    E--No and DO NOT have tablename-->H[ ERROR ];
    E--No and have tablename-->G[ Use specified table name ];
```

And here's the configuration being used:

```
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

```    