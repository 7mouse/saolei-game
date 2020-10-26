<template>
  <div id="app">
    <div id="saolei" :style="{'width':Width, 'height':Width, }" :class="{unclick:unClick}">
      <div v-for="item in boxSize" :key="item" class="boom-box" :data="item" @mousedown="onMouseDown($event)" @contextmenu.prevent="">
      </div>
    </div>
    <div id="game-setting">
      <p v-text="'成功的次数: '+success">成功</p>
      <p v-text="'失败的次数: '+fail">失败</p>
      <input v-model="inputNum" type="number" placeholder="输入阵列数" @keydown.enter="settingGame">
      <input v-model="boomNum" type="number" placeholder="输入雷数" @keydown.enter="settingGame">
      <button @click="settingGame">确定</button>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      unClick: true,
      booms: [],
      boxes: [],
      boxSize: 16,
      boomSize: 6,
      inputNum: 4,
      boomNum: 6,
      Width: 0,
      success: 0,
      fail: 0
    }
  },
  created() {
    this.fail = (localStorage.getItem('fail') == null) ? 0: localStorage.getItem('fail');
    this.success = (localStorage.getItem("success") == null) ? 0 : localStorage.getItem('fail');
  },
  mounted() {
    this.reset(this.boxSize, this.boomSize);
  },
  methods: {
    settingGame() {
      if (this.inputNum < 4) {
        alert('每行最小4格喔');
        this.inputNum = 4;
      }
      let num = +this.inputNum;
      this.Width = 50*num + 'px';
      this.reset(Math.pow(num, 2), this.boomNum);
    },
    reset(size, sum) {
      this.unClick = false;
      this.boxSize = size;
      this.boomSize = sum < size ? sum : size;
      this.initBoxes();
      this.initBooms();
    },
    initBoxes() {
      let box = document.getElementsByClassName('boom-box');
      for (let item of box) {
        item.className = 'boom-box';
        item.innerHTML = ''
      }
      this.boxes.length = this.boxSize;
      this.boxes.fill(0);
    },
    initBooms() {
      let i = 0;
      this.booms.length = 0;
      while (i < this.boomSize) {
        let num = 0|(Math.random()*this.boxSize+1);
        if (this.booms.indexOf(num) == -1) {
          i+=1;
          this.booms.push(num);
        }
      }
      let size = Math.sqrt(this.boxSize);
      for (let id of this.booms) {
        this.boxes[id-1] = NaN;
        if (id - size > 0) { //上
          this.boxes[id-size-1]++;
          if ((id-1)%size != 0) {
            this.boxes[id-size-2]++;
          }
          if (id < this.boxSize && id%size != 0) {
            this.boxes[id-size]++;
          }
        }
        if ((id-1)%size != 0) { //左
          this.boxes[id-2]++;
        }
        if (id < this.boxSize && id%size != 0) { //右
          this.boxes[id]++;
        }
        if (id + size <= this.boxSize) {
          this.boxes[id+size-1]++;
          if ((id-1)%size != 0) { //左
            this.boxes[id-2+size]++;
          }
          if (id < this.boxSize && id%size != 0) { //右
            this.boxes[id+size]++;
          }
        }
      }
    },
    onMouseDown(e) {
      switch(e.button) {
        case 0: {
          let id = +e.target.getAttribute('data'); // 字符串转数字
          if (e.target.classList.contains('mark')) {
            console.log('mark');
          } else if (this.booms.includes(id)) {
            // 踩雷
            this.gameOver();
          } else {
            // 没踩雷
            this.onClicked(id);
            this.gamePass();
          }
          break;
        }
        case 1: 
          console.log(e.target); 
          break;
        case 2: {
          e.target.classList.toggle('mark');
          break;
        }
      }
    },
    onClicked(id) {
      let el = this.findIdBox(id);
      if (el.classList.contains('clicked') || this.booms.includes(id)) {
        return;
      }
      el.classList.add('clicked')
      el.classList.add('unclick');
      el.innerHTML = this.boxes[id-1];
      let size = Math.sqrt(this.boxSize);
      if (this.boxes[id-1] == 0) {
        if (id - size > 0) { //上
          this.onClicked(id-size);
          if ((id-1)%size != 0) {
            this.onClicked(id-size-1);
          }
          if (id < this.boxSize && id%size != 0) {
            this.onClicked(id-size+1);
          }
        }
        if ((id-1)%size != 0) { //左
          this.onClicked(id-1);
        }
        if (id < this.boxSize && id%size != 0) { //右
          this.onClicked(id+1);
        }
        if (id + size <= this.boxSize) {
          this.onClicked(id+size);
          if ((id-1)%size != 0) { //左
            this.onClicked(id-1+size);
          }
          if (id < this.boxSize && id%size != 0) { //右
            this.onClicked(id+size+1);
          }
        }
      }
    },
    gameOver() {
      let box = document.getElementsByClassName('boom-box');
      for (let item of box) {
        let id = +item.getAttribute('data');
        if (this.booms.includes(id)) {
          item.classList.add('boom');
        }
      }
      
      setTimeout(()=>{ // 防止弹窗打断UI更新
        alert('Game Over!');
      }, 0)
      this.fail++;
      localStorage.setItem('fail', this.fail);
      this.unClick = true;
    },
    findIdBox(id) {
      let boomBoxes = document.getElementsByClassName('boom-box');
      for (let i = 0; i < boomBoxes.length; i++) {
        if (+boomBoxes[i].getAttribute('data') == id) {
          return boomBoxes[i];
        }
      }
    },
    gamePass() {
      let flag = true;
      for(let i = 0; i < this.boxSize; i++) {
        let id = this.boxes[i];
        console.log(id)
        if (0 <= id && id < 9) {
          let div = this.findIdBox(i+1)
          console.log(div)
          if (!div.classList.contains('clicked')) {
            flag = false;
            break;
          }
        }
      }
      if (flag) {
        for (let id of this.booms) {
          let div = this.findIdBox(id);
          div.classList.add('mark')
        }
        setTimeout(()=>{ // 防止弹窗打断UI更新
          alert('你成功了!');
        }, 0)
        this.unClick = true;
        this.success++;
        localStorage.setItem("success", this.success);
      }
    }
  }
}
</script>

<style>
html, body, div, ul{
  margin: 0;
  padding: 0;
}
ul {
  padding-left: 0;
}
li {
  list-style-type: none;
}
#saolei {
  display: flex;
  /* justify-content: space-around; */
  align-content: flex-start;
  flex-wrap: wrap;
  min-width: 200px;
  min-height: 200px;
  border: 1px solid #000;
  margin: 0 auto;
}
#saolei.unclick {
  pointer-events:none;
}
#game-setting {
  text-align: center;
}
.boom-box {
  display: flex;
  justify-content: center;
  align-items: flex-start;
  box-sizing: border-box;
  width: 50px;
  height: 50px;
  border: 1px solid #000;
}
.boom-box.clicked {
  background-color: #AAA;
}
.boom-box.clicked.unclick {
  pointer-events: none;
}
.boom-box.boom {
  background: url('./assets/Boom.svg');
  background-size: 100% 100%;
}
.boom-box.mark {
  background: url('./assets/Pin Tag Map Mark Geotag Flag.svg');
  background-size: 100% 100%;
}
.box-menu {
  float: left;
  width: 50px;
  z-index: 2;
  background-color: #DDD;
}
.box-menu li{
  text-align: center;
  color: white;
  background-color: #000;
  margin-bottom: 2px;
  cursor: pointer;
}
.box-menu li:hover {
  background-color: #ddd;
}

.box-menu li:active {
  background-color: red;
}
</style>
