<template>
    <div>
        <div v-if="analyser">
            <canvas-visualizer :audio-analyser="analyser"></canvas-visualizer>
            <tuner-visualizer :note="note" :pitch="pitch" :cents="cents"></tuner-visualizer>
            <!--            <h2>{{note}} {{cents}}</h2>-->
            <VueApexCharts width="500" type="bar" :options="options" :series="series"></VueApexCharts>
        </div>
    </div>
</template>

<script>
    import tuner from "./tuner"
    import CanvasVisualizer from "./components/Canvas-visualizer";
    import TunerVisualizer from "./components/Tuner-Visualizer";
    import VueApexCharts from 'vue-apexcharts';

    export default {
        name: 'App',
        components: {TunerVisualizer, CanvasVisualizer, VueApexCharts},
        data() {
            return {
                context: null,
                playing: false,
                analyser: null,
                bufferLength: null,
                frequencyData: null,
                rafID: null,
                buf: null,
                note: null,
                pitch: null,
                cents: null,
                options: {
                    chart: {
                        id: 'vuechart-example'
                    },
                    xaxis: {
                        categories: [1991, 1992, 1993, 1994, 1995, 1996, 1997, 1998]
                    }
                },
                series: [{
                    name: 'series-1',
                    data: [30, 40, 45, 50, 49, 60, 70, 91]
                }]
            }

        },
        mounted: function () {
            try {
                navigator.getUserMedia =
                    navigator.getUserMedia ||
                    navigator.webkitGetUserMedia ||
                    navigator.mozGetUserMedia;

                // ask for an audio input
                navigator.mediaDevices.getUserMedia(
                    {
                        "audio": true
                    }).then(stream => {
                    this.initializeStream(stream)
                }, this.didntGetStream);
            } catch (e) {
                alert('getUserMedia threw exception :' + e);
            }
        },
        methods: {
            initializeStream(stream) {
                console.log('got stream!')
                this.context = new AudioContext()
                this.analyser = this.context.createAnalyser();

                const source = this.context.createMediaStreamSource(stream);
                const scriptProcessor = this.context.createScriptProcessor();

                this.analyser.fftSize = 2048;
                this.frequencyData = new Uint8Array(this.analyser.frequencyBinCount);
                this.bufferLength = this.analyser.frequencyBinCount;
                this.buf = new Float32Array(this.bufferLength);

                source.connect(this.analyser);
                this.analyser.connect(scriptProcessor);
                scriptProcessor.connect(this.context.destination)
                this.updatePitch()
            },
            updatePitch() {
                this.analyser.getFloatTimeDomainData(this.buf);
                let pitch = tuner.autoCorrelate(this.buf, this.context.sampleRate);
                let note = tuner.noteFromPitch(pitch);
                if (pitch !== -1) {
                    this.note = tuner.noteNameFromNum(tuner.noteFromPitch(pitch));
                    this.pitch = pitch;
                    this.cents = tuner.centsOffFromPitch(pitch, note);
                }

                if (!window.requestAnimationFrame)
                    window.requestAnimationFrame = window.webkitRequestAnimationFrame;
                this.rafID = window.requestAnimationFrame(this.updatePitch);
            },

            didntGetStream() {
                console.log('didnt get stream')
            },
        }
    }
</script>

<style>
    #app {
        font-family: Avenir, Helvetica, Arial, sans-serif;
        -webkit-font-smoothing: antialiased;
        -moz-osx-font-smoothing: grayscale;
        text-align: center;
        color: #2c3e50;
        margin-top: 60px;
    }
</style>
