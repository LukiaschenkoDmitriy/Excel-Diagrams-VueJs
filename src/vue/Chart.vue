<script setup lang="ts">
import { reactive } from 'vue';

let props = defineProps({
    name: {
        type: String,
        required: true
    },
    value: {
        type: String,
        required: true
    },
    height: {
        type: String,
        required: true
    },
    widthConstant: {
        type: String,
        required: true
    }
})

let style = reactive({
    width: "0px",
    height: "0px",
    background: "none",
    border: "1px solid white",
    transform: "translate(0px,0px)",
    transition: "height 1.5s ease-in-out, width 1s ease-in-out, transform .5s ease-in-out"
});

let text = reactive({
    style : {
        'font-size': "10pt",
        position: "absolute",
        color: "white",
        padding: "10px",
        transition: "all .5s ease-out"
    },
    hidden: true
})

let initHeight = () => {
    setTimeout(() => {
        style.height = `${props.height}` + "px";
        style.width =  `${props.widthConstant}` + "px";
        style.transform = `translate(0px, ${500 - parseInt(props.height)}px)`;
    }, 100);
}

function mouseoverEvent(event: any) {
    text.hidden = false;
    style.background = 'white';
}

function mouseoutEvent(event: any) {
    text.hidden = true;
    style.background = 'none';
}

</script>

<template>
    <div :style="text.style">
        {{(text.hidden) ? "" : (name + " : " + value)}}
    </div>
    <div @mouseover="mouseoverEvent" 
         @mouseout="mouseoutEvent" 
         :style="style">{{initHeight()}}
    </div>
</template>