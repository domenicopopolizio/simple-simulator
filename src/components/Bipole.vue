<template>
<div class="bipole" :style="`left:${$attrs.data.x}px;top:${$attrs.data.y}px;`" 
    :class="{rotated:$attrs.data.rotated, flipped:$attrs.data.flipped}"> 
    <img :src="`/components/${$attrs.data.type}.svg`"/>

    <svg :height="nodeRay*2" :width="nodeRay*2" class="node left" @click="nodeTouched">
        <circle :r="nodeRay" :cx="nodeRay" :cy="nodeRay" class="left" ></circle>
    </svg>
    
    <svg :height="nodeRay*2" :width="nodeRay*2" class="node right"  @click="nodeTouched">
        <circle :r="nodeRay" :cx="nodeRay" class="right"  :cy="nodeRay" ></circle>
    </svg>
 
    <span class="value"> {{$attrs.data.value}}{{um}} - {{name}}{{$attrs.data.id}} </span>
</div>
</template>

<script>
export default {
    name: 'bipole',
    attrs: {
        data: {
            required: true, 
        },  
        index: Number
    },
    computed: {
        name() {
            switch(this.$attrs.data.type) {
                case "resistor": 
                    return "R";
                    break;
                case "current": 
                    return "J";
                    break;
                case "voltage": 
                    return "E";
                    break;
            }
        },
        um() {
            switch(this.$attrs.data.type) {
                case "resistor": 
                    return "Ω";
                    break;
                case "current": 
                    return "A";
                    break;
                case "voltage": 
                    return "V";
                    break;
            }
        }
    },
    data:()=>({
        nodeRay: 4.5
    }),
    methods: {
        nodeTouched(event) {  
            event.target.style.fill = "red"
            let rect = event.target.getBoundingClientRect(); 
            this.$emit('nodeTouched', {
                index: this.index,
                data: this.$attrs.data,
                x: rect.x+rect.width/2,
                y: rect.y+rect.width/2,
                point: event.target.classList.contains("right")?this.$attrs.data.right:this.$attrs.data.left
            })
        }
    }
}
</script>

<style lang="scss">
.bipole {
    
    position: relative;
    width: 50px;

    img { 
        height: 50px;
        width: auto; 
    }

    .node {
        position: absolute;        
        top: calc(50% - 6px);
        cursor: pointer;
    }

    .left {
        left: -5px;
    }
    .right {
        right: -5px; 
    }

    .value {
        position: absolute;
        width: 100%;
        left:0;
        top: -10px;
        text-align: center;
        white-space: nowrap;
    }
    
} 
</style>