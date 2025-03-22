---
theme: dashboard
title: New Gene Selection Template
toc: false
---

## Gene Selection template
```js
import * as aq from "npm:arquero";
import {DuckDBClient} from "npm:@observablehq/duckdb"
import * as duckdb from "npm:@duckdb/duckdb-wasm"
import * as d3 from "npm:d3";
import * as Inputs from "npm:@observablehq/inputs";
```
```js
const gene_expression_matrix = FileAttachment("gene_expression_cohort_info_sub.csv").csv()
```


```js
const db = DuckDBClient.of({gene_expression_matrix});
console.log("db", db)
```


```js
console.log("gene_expression_matrix:", gene_expression_matrix)
gene_expression_matrix
```

```js
const geneArray = [];

const header = Object.keys(gene_expression_matrix[0]);

// Remove non-gene columns: 'Sample_Id', 'race', 'ethnicity', 'study_accession', 'vaccine', 'gender'
const nonGeneColumns = ['sample_id', 'race', 'ethnicity', 'study_accession', 'vaccine', 'gender', 'age_imputed'];
const geneNames = header.filter(column => !nonGeneColumns.includes(column));

// Append gene names to geneArray if they are not already present
geneNames.forEach(gene => {
  if (!geneArray.includes(gene)) {
    geneArray.push(gene);
  }
});

console.log("Updated geneArray:", geneArray);
```

```js
const uniqueVaccines = Array.from(new Set(gene_expression_matrix.map(row => row.vaccine)));
console.log("Unique vaccine values:", uniqueVaccines);
```

```js
const searchGene = view(Inputs.search(geneArray, {placeholder: "Search gene_name"}))
//console.log("search: ", search)
```

```js
const searchArrayGene = searchGene.map(value => value ? [value] : []);
console.log("searchArrayGene: ", searchArrayGene)
```

```js
const gene_rows = view(Inputs.table(searchArrayGene, {width: 200, maxHeight: 200, required: false}))
console.log("gene_rows: ", gene_rows)
```

```js
const final_gene_rows = gene_rows.flat()
console.log("final_gene_rows: ", final_gene_rows)
```

<!-- ```js
final_gene_rows
``` -->

```js
const gender = view(Inputs.checkbox(["Male", "Female"], {label: "Gender"}));
console.log("gender: ", gender)
```

<!-- ```js
gender
``` -->

```js
const min_age = view(Inputs.range([0, 100], {label: "Minimum Age:", step: 1, value: 0}));
```

```js
const max_age = view(Inputs.range([min_age, 100], {label: "Maximum Age:", step: 1, value: min_age}));
```

```js
const searchVaccine = view(Inputs.search(uniqueVaccines, {placeholder: "Search vaccine"}))
```

```js
const searchVaccineGene = searchVaccine.map(value => value ? [value] : []);
console.log("searchVaccineGene: ", searchVaccineGene)
```

```js
const vaccine_rows = view(Inputs.table(searchVaccineGene, {width: 200, maxHeight: 200, required: false}))
console.log("vaccine_rows: ", vaccine_rows)
```

```js
const final_vaccine_rows = vaccine_rows.flat()
console.log("final_vaccine_rows: ", final_vaccine_rows)
```

<!-- ```js
final_vaccine_rows
``` -->

<!-- ```js
const to_plot = view(Inputs.radio(["value", "z_score"], {label: "Choose data to view", value: "value"}))
``` -->

```js
const zscore_toggle = view(Inputs.toggle({
    label: "Use Z-Score", value: false
}))
```

<!-- ```js
to_plot
``` -->


