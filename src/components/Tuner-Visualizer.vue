<template>
    <div>
            <h1>{{pitch}}</h1>
            <h1>{{note}}</h1>
            <h1>{{cents}}</h1>
    </div>
</template>

<script>
    export default {
        name: "tuner-visualizer",
        props: [
            "pitch",
            "note",
            "cents"
        ],
        beforeDestroy: function () {
            window.removeEventListener('resize', this.handleResize)
        },
        mounted: function () {
            console.log('mounted tuner visualizer')
            window.addEventListener('resize', this.handleResize)
            this.myCanvas = this.$refs.canvas
            this.myCtx = this.myCanvas.getContext('2d')
            this.myCanvas.width = this.myCanvas.parentElement.getBoundingClientRect().width
            this.myCanvas.height = this.myCanvas.parentElement.getBoundingClientRect().height
        },
        methods: {
            handleResize(event) {
                this.myCanvas.width = this.myCanvas.parentElement.getBoundingClientRect().width
                this.myCanvas.height = this.myCanvas.parentElement.getBoundingClientRect().height
            },
            _fillCanvasBG: function () {
                // Resets the canvas to black
                const w = this.myCanvas.width
                const h = this.myCanvas.height
                this.myCtx.fillStyle = 'white'
                this.myCtx.fillRect(0, 0, w, h)
            },
            _drawBars: function (xPos, barHeight, barWidth) {
                this.myCtx.fillStyle = 'black'
                let yPos = (this.myCanvas.height - barHeight)
                this.myCtx.fillRect(xPos, yPos, barWidth, barHeight)
            }
        }
    }
</script>
