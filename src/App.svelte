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
			<div id="score-distributions-view" class="view-panel" style="height:700px;">
				<div class="view-title" >Score Distributions</div>
				
				<svg id="squares" width="1300" height="510"></svg>
				
				<script type="text/javascript">
					var num_class = 10;
					var num_bin = 10;
					var num_bin_in_each_col = 4;
					var num_max_boxes_in_each_row = 11;
					var bin_size = 11; // in px

					// color (i.e., color(i) will give a hex value)
					var color = d3.scaleOrdinal(d3.schemeCategory10);

					d3.json("static/prediction_results.json").then(function(data){ 
						console.log("DDDD: ", data)

						// transform data into a nested structure
						var data_nested = [];
						for (var k = 0; k < num_class; ++k) {
							var inner_nested = [];
							for (var b = 0; b < num_bin; ++b) {
								inner_nested.push({ "class": k, "bin_no": b, "instances": [] });
							}
							data_nested.push({ "class": k, "bins": inner_nested });
						}

						// sort instances so that incorrect instances moved to the top
						data.test_instances.sort((a, b) => (Math.abs(a.true_label - a.predicted_label) > (Math.abs(b.true_label - b.predicted_label))) ? -1 : 1)

						// place data into the nested structure
						data.test_instances.forEach(d => {
							d.predicted_score = d.predicted_scores[d.predicted_label];
							var bin_no = Math.floor(d.predicted_score * num_bin);
							if (bin_no >= num_bin) {
								bin_no = num_bin - 1;
							}
							data_nested[d.predicted_label].bins[bin_no].instances.push(d);
						});

						// print out the nested structure
						console.log('data nested: ', data_nested);


						// create a column for each class
						var classes = d3.select("svg#squares")
							.selectAll("g.class_column")
							.data(data_nested)
							.enter()
							.append("g")
							.attr("class", "class_column")
							.attr("transform", (d, i) =>
								`translate(${i * (num_max_boxes_in_each_row * bin_size + 8) + 10}, 20)`);

						// title at the top
						classes.append("text")
							.attr("class", "class_title")
							.text((d, i) => `Class ${i}`)
							.attr("transform", "translate(0, -10)")
							.style("fill", (d, i) => color(i));

						// vertical line
						classes.append("line")
							.attr("x1", 0)
							.attr("y1", -2)
							.attr("x2", 0)
							.attr("y2", (bin_size + 1) * num_bin_in_each_col * num_bin)

						// for each column, create 10 bins
						var bins = classes.selectAll("g.bin")
							.data(d => d.bins)
							.enter()
							.append("g")
							.attr("class", "bin")
							.attr("transform", (d, i) =>
								`translate(2, ${(bin_size + 1) * num_bin_in_each_col * (num_bin - i - 1)})`);

						// very short horizontal line
						bins.append("line")
							.attr("x1", -3)
							.attr("y1", -2)
							.attr("x2", 0)
							.attr("y2", -2)

						bins.append("text")
							.attr("transform", "translate(-11, 1)")
							.style("fill", "#999")
							.text((d, i) => (i < 9 ? `.${i + 1}` : "1."));

						// for each bin, place boxes
						var instances = bins.selectAll("g")
							.data(d => d.instances)
							.enter()
							.append("g")
							.attr("id", d => `instance_box_${d.id}`)
							.attr("class", "instance_box")
							.attr("transform", (d, i) => `translate(${bin_size * Math.floor(i / num_bin_in_each_col)}, ${bin_size * (i % num_bin_in_each_col)})`);

						// square box using rect
						instances.append("rect")
							.attr("x", 0)
							.attr("y", 0)
							.attr("width", bin_size - 1)
							.attr("height", bin_size - 1)
							.style("fill", d => color(d.true_label));

					});
				</script>
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
