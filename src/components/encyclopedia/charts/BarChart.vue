<template>
<div>
    <chart 
    :category="category" 
    :recyclable="recyclable" 
    :height="300" 
    ></chart>
</div>
</template>


<script>
import Chart from "./BarChart.js"

export default {
    name: 'BarChart',    
    components: {
        Chart
    },
    props: ['category']
};


</script>



<style>

</style>