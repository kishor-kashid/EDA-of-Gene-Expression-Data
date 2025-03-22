---
theme: dashboard
title: Gene Selection Template
toc: false
---

## Gene Selection template
```js
import * as aq from "npm:arquero";
import {DuckDBClient} from "npm:@observablehq/duckdb"
import * as duckdb from "npm:@duckdb/duckdb-wasm"
import * as d3 from "npm:d3";
```
```js
const gene_expression_matrix = FileAttachment("Demo_Data.csv").csv()

```


```js
console.log("gene_expression_matrix:", gene_expression_matrix)
gene_expression_matrix
```

```js
const geneArray = [];

const header = Object.keys(gene_expression_matrix[0]);

// Remove non-gene columns: 'Sample_Id', 'race', 'ethnicity', 'study_accession', 'vaccine', 'gender'
const nonGeneColumns = ['Sample_Id', 'race', 'ethnicity', 'study_accession', 'vaccine', 'gender'];
const geneNames = header.filter(column => !nonGeneColumns.includes(column));

// Append gene names to geneArray if they are not already present
geneNames.forEach(gene => {
  if (!geneArray.includes(gene)) {
    geneArray.push(gene);
  }
});

console.log("Updated geneArray:", geneArray);
```



## Working


<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Search Bar with Multiple Selection</title>
    <style>
        /* Add some basic styling */
        .suggestions {
            border: 1px solid #ccc;
            max-height: 150px;
            overflow-y: auto;
        }
        .suggestion-item {
            padding: 5px;
            cursor: pointer;
        }
        .suggestion-item:hover {
            background-color: #f0f0f0;
        }
        .selected-values {
            margin-top: 10px;
        }
        .selected-item {
            display: inline-block;
            background-color: #007bff;
            color: #fff;
            padding: 5px;
            margin-right: 5px;
            margin-bottom: 5px;
            border-radius: 5px;
        }
    </style>
</head>
<body>
    <input type="text" id="searchBar" placeholder="Type to search for gene...">
    <div id="suggestions" class="suggestions"></div>
    <input type="text" id="genderSearchBar" placeholder="Type to search for gender...">
    <div id="genderSuggestions" class="suggestions"></div>
    <input type="text" id="vaccineSearchBar" placeholder="Type to search for vaccine...">
    <div id="vaccineSuggestions" class="suggestions"></div>
    <div id="selectedValues" class="selected-values"></div>
    <button id="searchButton">Search</button> <!-- Search Button -->
</body>
</html>

