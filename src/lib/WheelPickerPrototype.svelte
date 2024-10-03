<script lang='ts'>
	import { onMount } from "svelte";

    export let data:{value: number|string, label: string}[] = [
        {value: 5, label: "This is"},
        {value: 10, label: "only a"},
        {value: 15, label: "placeholder "},
        {value: 20, label: "you should "},
        {value: 25, label: "send your"},
        {value: 30, label: "data"},
        {value: 35, label: "in the"},
        {value: 40, label: "'wheelOptions'"},
        {value: 45, label: "prop"},
        {value: 50, label: "Lorem"},
        {value: 55, label: "Ipsum"},
        {value: 60, label: "Dolor"},
        {value: 70, label: "Simet"},
        {value: 80, label: "1:20 Hours"},
        {value: 90, label: "1:30 Hours"},
        {value: 100, label: "1:40 Hours"},
        {value: 120, label: "1:50 Hours"},
        {value: 130, label: "2:00 Hours"},
        {value: 145, label: "2:15 Hours"},
        {value: 160, label: "2:30 Hours"},
        {value: 175, label: "2:45 Hours"},
        {value: 190, label: "3:00 Hours"},
        {value: 210, label: "3:20 Hours"},
        {value: 230, label: "3:40 Hours"},
        {value: 250, label: "4:00 Hours"}
    ]

    let wheelWindowHeight:number
    export let marginY = 10
    export let fillParent:boolean = false
    export let height:string
    $: fillParentStyle = fillParent || !height 
                            ? "height: 100%; width: 100%" 
                            : `height: calc(${height} * ${wrapperScale}); width: ${data.slice().sort((a, b) => a.label.length > b.label.length ? -1 : 1)[0].label.length * 2}ch`

    let wheelContainerHeight:number
    $: fontSize = Math.round((wheelContainerHeight/visibleOptions)*0.8)
    $: wrapperScale = (1-marginY*0.01)+perspective/180
    $: wheelWindowTop =`calc(50% - ${(wrapperScale*wheelWindowHeight)/(data.length*2)}px)`
    
    let mousedown:boolean = false
    let touchstart:boolean = false
    let touchstartPosition:number = 0
    function grabWheel(e:TouchEvent|MouseEvent) {
        resetDrag()

        if (e.type === "mousedown") {
            mousedown=true
            touchstartPosition = (e as MouseEvent).clientY
        }
        else if (e.type === "touchstart") {
            touchstart = true
            touchstartPosition = (e as TouchEvent).changedTouches[0].clientY
        }
    }

    function releaseWheel(e:TouchEvent|MouseEvent) {
        mousedown = false;
        touchstart = false;
        yInertia = yOffset - previousYOffset
        continueWheelMomentum()
    }

    let dragPosition:number = 0
    function dragWheel(e:TouchEvent|MouseEvent) {
        if (!mousedown && !touchstart) return
        
        previousYOffset = yOffset
        
        if (mousedown) { 
            dragPosition = (e as MouseEvent).clientY
        }
        else if (touchstart) {
            dragPosition = (e as TouchEvent).changedTouches[0].clientY
        }
        
        /** resetYInertia will reassign the previousYOffset if the drag doesn't*/
        resetYInertia = setTimeout(() => {
            previousYOffset = yOffset
        }, 100)
    }

    $: yOffset = dragPosition === 0 ? 0 : dragPosition - touchstartPosition

    let leaningAngle:number[] = [] //stores the leaning angle of each option for the 3d effect
    export let visibleOptions = data.length/2 
    let wheelStyle:string[] = []
    $: wheelRadius = Math.max(90, 200)
    let yDisplacement = 0 //stores the last vertical displacement to maintain the position
    $: maxScroll = -(wheelWindowHeight/data.length) * (data.length-1)
    $: pickerScroll =   (yDisplacement+yOffset) > 0 ? 0 : //will always be negative
                        (yDisplacement+yOffset) <  maxScroll ? 
                        maxScroll : yDisplacement+yOffset
    
    $: maxAngle = ((180+perspective)/visibleOptions) * (data.length-1)
    $: scrollPercent = pickerScroll/maxScroll
    $: rotationDisplacement = maxAngle*scrollPercent
    

    
    /** Prop number that sets the perspective, may be positive or negative. Positive values makes it seem closer, negative makes it seem farther */
    export let perspective = 50
    function generateTransforms() {
        for (let index = 0; index < data.length; index++) {
            leaningAngle[index] =  (index * ((180+perspective)/visibleOptions)) - rotationDisplacement
            const currentAngle = leaningAngle[index]
            
            if (Math.abs(currentAngle) <= Math.abs(optionToSelect.angle)) {
                optionToSelect.index = index
            }
            if (index === optionToSelect.index ) {
                optionToSelect.angle = currentAngle
            }

            const rotateX = currentAngle >= 90 ? 90 : currentAngle <= -90 ? -90 : currentAngle
            //turn to Math.max(Math.min(currentAngle, 90), -90)

            const absRotation = Math.abs(rotateX)

            const scale = (30-visibleOptions)/30+((200-absRotation)/200)**1.4
            const translateY = rotateX*visibleOptions/3


            const opacity = absRotation < 10 ? 1-absRotation/50 : 0.7-absRotation/150
            
            wheelStyle[index] = `transform: rotateX(${rotateX}deg) translateY(${translateY}%) scale(${scale}) ; opacity: ${opacity}; --op:${absRotation}`
        }
    } 

    onMount(() => {
        generateTransforms()
        yOffset++
    })
    $: if (yOffset) generateTransforms()

    $: wheelWrapperTranslate = `transform: translateY(${pickerScroll}px)`

    let previousYOffset:number = 0
    let yInertia:number = 0
    let resetYInertia:number 
    let optionToSelect:{angle:number, index: number} = {angle:1, index: 0} 

    async function asyncTimeout(ms:number) {
        return new Promise(res => setTimeout(res, ms))
    }

    export let bearingFriction:number = 3
    async function continueWheelMomentum() {
        if (mousedown || touchstart) return
        yOffset += yInertia
        yInertia *= (100-bearingFriction)/100
        let absYInertia = Math.abs(yInertia)
        if (absYInertia > 0.1 && pickerScroll < 0 && pickerScroll > maxScroll) {
            await asyncTimeout(8)
            continueWheelMomentum()
        } else {
            resetDrag()
        }
    }

    function resetDrag() {
        touchstartPosition = 0
        dragPosition = 0
        previousYOffset = 0
        pickerScroll >= 0 ? 
            yDisplacement = 0 : 
        pickerScroll <= maxScroll ? 
            yDisplacement = maxScroll : 
            yDisplacement += yOffset
    }
    
    // $: console.log(`%cmousedown: ${mousedown}`, `color: ${mousedown ? 'green' : 'red'}`)
    // $: console.log(`%ctouchstart: ${touchstart}`, `color: ${touchstart ? 'green' : 'red'}`)

