<template>
    <div>
        <div class="instruction-window" v-if="!gameStarted">
            <h2>Use these keys to move blocks</h2>
            <img src="src/img/keys.png" alt="" rel="preload">
            <p>Press any button to continue</p>
        </div>
        <div :style="blurBackground">
            <div class="score">
                <p v-if="!gameOver">Score: {{ score }}</p>
                <p v-else>Game Over!<br> Your score: {{ score }}</p>
            </div>
            <div class="container">
                <div v-for="(row, i) in nodes" class="row">
                    <div v-for="(node, j) in row" class="node" :style="nodeStyle(i, j)"><div>{{ node }}</div></div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        name: "Game-2048",
        data(){
            return {
                nodes: [
                    [null, null, null, null],
                    [null, 2, null, null],
                    [null, null, 2, null],
                    [null, null, null, null],
                ],
                allNodesUsed: false,
                score: 0,
                gameStarted: false,
                gameOver: false
            }
        },
        methods: {
            moveBlocks(e){
                this.gameStarted = true;
                if(this.allNodesUsed){
                    this.checkGameStatus();
                }

                let callback = () => {changed = true};
                let changed = false;

                if(e.key === "ArrowLeft" || e.key.toLowerCase() === "a"){
                    for(let i = 0; i <= 3; i+=1){
                        for(let j = 0; j <= 3; j+=1){
                            this.doSwipeForSingleNode(i, j, -1, false, callback);
                        }
                    }
                } else if(e.key === "ArrowRight" || e.key.toLowerCase() === "d"){
                    for(let i = 0; i <= 3; i+=1){
                        for(let j = 3; j >= 0; j-=1){
                            this.doSwipeForSingleNode(i, j, 1, false, callback);
                        }
                    }
                } else if(e.key === "ArrowUp" || e.key.toLowerCase() === "w"){
                    for(let i = 0; i <= 3; i+=1){
                        for(let j = 0; j <= 3; j+=1){
                            this.doSwipeForSingleNode(i, j, -1, true, callback);
                        }
                    }
                } else if(e.key === "ArrowDown" || e.key.toLowerCase() === "s"){
                    for(let i = 3; i >= 0; i-=1){
                        for(let j = 0; j <= 3; j+=1){
                            this.doSwipeForSingleNode(i, j, 1, true, callback);
                        }
                    }
                }

                if(changed) this.addNewNode();
            },
            doSwipeForSingleNode(i, j, term, isVertical, callback){
                if(this.nodes[i][j] === null) return;
                if(!isVertical){
                    while(j >= 0 && j <= 3){
                        let currentNode = this.nodes[i][j];
                        let previousNode = this.nodes[i][j+term];
                        if(previousNode === null || previousNode === currentNode){
                            if(previousNode === null){
                                this.$set(this.nodes[i], j, null);
                                this.$set(this.nodes[i], j+term, currentNode);
                            }
                            if(previousNode === currentNode){
                                this.score += currentNode*2;
                                this.$set(this.nodes[i], j, null);
                                this.$set(this.nodes[i], j+term, currentNode*2);
                            }
                            callback();
                        }
                        j+=term;
                    }
                } else {
                    while(i+term >= 0 && i+term <= 3){
                        let currentNode = this.nodes[i][j];
                        let previousNode = this.nodes[i+term][j];
                        if(previousNode === null || previousNode === currentNode) {
                            if (previousNode === null) {
                                this.$set(this.nodes[i], j, null);
                                this.$set(this.nodes[i + term], j, currentNode);
                            }
                            if (previousNode === currentNode) {
                                this.score += currentNode*2;
                                this.$set(this.nodes[i], j, null);
                                this.$set(this.nodes[i + term], j, currentNode * 2);
                            }
                            callback();
                        }
                        i+=term;
                    }
                }
            },
            addNewNode(){
                let nullNodes = [];
                for(let i = 0; i <= 3; i++){
                    for(let j = 0; j <= 3; j++){
                        if(this.nodes[i][j] === null){
                            nullNodes.push([i, j]);
                        }
                    }
                }

                if(nullNodes.length === 1){
                    this.allNodesUsed = true;
                } else {
                    this.allNodesUsed = false;
                }

                if(nullNodes.length){
                    let randomNodeIdx = Math.floor(Math.random() * nullNodes.length);
                    let randomNode = nullNodes[randomNodeIdx];
                    let i = randomNode[0];
                    let j = randomNode[1];
                    let value = Math.random() > 0.9 ? 4 : 2;
                    this.$set(this.nodes[i], j, value);

                    let node = document.getElementsByClassName('node')[i*4+j];
                    node.classList.add('zoomIn', 'animated');
                    setInterval(() => {node.classList.remove('zoomIn', 'animated')}, 1000);
                }
            },
            checkGameStatus(){
                let gameOver = true;
                for(let i = 0; i <= 3; i++){
                    for(let j = 0; j <= 3; j++){
                        let currentNode = this.nodes[i][j];
                        if(i > 0){
                            if(currentNode === this.nodes[i-1][j]) gameOver = false;
                        }
                        if(i < 3){
                            if(currentNode === this.nodes[i+1][j]) gameOver = false;
                        }
                        if(j < 3){
                            if(currentNode === this.nodes[i][j+1]) gameOver = false;
                        }
                    }
                }
                this.gameOver = gameOver;
            },
            nodeStyle(i, j){
                let nodeValue = this.nodes[i][j];
                if (nodeValue === null) return;
                let backgroundColor = this.nodeBackgroundColor(i, j);
                let fontColor,
                    fontSize;

                if(nodeValue <= 4) {
                    fontColor = "#786e65"
                } else {
                    fontColor = '#f9f6f2';
                }

                if(nodeValue < 100){
                    fontSize = 60;
                } else if(nodeValue >= 100 && nodeValue < 1000){
                    fontSize = 54;
                } else {
                    fontSize = 42;
                }

                return {"background-color": backgroundColor, "color" : fontColor, 'font-size': fontSize + 'px'};
            },
            nodeBackgroundColor(i, j){
                let nodeValue = this.nodes[i][j];
                let backgroundColor;

                switch (nodeValue) {
                    case 2:
                        backgroundColor = '#eee4da';
                        break;
                    case 4:
                        backgroundColor = '#ede0c8';
                        break;
                    case 8:
                        backgroundColor = '#f2b179';
                        break;
                    case 16:
                        backgroundColor = '#f59663';
                        break;
                    case 32:
                        backgroundColor = '#f67b5f';
                        break;
                    case 64:
                        backgroundColor = '#f65d3b';
                        break;
                    case 128:
                        backgroundColor = '#edce72';
                        break;
                    case 256:
                        backgroundColor = '#edcc61';
                        break;
                    case 512:
                        backgroundColor = '#edc850';
                        break;
                    case 1024:
                        backgroundColor = '#edc43f';
                        break;
                    case 2048:
                        backgroundColor = '#edc02e';
                        break;
                    case 4096:
                        backgroundColor = '#3e3932';
                        break;
                    default:
                        backgroundColor = '#ccc0b3';
                        break;
                }
                return backgroundColor;
            },
            startGame(){
                this.gameStarted = true;
                window.removeEventListener('keydown', this.startGame);
                window.addEventListener('keydown', this.moveBlocks);
            }
        },
        computed: {
          blurBackground(){
              return this.gameStarted === false ? {filter: 'blur(8px)'} : '';
          }
        },
        mounted() {
            window.addEventListener('keydown', this.startGame);
        }
    }
</script>
