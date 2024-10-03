<script lang="ts"> 
    
    //Types
    type Options = {
        visibleOptions?: number,
        density?: number,
        marginY ?: number,
        perspective?: number,
        staticPointerTimer?: number,
        friction?: number,
        selectedOption?:Partial<typeof data[0]>,
        overflow?: "hidden" | "visible"
        cylindrical?: boolean
    };
    type FillParentOff = {
        enable: false
        height?:undefined
    };
    type FillParentOn = {
        enable: true,
        height: string
    };
    type HoverEffectOff = {
        enable: false,
        contract?: undefined,
        stampPage?: undefined,
        transformSpeed?: undefined,
    };
    type HoverEffectOn = {
        enable: true,
        contract?: number,
        stampPage?: boolean,
        transformSpeed?: number,
    };
    
    //Props
    export let data:{value: number|string, label: string}[] = [
        {value: 5, label: "This is"},
        {value: 10, label: "only a"},
        {value: 15, label: "placeholder "},
        {value: 20, label: "you should "},
        {value: 25, label: "send your"},
        {value: 30, label: "data"},
        {value: 35, label: "in the"},
        {value: 40, label: "'wheelOptions'"},
        {value: 45, label: "prop and"},
        {value: 50, label: "configure"},
        {value: 55, label: "your"},
        {value: 60, label: "settings"},
        {value: 70, label: "with the"},
        {value: 80, label: "following"},
        {value: 90, label: "props:"},
        {value: 100, label: "marginY:number"},
        {value: 120, label: "fillParent:boolean"},
        {value: 130, label: "visibleOptions:number"},
        {value: 145, label: "perspective:number"},
        {value: 160, label: "bearingFriction:number"},
    ];
    export let rollOnHover:HoverEffectOff|HoverEffectOn = {enable:false};
    export let fillParent:FillParentOff|FillParentOn = {enable:false};
    export let options: Options;
    
        
    // Local State
    let {   visibleOptions = 7,
            density = visibleOptions*2,
            marginY  = 10,
            perspective = 0,
            staticPointerTimer = 200,
            friction = 4,
            selectedOption = data[0],
            overflow = "hidden",
            cylindrical = true,
        } = options;
        let {   enable: hoverEnabled, 
                contract = hoverEnabled ? 1 : 0, 
                stampPage =  false,
                transformSpeed = hoverEnabled ? 0.3 : 0
            } = rollOnHover;
    let {height, enable: fillParentEnabled} = fillParent;
    let leaningAngle:number[] = [];
    let wheelStyle:string[] = [];
    let yOffset:number = 0;
    let mousedown:boolean = false;
    let touchstart:boolean = false;
    let touchstartPosition:number = 1;
    let dragPosition:number = 0;
    let resetYInertia:number;
    let prevDisplacement:number = 0 ;
    let prevOffset:number = 0;
    let yInertia:number = 0;
    let hovering:boolean = !hoverEnabled;
    let optionToSelect:{angle:number, index: number} = {angle:1, index: 0}; 

    //Reactive declarations
    $: longestLabel = data.slice().sort((a, b) => a.label.length > b.label.length ? -1 : 1)[0].label;
    $: maxAngle = ((180+perspective)/visibleOptions) * data.length-1;
    $: maxScroll = -(wheelContainerHeight/data.length) * data.length-1;
    $: pickerDisplacement = Math.max(Math.min(prevDisplacement+yOffset, 0), maxScroll);
    $: yOffset = dragPosition === 0 ? 0 : dragPosition - touchstartPosition;
    $: scrollPercent = pickerDisplacement/maxScroll;
    $: rotationDisplacement = maxAngle*scrollPercent;
    $: wheelContainerTranslate = `transform: translateY(${stampPage ? 0 : pickerDisplacement}px);`;
    $: wheelWrapperScale = `transform: scale(${1+perspective/180})`;
    $: fillParentStyle = fillParentEnabled || !height 
            ? "height: 100%; width: 100%" 
            : `height: calc(${height} * ${wrapperScale}); width: ${longestLabel.length * 2}ch`;
    $: wrapperScale = (1-marginY*0.02)+perspective/180;
    $: wheelContainerTop = `calc(50% - ${(wrapperScale * wheelContainerHeight ) / ( data.length * 2 )}px)`;
    $: fillWidthFontSize = wheelWindowWidth/(longestLabel.length);
    $: fillHeightFontSize = Math.round(wheelWindowHeight/(stampPage ? data.length : visibleOptions))*0.8;
    $: fontSize = Math.min(fillWidthFontSize, fillHeightFontSize);

    // Reactive statements
    $: generateTransforms(rotationDisplacement, hovering);
    $: if (!options.density) density = visibleOptions*2;
    $: if (!fillParentEnabled && !height) {
        console.warn("The fillParent prop is enabled but its `height` property is not set, the WheelPicker will automatically fill the parent until the height is set.");
    };
    $: if (contract > 99 ) {
        contract = 99; 
        console.warn("The `contract` property in the `rollOnHover` prop cannot be higher than 99");
    };

    // Bindings
    let wheelWindowWidth:number;
    let wheelWindowHeight:number;
    let wheelContainerHeight:number;
    $: console.log(!stampPage)
    
    // Functions
    function grabWheel(e:TouchEvent|MouseEvent) {
        resetDrag(false)
        if (e.type === "mousedown") {
            mousedown = true;
            touchstartPosition = (e as MouseEvent).clientY;
        }
        else if (e.type === "touchstart") {
            touchstart = true;
            touchstartPosition = (e as TouchEvent).changedTouches[0].clientY;
        };
    }
    function releaseWheel(e:TouchEvent|MouseEvent) {
        mousedown = false;
        touchstart = false;
        yInertia = yOffset - prevOffset;
        continueWheelMomentum();
    }
    function dragWheel(e:TouchEvent|MouseEvent) {
        if (!mousedown && !touchstart) return;
        
        prevOffset = yOffset;
        
        if (mousedown) { 
            dragPosition = (e as MouseEvent).clientY;
        }
        else if (touchstart) {
            dragPosition = (e as TouchEvent).changedTouches[0].clientY;
        };

        resetYInertia = setTimeout(() => {
            prevOffset = yOffset;
        }, staticPointerTimer);
    }
    function generateTransforms(displacement:number, hovering:boolean) {
        for (let index = 0; index < data.length; index++) {

            leaningAngle[index] =  (index * ((180+perspective)/visibleOptions)) - displacement;
            const roll = dragPosition || hovering;
            const currentAngle = roll 
                    ? leaningAngle[index] :
                    leaningAngle[index]-(leaningAngle[index]*(contract/100));
            
            if (Math.abs(currentAngle) < Math.abs(optionToSelect.angle)) {
                optionToSelect.index = index;
            };
            if (index === optionToSelect.index ) {
                optionToSelect.angle = currentAngle;
            };

            const rotateX = Math.max(Math.min(currentAngle, 90), -90);

            // y = a(x-h)**2 + k where k = 100, y = 0 and x = 0
            const k = 100
            const h = Math.max(30, 100-visibleOptions*3)
            const a = -100/h**2
            const x = currentAngle

            const translateY = !roll 
                    ? 0 : currentAngle >= 0
                    ? a*(x-h)**2+k
                    : -a*(x+h)**2-k
            
            const absRotation = Math.abs(rotateX);

            const scale = roll ? (density-visibleOptions)/density+((180-absRotation)/180)**1.4 : 1;
            
            const opacity = 0.6-absRotation/150;
            
            wheelStyle[index] = `
                transform: translateY(${translateY}%) rotateX(${rotateX}deg) scale(${scale});
                transform: translateY(${translateY}%) scale(${scale}); 
                translate: 0% ${translateY}%;
                scale: ${scale};
                rotate: x ${rotateX}deg;
                opacity: ${opacity};
                `;
        }
    };
    function resetDrag(center:boolean) {
        touchstartPosition = 0;
        dragPosition = 0;
        prevDisplacement = Math.max(Math.min(prevDisplacement+yOffset, 0), maxScroll);
        prevOffset = 0;
        if (center) centerSelectedOption();
    };
    function round(num: number, decimalPoints: number = 1) {
        decimalPoints = Math.max(0, decimalPoints)
        return Math.round(10**decimalPoints*num)/10**decimalPoints
    }
    async function asyncTimeout(ms:number) {
        return new Promise(res => setTimeout(res, ms));
    };
    async function continueWheelMomentum() {
        if (mousedown || touchstart) return;
        yOffset += yInertia;
        yInertia *= (100-friction)/100;
        let absYInertia = Math.abs(yInertia);
        if (dragPosition && absYInertia > 0.1-friction/100 && pickerDisplacement < 0 && pickerDisplacement > maxScroll) {
            await asyncTimeout(8);
            continueWheelMomentum()
        } else {
            resetDrag(true);
        };
    };
    async function centerSelectedOption() {
        yOffset -= optionToSelect.angle*friction/100;
        if (Math.abs(optionToSelect.angle) > 0.5 && !dragPosition) {
            await asyncTimeout(8);
            centerSelectedOption();
        } else {
            selectedOption = data[optionToSelect.index];
            console.log("stopped");
        };
    };

