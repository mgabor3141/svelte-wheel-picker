<script>
	import WheelPickerProtoType from "$lib/WheelPickerPrototype.svelte";
	import WheelPicker from "$lib/WheelPicker.svelte";

    let perspective = 100
</script>

<div>
	<!-- <OldWheelPicker style="background-color: #272727; color: white"></OldWheelPicker> -->
	<!-- <WheelPickerProtoType perspective={perspective} fillParent={true}  height="50px" visibleOptions={30}></WheelPickerProtoType> -->
	<WheelPicker 
		options={{visibleOptions: 30}}
		fillParent={{enable:true, height: "1rem"}}
		rollOnHover={{enable: false, contract: 99, stampPage: false}}
	></WheelPicker>
</div>

<style>
	div {
		position: fixed;
		top: 25vh;
		/* left: 25vh; */
		width: 50vw;
		height: 50vh;
		border: 2px solid #101010;
		/* background-color: yellow; */
		display: flex;
		align-items: center;
		justify-content: center;
	}
</style>