```js
        console.log("Inside Script geneNames", geneNames);
        let selectedRows = [];
        let selectedGenders = []; // To store selected genders
        let selectedVaccines = []; // To store selected vaccines

        // Initialize variables
        const searchBar = document.getElementById('searchBar');
        const genderSearchBar = document.getElementById('genderSearchBar');
        const vaccineSearchBar = document.getElementById('vaccineSearchBar');

        const suggestionsDiv = document.getElementById('suggestions');
        const genderSuggestionsDiv = document.getElementById('genderSuggestions');
        const vaccineSuggestionsDiv = document.getElementById('vaccineSuggestions');
        const selectedValuesDiv = document.getElementById('selectedValues');
        const searchButton = document.getElementById('searchButton'); // Search Button

        // Filter and show suggestions as the user types in gene search bar
        searchBar.addEventListener('input', function () {
            const query = this.value.toLowerCase();
            suggestionsDiv.innerHTML = '';

            if (query) {
                const filteredData = geneNames.filter(item => item.toLowerCase().includes(query));
                filteredData.forEach(item => {
                    const suggestionItem = document.createElement('div');
                    suggestionItem.classList.add('suggestion-item');
                    suggestionItem.textContent = item;
                    suggestionItem.addEventListener('click', function () {
                        addSelectedValue(item);
                        filterDataByGene(item);
                    });
                    suggestionsDiv.appendChild(suggestionItem);
                });
            }
        });

        // Filter and show suggestions as the user types in gender search bar
        genderSearchBar.addEventListener('input', function () {
            const query = this.value.toLowerCase();
            genderSuggestionsDiv.innerHTML = '';

            if (query) {
                const filteredData = Array.from(new Set(gene_expression_matrix.map(item => item.gender)))
                    .filter(item => item.toLowerCase().includes(query));
                filteredData.forEach(item => {
                    const suggestionItem = document.createElement('div');
                    suggestionItem.classList.add('suggestion-item');
                    suggestionItem.textContent = item;
                    suggestionItem.addEventListener('click', function () {
                        addSelectedValue(item);
                        addSelectedGender(item); // Add to selected genders
                    });
                    genderSuggestionsDiv.appendChild(suggestionItem);
                });
            }
        });

        // Filter and show suggestions as the user types in vaccine search bar
        vaccineSearchBar.addEventListener('input', function () {
            const query = this.value.toLowerCase();
            vaccineSuggestionsDiv.innerHTML = '';

            if (query) {
                const filteredData = Array.from(new Set(gene_expression_matrix.map(item => item.vaccine)))
                    .filter(item => item.toLowerCase().includes(query));
                filteredData.forEach(item => {
                    const suggestionItem = document.createElement('div');
                    suggestionItem.classList.add('suggestion-item');
                    suggestionItem.textContent = item;
                    suggestionItem.addEventListener('click', function () {
                        addSelectedValue(item);
                        addSelectedVaccine(item); // Add to selected vaccines
                    });
                    vaccineSuggestionsDiv.appendChild(suggestionItem);
                });
            }
        });

        // Add the selected value to the selectedValuesDiv
        function addSelectedValue(value) {
            if (!Array.from(selectedValuesDiv.children).some(item => item.textContent === value)) {
                const selectedItem = document.createElement('div');
                selectedItem.classList.add('selected-item');
                selectedItem.textContent = value;
                selectedValuesDiv.appendChild(selectedItem);
            }
        }

        // Add the selected gender to the selectedGenders array
        function addSelectedGender(gender) {
            if (!selectedGenders.includes(gender)) {
                selectedGenders.push(gender);
            }
        }

        // Add the selected vaccine to the selectedVaccines array
        function addSelectedVaccine(vaccine) {
            if (!selectedVaccines.includes(vaccine)) {
                selectedVaccines.push(vaccine);
            }
        }

        // Function: Filter data by gene name and aggregate all values for multiple selections
        function filterDataByGene(geneName) {
            // Find all rows in gene_expression_matrix for the given geneName
            const filteredRows = gene_expression_matrix.map(row => ({
                gene_name: geneName,
                value: row[geneName],
                vaccine: row.vaccine,
                gender: row.gender
            }));

            // Check if the selectedRows already contains any of the rows for this geneName
            const existingGeneRows = selectedRows.filter(row => row.gene_name === geneName);

            // Determine which rows to add by excluding those already present
            const newRows = filteredRows.filter(row => !existingGeneRows.some(existingRow => existingRow.value === row.value));

            // Append only new rows to selectedRows
            selectedRows = [...selectedRows, ...newRows];

            console.log('Aggregated Selected Rows for Multiple Gene Selections:', selectedRows);
        }

        // Function: Filter data by gender, supporting multiple gender selections
        function filterDataByGender() {
            let filteredRows;

            if (selectedGenders.length > 0) {
                if (selectedRows.length > 0) {
                    // If gene_name is selected, filter within the selectedRows by multiple genders
                    filteredRows = selectedRows.filter(row => selectedGenders.includes(row.gender));
                } else {
                    // If no gene_name is selected, filter within the entire gene_expression_matrix by multiple genders
                    filteredRows = gene_expression_matrix.filter(row => selectedGenders.includes(row.gender));
                }
            } else {
                // If no genders are selected, keep current selected rows
                filteredRows = selectedRows;
            }

            selectedRows = filteredRows; // Update selectedRows to reflect the filtered data by gender
            console.log('Selected Rows by Gender (Multiple Selection):', selectedRows);
        }

        // Function: Filter data by vaccine, supporting multiple vaccine selections
        function filterDataByVaccine() {
            let filteredRows;

            if (selectedVaccines.length > 0) {
                if (selectedRows.length > 0) {
                    // If gene_name is selected, filter within the selectedRows by multiple vaccines
                    filteredRows = selectedRows.filter(row => selectedVaccines.includes(row.vaccine));
                } else {
                    // If no gene_name is selected, filter within the entire gene_expression_matrix by multiple vaccines
                    filteredRows = gene_expression_matrix.filter(row => selectedVaccines.includes(row.vaccine));
                }
            } else {
                // If no vaccines are selected, keep current selected rows
                filteredRows = selectedRows;
            }

            selectedRows = filteredRows; // Update selectedRows to reflect the filtered data by vaccine
            console.log('Selected Rows by Vaccine (Multiple Selection):', selectedRows);
        }

        // Function to calculate the Z-score for each value in selectedRows
        function calculateZScores() {
            // Extract the values to calculate mean and standard deviation
            const values = selectedRows.map(row => row.value);

            // Calculate mean and standard deviation using d3
            const mean = d3.mean(values);
            const standardDeviation = d3.deviation(values);

            // Calculate Z-score for each value and add it to the row object
            selectedRows = selectedRows.map(row => {
                const zScore = (row.value - mean) / standardDeviation;
                return { ...row, zScore: zScore };
            });

            console.log('Selected Rows with Z-Scores:', selectedRows);
        }

        // Event listener for the search button
        searchButton.addEventListener('click', function () {
            // Apply filters in sequence
            if (selectedRows.length > 0) {
                filterDataByGender();
                filterDataByVaccine();
            } else {
                // If no gene names are selected, start with full matrix and filter by gender and vaccine
                selectedRows = gene_expression_matrix;
                filterDataByGender();
                filterDataByVaccine();
            }
            calculateZScores();
            console.log('Final Filtered Rows:', selectedRows);
            view(selectedRows); // Assuming view is a function to display the final results

        });
```