```js
if (final_gene_rows.length > 0) {
    const columns = final_gene_rows.join(', ');

    // Convert selected gender and vaccine arrays to SQL-compatible format
    const selectedGenders = gender.length > 0 ? `gender IN ('${gender.join("', '")}')` : '';
    const selectedVaccines = final_vaccine_rows.length > 0 ? `vaccine IN ('${final_vaccine_rows.join("', '")}')` : '';

    // Construct the WHERE clause based on selected filters
    const whereConditions = [];
    if (selectedGenders) whereConditions.push(selectedGenders);
    if (selectedVaccines) whereConditions.push(selectedVaccines);
    if (min_age > 0) whereConditions.push(`age_imputed >= ${min_age}`);
    if (max_age > 0) whereConditions.push(`age_imputed <= ${max_age}`);
    const whereClause = whereConditions.length > 0 ? `WHERE ${whereConditions.join(' AND ')}` : '';

    // Construct the SQL query
    const query = `SELECT Sample_Id, ${columns}, race, ethnicity, study_accession, vaccine, gender, age_imputed FROM gene_expression_matrix ${whereClause}`;
    console.log("Generated SQL query:", query);

    // Execute the query and log results
    const bins = await db.query(query);
    console.log("bins: ", bins);
    //view(bins);
    //const arrowTable = bins.batches[0].toArray();
    const arrowTable = bins.batches[0].toArray().map(row => Object.assign({}, row));
    //view(arrowTable)

    arrowTable.forEach(row => {
        final_gene_rows.forEach(gene => {
            // Extract values for the current gene column
            const values = arrowTable.map(d => parseFloat(d[gene]));
            
            // Calculate mean and standard deviation using D3
            const mean = d3.mean(values);
            const stdDev = d3.deviation(values);

            // Calculate the z-score for the current value
            const zScore = (parseFloat(row[gene]) - mean) / stdDev;

            // Add the z-score to the data object
            row[`${gene}_zScore`] = zScore;
        });
    });
    //view(arrowTable)



    // Prepare data for Plot
    const plotData = final_gene_rows.flatMap(gene => {
        return arrowTable.map(row => ({
            gene: gene,
            value: parseFloat(row[gene]),
            zScore: row[`${gene}_zScore`],
            gender: row.gender
        }));
    });
    view(plotData)

    // Plotting
    
    if (!zscore_toggle){
        view(Plot.plot({
            y: {
                grid: true,
                inset: 2
            },
            facet: {
                data: plotData,
                x: "gender",  // Facet by gender
            },
            marks: [
                Plot.frame({ strokeOpacity: 0.1 }),
                Plot.boxY(plotData, { x: "gene", y: "value", fill: "gender", tip: true })
            ]
        }));
    }
    else{
        view(Plot.plot({
            y: {
                grid: true,
                inset: 2
            },
            facet: {
                data: plotData,
                x: "gender",  // Facet by gender
            },
            marks: [
                Plot.frame({ strokeOpacity: 0.1 }),
                Plot.boxY(plotData, { x: "gene", y: "zScore", fill: "gender", tip: true })
            ]
        }));
    }

    //Heatmap code

    const inset_slider = view(Inputs.range(
        [0, 10], 
        {value: 0.1, step: 0.1, label: "Inset amount:"}
    ))

    const padding_slider = view(Inputs.range(
        [0, 1],
        {value: 0, step: 0.01, label: "Padding:"}
    ))

    const marginleft_slider = view(Inputs.range(
        [0, 200],
        {value: 120, step: 1, label: "Margin Left:"}
    ))

    const marginbottom_slider = view(Inputs.range(
        [0, 200],
        {value: 120, step: 1, label: "Margin Bottom:"}
    ))

    const marginright_slider = view(Inputs.range(
        [0, 200],
        {value: 120, step: 1, label: "Margin Right:"}
    ))

    const ticksize_y_slider = view(Inputs.range(
        [0, 20],
        {value: 4, step: 1, label: "Tick Size Y:"}
    ))

    const ticksize_x_slider = view(Inputs.range(
        [0, 20],
        {value: 4, step: 1, label: "Tick Size X:"}
    ))

    const height_slider = view(Inputs.range(
        [0, 2000],
        {value: 700, step: 1, label: "Height:"}
    ))

    const width_slider = view(Inputs.range(
        [0, 2000],
        {value: 500, step: 1, label: "Width:"}
    ))

    //Heatmap plotting

    const fill_value = zscore_toggle ? 'zScore' : 'value'

    view(Plot.plot({
        color: {type: 'diverging', scheme: 'BuRd', pivot: 0, legend: true},
        marks: [
            Plot.cell(
                plotData,
                Plot.group(
                    {fill: "mean"},
                    {x: "gene", y: fill_value, fill: fill_value}
                )
                )
        ]
    }))
} else {
    console.log("Query not executed.");
}
```