</script>

<div class="wheelWindow" role="button" tabindex="0"
     style="{fillParentStyle}; font-size: {fontSize}px; overflow: {overflow};"
     on:mousedown|preventDefault|stopPropagation={grabWheel}    
     on:touchstart|preventDefault|stopPropagation|nonpassive={grabWheel}
     on:mouseup|preventDefault|stopPropagation={releaseWheel}
     on:touchend|preventDefault|stopPropagation={releaseWheel}
     on:mousemove|preventDefault|stopPropagation={dragWheel}
     on:touchmove|preventDefault|stopPropagation|nonpassive={dragWheel}
     on:mouseenter={() => {if (rollOnHover.enable) hovering = true}}
     on:mouseleave={() => {if (rollOnHover.enable && !mousedown && !touchstart) hovering = false}}
     bind:clientHeight={wheelWindowHeight} 
     bind:clientWidth={wheelWindowWidth}
>
    <div class="wheelWrapper" style="{wheelWrapperScale};">
        <div class="wheelContainer" 
            bind:clientHeight={wheelContainerHeight}
            style=" top: {stampPage ? `${marginY}px` : wheelContainerTop}; 
                    {wheelContainerTranslate};
                  "
        >
            {#each data as option, i (i)}
                <span class="wheelOption" 
                    style=" {wheelStyle[i]}; 
                            {i === optionToSelect.index  ? "opacity: 1" : ""};
                            {dragPosition ? "" : `transition: transform ${transformSpeed}s ease-in-out, translate ${transformSpeed}s ease-in-out, scale ${transformSpeed}s ease-in-out, rotate ${transformSpeed}s 0.1s ease-in-out;`}
                          "
                >
                    {option.label}
                </span>                
            {/each}
        </div>
    </div>
</div>

<style>
    .wheelWindow {
        height: 90%;
        width: 90%;
        border: 2px solid darkgray;
        cursor: grab;
	    font-family: Arial, -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu,
		Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
    }
    
    .wheelWrapper {
        height: 100%;
        width: 100%;
        position: relative;
        display: flex;
        align-items: center;
    }

    .wheelContainer {
        position: absolute;
        display: flex;
        flex-direction: column;
        justify-content: flex-start;
        align-items: center;
        width: 100%;
    }

    span {
        white-space: nowrap;
    }
</style>