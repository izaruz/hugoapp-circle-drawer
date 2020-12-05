<template>
    <section class="crud">
        <div class="row justify-content-center">
            <div class="col-12">
                <h3>Circle Drawer</h3>
            </div>
            <div class="col-4">
                <div class="row text-center">
                    <button type="submit" @click="undoItem()" class="btn btn-purple mr-auto ml-auto">Undo</button>
                    <button type="submit" @click="redoItem()" class="btn btn-outline-purple mr-auto ml-auto">Redo</button>
                </div>
            </div>
        </div>
        <div class="row justify-content-center pt-4">
            <div class="col-6">
                <canvas id="canvas" width=600 height=600 class="canvas"></canvas>
            </div>
        </div>

        <div v-if="showModal">
            <transition name="modal">
                <div class="modal-mask">
                    <div class="modal-wrapper">
                    <div class="modal-dialog" role="document">
                        <div class="modal-content">
                            <div class="modal-header">
                                <h5 class="modal-title">Adjust diameter</h5>
                                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                                <span aria-hidden="true" @click="toggleModal()">&times;</span>
                                </button>
                            </div>
                            <div class="modal-body">
                                <input type="range" v-model="adjustRadius" @change="modifyCircle()" min="0" max="300" class="custom-range" id="adjustRadius">
                            </div>
                        </div>
                    </div>
                    </div>
                </div>
            </transition>
        </div>
        
    </section>
</template>

<script>

export default {
    name: "Drawer",
    data() {
        return {
            vueCanvas: null,
            canvasW: 0,
            canvasH: 0,
            circles: [],
            circle: {},
            circleColor: "#333",
            fillColor: "#6542f4",
            offsetX: "",
            offsetY: "",
            startX: 0,
            startY: 0,
            radius: 0,
            mouseClick: false,
            rect: "",
            showModal: false,
            adjustRadius: 0,
            selectedCircle: "",
            selectedCircleRedius: 0,
            undoRadius: {},
            redoRadius: {}
        }
    },
    mounted() {
        let canvasItem = document.getElementById("canvas")
        let canvasContext = canvasItem.getContext("2d")  
        this.rect = canvasItem.getBoundingClientRect()
        this.vueCanvas = canvasContext
        this.canvasW = canvasItem.width
        this.canvasH = canvasItem.height
        this.offsetX = this.rect.left
        this.offsetY = this.rect.top

        canvasItem.addEventListener('mousedown', this.drawMouseDown, false)
        canvasItem.addEventListener('mouseup', this.drawMouseUp, false)
        canvasItem.addEventListener('mousemove', this.drawMouseMove, false)
    },
    methods: {
        toggleModal() {
            this.showModal = !this.showModal
        },
        modifyCircle(state = true) {
            this.vueCanvas.clearRect(0, 0, this.canvasW, this.canvasH)
            this.circles.forEach((item, index) => {
                let data = {
                    canvas: this.vueCanvas,
                    fill: false,
                    fillColor: this.fillColor,
                    stroke: this.circleColor
                }
                
                if (state) {
                    this.undoRadius.index = this.selectedCircle
                    this.undoRadius.radius = item.radius
    
                    this.redoRadius.index = this.selectedCircle
                    this.redoRadius.radius = this.adjustRadius
                }
            

                if (this.selectedCircle == index) {
                    data.adjustRadius = this.adjustRadius
                }
                item.draw(data)
            })
        },
        undoItem() {
            this.selectedCircle = this.undoRadius.index
            this.adjustRadius = this.undoRadius.radius
            this.modifyCircle(false)
        },
        redoItem() {
            this.selectedCircle = this.redoRadius.index
            this.adjustRadius = this.redoRadius.radius
            this.modifyCircle(false)
        },
        drawCircle(startX, startY) {
            this.startX = startX
            this.startY = startY
            this.radius
            this.draw = function(data) {
                if (data.adjustRadius) {
                    this.radius = data.adjustRadius
                }
                data.canvas.beginPath()
                data.canvas.arc(this.startX, this.startY, this.radius, 0, 2 * Math.PI)
                if (data.fill) {
                    data.canvas.fillStyle = data.fillColor
                    data.canvas.fill()
                }
                data.canvas.lineWidth = 2
                data.canvas.strokeStyle = data.stroke
                data.canvas.stroke()
            }
        },
        drawMouseDown(e) {
            e.preventDefault()
            if (e.button == 0) {
                this.startX = parseInt(e.clientX - this.offsetX)
                this.startY = parseInt(e.clientY - this.offsetY)
                this.isMouseDown = true
                this.circle = new this.drawCircle(this.startX, this.startY)
                this.circles.push(this.circle)
            } else {
                let xCoord = parseInt(e.clientX - this.offsetX)
                let yCoord = parseInt(e.clientY - this.offsetY)
                this.circles.forEach((item, index) => {
                    let selected = this.searchCoordenate(xCoord, yCoord, item.startX, item.startY, item.radius)

                    if (selected) {
                        this.adjustRadius = item.radius
                        this.showModal = true
                        this.selectedCircle = index
                    }

                    let data = {
                        canvas: this.vueCanvas,
                        fill: selected,
                        fillColor: this.fillColor,
                        stroke: this.circleColor,
                        circleIndex: index
                    }
                    item.draw(data)
                })
            }
        },
        drawMouseUp() {
            this.isMouseDown = false
            this.circle = null
        },
        distance(startX, startY, mouseX, mouseY) {
            return Math.sqrt(Math.pow(startX - mouseX, 2) + Math.pow(startY - mouseY, 2))
        },
        searchCoordenate(x, y, cx, cy, radius) {
            let search = (x - cx) * (x - cx) + (y - cy) * (y - cy)
            return search <= radius * radius
        },
        drawMouseMove(e) {
            if (!this.isMouseDown) {
                return
            }

            let mouseX = parseInt(e.clientX - this.offsetX)
            let mouseY = parseInt(e.clientY - this.offsetY)
            this.circle.radius = this.distance(this.startX, this.startY, mouseX, mouseY)
            this.vueCanvas.clearRect(0, 0, this.canvasW, this.canvasH)
            this.circles.forEach((item) => {
                let data = {
                    canvas: this.vueCanvas,
                    fill: false,
                    fillColor: this.fillColor,
                    stroke: this.circleColor
                }
                item.draw(data)
            })
        }
    }
}
</script>

<style scoped>
.btn-purple {
    color: #fff;
    background-color: #6542f4;
    border-color: #6542f4;
}
.btn-outline-purple {
    color: #6542f4;
    border-color: #6542f4;
}
.error-input {
    background-color: #e3559a;
    color: #fefefe;
}
.active-item {
    background-color: #66bec7;
    border-radius: 5px;
    color: #fefefe;
    font-weight: 700;
}
.canvas{
    border: 1px solid #333;
}
.modal-mask {
    position: fixed;
    z-index: 9998;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, .5);
    display: table;
    -webkit-transition: opacity 500ms ease-in;
    -moz-transition: opacity 500ms ease-in;
    transition: opacity 500ms ease-in;
}
.modal-wrapper {
    display: table-cell;
    vertical-align: middle;
}
</style>