<!-- 
view(Plot.plot({
        padding: padding_slider,
        marginBottom: marginbottom_slider, 
        marginLeft: marginleft_slider,
        marginRight: marginright_slider,
        height: height_slider,
        width: width_slider,
        x: {
            axis: 'bottom', label: null, tickRotate: 45, tickSize: ticksize_x_slider
        },
        y: {axis: 'left', label: null, tickSize: ticksize_y_slider},
        color: {type: 'diverging', scheme: 'BuRd', pivot: 0, legend: true},
        marks: [
            Plot.cell(
                plotData,
                Plot.group(
                    {fill: "median"},
                    {x: fill_value, y: (d) => d.gene, fill: fill_value, inset: inset_slider}
                )
                )
            //Plot.frame(),
            Plot.cell(
                plotData,
                {
                    x: fill_value, y: "gene", 
                    inset: inset_slider
                }
            )
        ]
    })) -->

<!-- ```js
const heatmap = vl.markRect({tooltip: {"content": "barplt_df"}, clip: true})
  .data(plotData)
  .encode(
    vl.y().fieldN("value"),
    vl.x().fieldO("gene"),
    vl.color().average("Fold_Exp_Day_28_30").scale({ scheme: "plasma", reverse: true }),
    // vl.size().average("precip")
  )
  .width(400)
  .render()
``` -->

<!-- 
```js
fill_value = zscore_toggle ? 'zscore' : 'log2FoldChange'
fill_label = zscore_toggle ? 'Z-score' : 'Log2FoldChange'

Plot.plot({
    padding: padding_slider,
    marginBottom: marginbottom_slider, 
    marginLeft: marginleft_slider,
    marginRight: marginright_slider,
    height: height_slider,
    width: width_slider,
    x: {
        axis: 'bottom', label: null, tickRotate: 45, tickSize: ticksize_x_slider, 
        domain: x_labels
    },
    y: {axis: 'left', label: null, tickSize: ticksize_y_slider},
    color: {type: 'diverging', scheme: 'BuRd', pivot: 0, legend: true, label: fill_label},
    marks: [
        Plot.frame(),
        Plot.cell(
            dt_hm,
            {
                x: 'obs_name', y: 'gene', 
                fill: fill_value, 
                inset: inset_slider
            }
        )
    ]
})
```
 -->




<!-- 
```js
const inset_slider = view(Inputs.range(
    [0, 10], 
    {value: 0.1, step: 0.1, label: "Inset amount:"}
))
```

```js
const padding_slider = view(Inputs.range(
    [0, 1],
    {value: 0, step: 0.01, label: "Padding:"}
))
```

```js
const marginleft_slider = view(Inputs.range(
    [0, 200],
    {value: 120, step: 1, label: "Margin Left:"}
))
```

```js
const marginbottom_slider = view(Inputs.range(
    [0, 200],
    {value: 120, step: 1, label: "Margin Bottom:"}
))
```

```js
const marginright_slider = view(Inputs.range(
    [0, 200],
    {value: 120, step: 1, label: "Margin Right:"}
))
```

```js
const ticksize_y_slider = view(Inputs.range(
    [0, 20],
    {value: 4, step: 1, label: "Tick Size Y:"}
))
```


```js
const ticksize_x_slider = view(Inputs.range(
    [0, 20],
    {value: 4, step: 1, label: "Tick Size X:"}
))
```

```js
const height_slider = view(Inputs.range(
    [0, 2000],
    {value: 700, step: 1, label: "Height:"}
))
```

```js
const width_slider = view(Inputs.range(
    [0, 2000],
    {value: 500, step: 1, label: "Width:"}
))
``` -->