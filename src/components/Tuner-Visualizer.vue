<template>
    <div>
        <h1>{{pitch}}</h1>
        <h1>{{note}}</h1>
        <h1>{{cents}}</h1>
        <canvas ref="canvas2"></canvas>
    </div>
</template>

<script>
    import * as d3 from 'd3'

    export default {
        name: "tuner-visualizer",
        props: [
            "pitch",
            "note",
            "cents"
        ],
        data() {
            return {
                myCanvas: null,
                myCtx: null,
                middleA: 440,
                semitone: 69,
                bufferSize: 4096,
                width: 400,
                height: 400,
                radius: 200,
                centsRange: 50,
                amountBars: 13,
            }
        },

        mounted: function () {
            this.myCanvas = this.$refs.canvas2
            this.myCtx = this.myCanvas.getContext('2d')
            this.width = this.myCanvas.width
            this.height = this.myCanvas.height
            this.draw()

        },
        watch: {
            pitch: function (oldVal, newVal) {
                this.draw();
            }
        },
        methods: {
            xLineScale: function (x) {
                let scale = d3.scaleLinear()
                    .domain([0, this.amountBars - 1])
                    .range([-this.centsRange, this.centsRange]);
                return Math.round(scale(x))
            },
            isClosest: function (i) {
                let displayedCents = this.xLineScale(i);
                let distance = Math.abs(this.cents - displayedCents);
                return distance < this.centsRange / (this.amountBars * 2)
            },
            draw: function () {
                const center = Math.floor(this.amountBars / 2);

                let BarWidth = this.width / this.amountBars;
                let BarHeight = this.height / this.amountBars;

                this._fillCanvasBG();
                this.myCtx.fillStyle = 'white'

                this.myCtx.font = this.radius * 0.05 + "px arial";
                // this.myCtx.textBaseline = "middle";
                this.myCtx.textAlign = "center";

                for (let i = 0; i < this.amountBars; i++) {
                    let xPos = BarWidth * i;
                    let yPos = this.height - BarHeight
                    let cents = this.xLineScale(i);

                    if (i === center) this.myCtx.fillStyle = 'green'
                    else if (this.isClosest(i)) this.myCtx.fillStyle = 'red'
                    else this.myCtx.fillStyle = 'white'

                    this.myCtx.fillText(cents.toString(), xPos + BarWidth / 2, yPos)
                    this.myCtx.fillRect(xPos, yPos, BarWidth * 0.8, BarHeight)
                }
            },

            _fillCanvasBG: function () {
                // Resets the canvas to black
                const w = this.myCanvas.width
                const h = this.myCanvas.height
                this.myCtx.fillStyle = 'black'
                this.myCtx.fillRect(0, 0, w, h)
            },
        }
    }
</script>
