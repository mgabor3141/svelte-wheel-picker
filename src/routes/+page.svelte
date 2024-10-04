<script lang="ts">
	import WheelPickerProtoType from "$lib/WheelPickerPrototype.svelte";
	import WheelPicker from "$lib/WheelPicker.svelte";
	import type {DataOption} from "$lib/WheelPicker.svelte";

    let options = {
		visibleOptions: 10,
	}
	let selectedOption:DataOption
</script>
{#if selectedOption}
  <p>Selected: {selectedOption.label} (Value: {selectedOption.value})</p>
{/if}
<div>
	<!-- <OldWheelPicker style="background-color: #272727; color: white"></OldWheelPicker> -->
	<!-- <WheelPickerProtoType perspective={perspective} fillParent={true}  height="50px" visibleOptions={30}></WheelPickerProtoType> -->
	<WheelPicker 
		bind:options={options}
		bind:selectedOption
		fillParent={{enable:true}}
	></WheelPicker>
</div>


<style>
	div {
		position: fixed;
		top: 25%;
		/* left: 25vh; */
		width: 100%;
		height: 50%;
		/* border: 2px solid #101010; */
		/* background-color: yellow; */
		display: flex;
		align-items: center;
		justify-content: center;
	}
</style>