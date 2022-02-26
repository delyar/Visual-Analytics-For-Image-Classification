<!-- 
	This is the skeleton code provided by Prof. Minsuk Kahng.
	Please feel free to revise the existing code.
-->

<svelte:head>
	<meta charset="utf-8">
</svelte:head>

<script>
	import { onMount } from "svelte";
	import { scaleLinear, scaleOrdinal } from "d3-scale";
	import { schemeCategory10 } from "d3-scale-chromatic";

	let colorScale;

	let instances;

	const numClasses = 10;
	const numBins = 10;
	let binsByClasses = [];

	let selectedPoint;
	let imgPath = "data:image/gif;base64,R0lGODlhAQABAAD/ACwAAAAAAQABAAACADs=";
	let selectedTrueLabel = ""
	let selectedPrediction = ""
	let selectedConfidence = 0
	let axisWidth = 800
	let verticalSpace = 63
	let svgDistributionPath; 

	function handleMouseEnter(record) {
		selectedPoint = record["id"];
		imgPath = "static/images/" + record.filename
		selectedTrueLabel = record.true_label
		selectedPrediction = record.predicted_label
		selectedConfidence = record.predicted_scores[selectedPrediction]
		svgDistributionPath = "M"
		record.predicted_scores.forEach(function (score, i) {
			svgDistributionPath += (score*axisWidth +15).toString() + "," + ((i+1)*verticalSpace).toString() + (i!==9? "L": "")
		});
	}

	onMount(async () => {
		const fetched = await fetch("static/prediction_results.json");
		instances = (await fetched.json()).test_instances;

		// Transform data
		for (let k = 0; k < numClasses; ++k) {
			let binsForClass = [];
			for (let b = 0; b < numBins; ++b) {
				binsForClass.push({"class": k, "binNo": b, "instances": []});
			}
			binsByClasses.push({"class": k, "bins": binsForClass});
		}
		
		colorScale = scaleOrdinal(schemeCategory10).domain(instances.map(row => row["true_label"]));
		
		instances.forEach(instance => {
			instance.predicted_score = instance.predicted_scores[instance.predicted_label];
				var bin_no = Math.floor(instance.predicted_score * numBins);
				if (bin_no >= numBins) {
					bin_no = numBins - 1;
				}
				binsByClasses[instance.predicted_label].bins[bin_no].instances.push(instance);
			if (instance.predicted_score < 1) {
				instance.binToBelong = parseInt(instance.predicted_score*10)
			} else {
				instance.binToBelong = 9
			}
		});
		
		console.log('Bins By Classes', binsByClasses);

	});

	

</script>

