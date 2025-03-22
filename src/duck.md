---
title : duckdb
---

```js
import * as aq from "npm:arquero";
import {DuckDBClient} from "npm:@observablehq/duckdb"
import * as duckdb from "npm:@duckdb/duckdb-wasm"
import * as d3 from "npm:d3";
import { mean, deviation } from "d3-array";

//import * as arrow from 'apache-arrow';

await console.log("Import complete")
```
<!-- ```js
const db = DuckDBClient.of({gene_exp: FileAttachment("data/gene_expression_matrix_sub.csv")});
``` -->

<!-- ```js
// const gaia = FileAttachment("docs/.observablehq/cache/gene_expression_coldata.parquet").parquet()
const gene_exp = FileAttachment("gene_expression_matrix_sub.csv").csv()
const db = DuckDBClient.of({gene_exp});
console.log("gene_exp", gene_exp)
console.log("db", db)
``` -->

```js
const gene_exp = FileAttachment("Demo_Data.csv").csv()
const db = DuckDBClient.of({gene_exp});
console.log("gene_exp", gene_exp)
console.log("db", db)
```

```js
const bins = await db.query("SELECT * FROM gene_exp ")
console.log("bins: ",bins)

```

```js
const fieldName = 'SUB205386.1529_0_Days_BS1028555';
const dataArray = bins.batches[0].data.children
console.log("dataArray: ",dataArray)
```

<!-- ```js
const myTransposedArray = d3.transposeArray(bins)
console.log("myTransposedArray: ",myTransposedArray)

``` -->
<!-- ```js
const bins = await db.sql`SELECT *
FROM gene_exp
LIMIT 10`
```

```js
console.log("bins: ",bins)
``` -->




```js
Plot.rectY(bins, {x: "gene_name", y: "A1CF"}).plot()
```

### Data With z-score

```js
const data = bins.schema.fields
console.log("data: ", data)
```

```js
//values = bins.map(d => +d.'SUB205386.1529_0_Days_BS1028555');  
// Ensure the values are numbers
//console.log("d: ", bins.'SUB205386.1529_0_Days_BS1028555')
//const mean1 = d3.mean(data, d => d.'SUB205386.1529_0_Days_BS1028555')
const mean1 = d3.mean(bins, d => d['SUB205386.1529_0_Days_BS1028555']);
console.log("mean:", mean1)

const stdDev = d3.deviation(bins, d => d['SUB205386.1529_0_Days_BS1028555'])
console.log("stdDev:",stdDev)
```


```js
const zScores = data.map(d => ({
  ...d,
  zscore: (d['SUB205386.1529_0_Days_BS1028555'] - mean1) / stdDev
}))
console.log("zScores: ",zScores)
```
