<!-- 
	This is the skeleton code provided by Prof. Minsuk Kahng.
	Please feel free to revise the existing code.
-->
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

	onMount(async () => {
		const fetched = await fetch("static/prediction_results.json");
		instances = (await fetched.json()).test_instances;
		console.log(instances);

		// Transform data
		for (let k = 0; k < numClasses; ++k) {
			let binsForClass = [];
			for (let b = 0; b < numBins; ++b) {
				binsForClass.push({"class": k, "binNo": b, "instances": []});
			}
			binsByClasses.push({"class": k, "bins": binsForClass});
		}
		console.log('Bins By Classes', binsByClasses);

		colorScale = scaleOrdinal(schemeCategory10)
			.domain(instances.map(row => row["true_label"]));

		instances.forEach(instance => {
			// console.log('projection x', instance.projection[0])

		});
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
									class="point {selectedPoint == record["id"] ? 'this-one-is-selected-new-version': "unselected-circle"}"
									cx={record.projection[0]*4.5 + 100}
									cy={record.projection[1]*3.5 + 62} 
									r="4"
									style="fill: {colorScale(record["true_label"])}; stroke: {colorScale(record["predicted_label"])}"
									on:click={()=>{
										selectedPoint = record["id"];
										imgPath = "static/images/" + instances[record["id"]].filename
										selectedTrueLabel = instances[record["id"]].true_label
										selectedPrediction = instances[record["id"]].predicted_label
										selectedConfidence = instances[record["id"]].predicted_scores[selectedPrediction]
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
			<div id="score-distributions-view" class="view-panel">
				<div class="view-title">Score Distributions</div>
				<svg >


				</svg>
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
		stroke-opacity: 0.4;
	}


</style>