<main>
	<h1>Visual Analytics HW 3 - Delyar Tabatabai</h1>

	<div id="container">
		<div id="sidebar" style="width: 450px;">
			<div id="projection-view" class="view-panel">
				<div class="view-title">Projection View</div>
				<svg id="scatterplot-container">
					{#if instances !== undefined}	
						<g id="scatterplotData" transform="translate({50}, {20})" >
							{#each instances as record}
								<!-- svelte-ignore a11y-mouse-events-have-key-events -->
								<circle 
									id="datapoint-{record.id}"
									class="point {selectedPoint == undefined ? 'regular': (selectedPoint == record.id ? 'this-one-is-selected-new-version' : 'unselected-circle')}"
									cx={record.projection[0]*4.5 + 100}
									cy={record.projection[1]*3.5 + 62}
									r="4"
									style="fill: {colorScale(record["true_label"])}; stroke: {colorScale(record["predicted_label"])}"
									on:mouseenter={()=>{
										handleMouseEnter(record);
									}}
									on:mouseleave={()=>{
										selectedPoint=undefined
										imgPath = "data:image/gif;base64,R0lGODlhAQABAAD/ACwAAAAAAQABAAACADs="
									}}
								> <title>{record["true_label"]}</title></circle>
							{/each}
						</g>
					{/if}

				</svg>
			</div>
			<div id="selected-image-view" class="view-panel">
				<div class="view-title">Selected Image</div>
				<div id="selected-image-view-content">
					<img style="height:90px; width: 90px; float: left" src={imgPath} alt="No points selected">
					<div style="float: right">
						{#if selectedPoint !== undefined}
							<p>ID: {selectedPoint}</p>
							<p>Labled as {selectedTrueLabel}</p>
							<p>Predicted as {selectedPrediction} (Confidence: {selectedConfidence})</p>
						{/if}
					</div>
				</div>
			</div>
		</div>

		<div id="main-section" style="width: 1000px;">
			<div id="score-distributions-view" class="view-panel" style="height:760px;">
				<div class="view-title" >Score Distributions</div>
				
				<div style="float: left; width:10%;height:100%;padding-top:25px;">
					{#if instances !== undefined}	
						{#each binsByClasses as bin}
							<div style="height:6.3%;width:100%; margin-top:15px;">
								<div style="text-align: right; display:inline;float:right">
									<b style="margin:0; float: left; color:#565656;text-align: right; ">Class </b>
									<b style="background-color: {colorScale(bin.class)}; width:5px;"> {bin.class}</b>
								</div>
								<p style="margin:0;font-size: 12px;text-align: right;float:right">Labeled as {bin.class}</p>
								<p style="margin:0;font-size: 12px;text-align: right;float:right">Predicted as {bin.class}</p>
							</div>
						{/each}
					{/if}

				</div>
				
				<div style="float: right; width:90%; height:100%;">
					<div id="main-axis" style="height: 20px; width:100%;padding-left:20px">
						<script type="text/javascript">
							var width = 900,height = 20;
	
							var data = [0.0, 0.1, 0.2, 0.3, 1.0];
							
							// Append SVG 
							var svg = d3.select("#main-axis")
										.append("svg")
										.attr("width", width)
										.attr("height", height);
	
							// Create scale
							var scale = d3.scaleLinear()
										.domain([d3.min(data), d3.max(data)])
										.range([0, width - 100]);
	
							// Add scales to axis
							var x_axis = d3.axisBottom()
										.scale(scale);
	
							//Append group and insert axis
							svg.append("g").call(x_axis);
						</script>
					</div>
					<svg id="container" style=" width:100%; height:100%;">
						{#if instances !== undefined}	
							<path d="{svgDistributionPath}" fill="none" stroke="gray" stroke-width="{selectedPoint == undefined? 0 : 1}" />
							{#each binsByClasses as row,i}
								<rect x="15" y="{(i+1)*verticalSpace}" width="94%" height="0.5" fill='#565656'></rect>
								{#each {length:10} as _,j}
									{#each row.bins[j]["instances"] as binInstance,z}
										{#if z < 8}
											<image x="{15 + ((binInstance.binToBelong/10)*800) + (z*9)}" y="{(i)*verticalSpace + 50}" href="{"static/images/" + binInstance.filename}"  width="9" height="9"/>
											<rect x="{15 + ((binInstance.binToBelong/10)*800) + (z*9)}" y="{(i)*verticalSpace + 50}" width="9" height="9" fill='{colorScale(binInstance["predicted_label"])}' opacity="0.6" 
											on:mouseenter={()=>{
												handleMouseEnter(binInstance);
											}}
											on:mouseleave={()=>{
												selectedPoint=undefined
												imgPath = "data:image/gif;base64,R0lGODlhAQABAAD/ACwAAAAAAQABAAACADs="
											}}
											></rect>
										{:else if z> 7 && z<16}
											<image x="{15 + ((binInstance.binToBelong/10)*800) + ((z-8)*9)}" y="{(i)*verticalSpace + 42}" href="{"static/images/" + binInstance.filename}"  width="9" height="9"/>
											<rect x="{15 + ((binInstance.binToBelong/10)*800) + ((z-8)*9)}" y="{(i)*verticalSpace + 42}" width="9" height="9" fill='{colorScale(binInstance["predicted_label"])}' opacity="0.6"
											on:mouseenter={()=>{
												handleMouseEnter(binInstance);
											}}
											on:mouseleave={()=>{
												selectedPoint=undefined
												imgPath = "data:image/gif;base64,R0lGODlhAQABAAD/ACwAAAAAAQABAAACADs="
											}}></rect>
										{:else if z>15 && z<24}
											<image x="{15 + ((binInstance.binToBelong/10)*800) + ((z-16)*9)}" y="{(i)*verticalSpace + 34}" href="{"static/images/" + binInstance.filename}"  width="9" height="9"/>
											<rect x="{15 + ((binInstance.binToBelong/10)*800) + ((z-16)*9)}" y="{(i)*verticalSpace + 34}" width="9" height="9" fill='{colorScale(binInstance["predicted_label"])}' opacity="0.6"
											on:mouseenter={()=>{
												handleMouseEnter(binInstance);
											}}
											on:mouseleave={()=>{
												selectedPoint=undefined
												imgPath = "data:image/gif;base64,R0lGODlhAQABAAD/ACwAAAAAAQABAAACADs="
											}}></rect>
										{:else if z>23 && z<32}
											<image x="{15 + ((binInstance.binToBelong/10)*800) + ((z-24)*9)}" y="{(i)*verticalSpace + 26}" href="{"static/images/" + binInstance.filename}"  width="9" height="9"/>
											<rect x="{15 + ((binInstance.binToBelong/10)*800) + ((z-24)*9)}" y="{(i)*verticalSpace + 26}" width="9" height="9" fill='{colorScale(binInstance["predicted_label"])}' opacity="0.6"
											on:mouseenter={()=>{
												handleMouseEnter(binInstance);
											}}
											on:mouseleave={()=>{
												selectedPoint=undefined
												imgPath = "data:image/gif;base64,R0lGODlhAQABAAD/ACwAAAAAAQABAAACADs="
											}}></rect>
										{:else if z>31 && z<40}
											<image x="{15 + ((binInstance.binToBelong/10)*800) + ((z-32)*9)}" y="{(i)*verticalSpace + 18}" href="{"static/images/" + binInstance.filename}"  width="9" height="9"/>
											<rect x="{15 + ((binInstance.binToBelong/10)*800) + ((z-32)*9)}" y="{(i)*verticalSpace + 18}" width="9" height="9" fill='{colorScale(binInstance["predicted_label"])}' opacity="0.6"
											on:mouseenter={()=>{
												handleMouseEnter(binInstance);
											}}
											on:mouseleave={()=>{
												selectedPoint=undefined
												imgPath = "data:image/gif;base64,R0lGODlhAQABAAD/ACwAAAAAAQABAAACADs="
											}}></rect>
										{:else}
											<image x="{15 + ((binInstance.binToBelong/10)*800) + ((z-40)*9)}" y="{(i)*verticalSpace + 10}" href="{"static/images/" + binInstance.filename}"  width="9" height="9"/>
											<rect x="{15 + ((binInstance.binToBelong/10)*800) + ((z-40)*9)}" y="{(i)*verticalSpace + 10}" width="9" height="9" fill='{colorScale(binInstance["predicted_label"])}' opacity="0.6"
											on:mouseenter={()=>{
												handleMouseEnter(binInstance);
											}}
											on:mouseleave={()=>{
												selectedPoint=undefined
												imgPath = "data:image/gif;base64,R0lGODlhAQABAAD/ACwAAAAAAQABAAACADs="
											}}></rect>
										{/if}
									{/each}
									<!-- <text x="{40 + (j*80)}" y="{35 + (i*65)}">{row.bins[j]["instances"].length}</text> -->
									<rect x="{15 + (j*80)}" y="{(i+1)*verticalSpace}" width="1" height="4" fill='black'></rect> 
									<rect x="{815}" y="{(i+1)*verticalSpace}" width="1" height="4" fill='black'></rect> 
								{/each}
							{/each}
						{/if}
					</svg>
				</div> 
			</div>
		</div>
	</div>
</main>

<style>
	h1 {
		font-size: 1.3rem;
		font-weight: 500;
		margin-top: 0;
	}
	#container {
		display: flex;
	}
	#sidebar, #main-section {
		display: flex;
		flex-direction: column;
	}
	.view-panel {
		border: 2px solid #eee;
		margin-bottom: 10px;
		margin-right: 10px;
	}
	.view-title {
		background-color: #f3f3f3;
		font-size: 1.0rem;
		margin-bottom: 8px;
		padding: 3px 8px 5px 10px;
	}
	#selected-image-view-content {
		height: 100px;
		padding: 15px;
	}
	svg {
		margin: 5px;
	}
	circle.this-one-is-selected-new-version {
		stroke-width: 3;
		fill-opacity: 0.9;
		stroke-opacity: 1;
		r: 12
	}

	circle.unselected-circle {
		stroke-width: 1;
		fill-opacity: 0.3;
		stroke-opacity: 0.1;
		r:2
	}

	circle.regular {
		stroke-width: 1;
		fill-opacity: 0.4;
		stroke-opacity: 0.4;
		r:2
	}


</style>