</script>

<button class="wheelPickerContainer" 
    style="{fillParentStyle}; --visibleOptions: {visibleOptions}; font-size: {fontSize}px"
    on:mousedown|stopPropagation|preventDefault={grabWheel}
    on:touchstart|stopPropagation|preventDefault|nonpassive={grabWheel}
    on:mouseup|stopPropagation|preventDefault={releaseWheel}
    on:touchend|stopPropagation|preventDefault|nonpassive={releaseWheel}
    on:mousemove|stopPropagation|preventDefault={dragWheel}
    on:touchmove|stopPropagation|preventDefault|nonpassive={dragWheel}
    bind:clientHeight={wheelContainerHeight}
>
<!-- do the accessibility -->
    <button class="wheelWrapper" 
        style="{fillParentStyle}; transform: scale({wrapperScale})"
    >
        <div class="wheelWindow" 
            style="top: {wheelWindowTop}; {wheelWrapperTranslate}"
            bind:clientHeight={wheelWindowHeight} 
        >
            {#each data as option, i (i)}
                <span class="wheelOption " 
                    style="{wheelStyle[i]}"
                >
                    {option.label}
                </span>
            {/each}
        </div>
    </button>
</button>

<style>

    button {
    	--font-body: Arial, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu,
		Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
        background: unset;
        color: unset;
        border: unset;
        font-size: unset;
    }

    .wheelPickerContainer {
        /* overflow: hidden; */

        cursor: grab;
        background-color: #888888;
        display: flex;
        justify-content: center;
        align-items: center;
    }

    .wheelWrapper {
        height: 100%;
        width: 100%;
        padding: 10% 0;
        display: flex;
        justify-content: center;
    }

    .wheelWindow {
        display: flex;
        flex-direction: column;
        justify-content: flex-start;
        align-items: center;
        position: absolute;
        width: 100%;
        overflow: visible;
        scroll-snap-type: y proximity; 
    }

    .wheelOption {
        display: flex;
        justify-content: center;
        align-items: center;
    }
</style>