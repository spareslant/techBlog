---
title: "Concourse Ci"
date: 2020-04-27T21:15:25+01:00
draft: true
---
<!--- Below style are also defined in static/css/my.css file.
They are repeatedly defined here so that pandoc can generate
the final HTML with all necessary css styles.
Note: draft: true above. This prevents publishing it to GitHUB.
--->
<style>
/* To highlight text in Green in pre tag */
.hl {color: #008A00;}
/* To highlight text in Bold Green in pre tag */
.hlb {color: #008A00; font-weight: bold;}
/* To highlight text in Bold Red in pre tag */
.hlbr {color:#e90001; font-weight: bold;}
/* <code> tag does not work in blogger. Use following class with span tag */
.code {
    color:#7e168d; 
    background: #f0f0f0; 
    padding: 0.1em 0.4em;
    font-family: SFMono-Regular, Consolas, "Liberation Mono", Menlo, Courier, monospace;
}
</style>

### Concepts:

* a resource is fetched in a job via `get`
* `inputs` are specified in `tasks`.
* `outputs` are specified in `tasks` not in jobs.
* resource containers remain available even after the finish of job. Resource can be checked with following command.
   * fly -t tutorial intercept --check first-pipeline/my-local-repo
