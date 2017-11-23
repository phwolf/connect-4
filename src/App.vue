<template>
  <div id="app">
    <div class="header">
      {{ playerNames[player] }}'s turn
    </div>
    <table>
      <tr v-for="(row, ridx) in bordConst">
        <td v-for="(col, cidx)  in row"> 
          <disc-tile 
            v-bind:state="col"
            v-bind:win="winningDiscs[ridx][cidx]"
            v-bind:index="{ridx: ridx, cidx: cidx}"
            v-on:clicked="handleDisc"
            >
            </disc-tile>  
        </td>
      </tr>
    </table>
  </div>
</template>

<script>

import Vue from 'vue'
import discTile from './discTile.vue'

export default {
  name: 'app',
  components: { discTile },
  data () {
    return {
      dimension: {x: 7, y: 6},
      player: 0,
      bordConst: null,
      winningDiscs: null,
      playerNames: ['orange', 'blue']
    }
  },
  mounted () {
    var res = [], i, j;
    // initialize bord constellation
    for (i = 0; i < this.dimension.y; i++) {
      res.push([]);
      for (j = 0; j < this.dimension.x; j++) {
        res[i].push(null);
      }
    }
    this.bordConst = res;
    // initialize winning discs
    res = [];
    for (i = 0; i < this.dimension.y; i++) {
      res.push([]);
      for (j = 0; j < this.dimension.x; j++) {
        res[i].push(false);
      }
    }
    this.winningDiscs = res;    
  },
  methods: {
    handleDisc(e) {
      // Vue.set(this.bordConst[e.ridx], e.cidx, this.player);
      this.bordConst[e.ridx][e.cidx] = this.player;
      var correctedArray = this.applyGravity(this.bordConst, this.dimension.x, this.dimension.y);
      this.bordConst = correctedArray;

      this.player = (this.player + 1) % 2;
      var result = this.checkWinCondition(this.bordConst, this.dimension.x, this.dimension.y);

      // what to do when win happend
      if (result.win) {
        result.discs.forEach( d => {
          Vue.set(this.winningDiscs[d.row], d.col, true);
        })
      }

    },
    checkWinCondition(arr, dimx, dimy) {
      var me = this, check1, check2, check3, i, j;
      // check horizontal
      var horizontalWin = function() {
        var i, j, cnt, ply;
        for (i = 0; i < dimy; i++) {
          //check row
          for (j = 0; j < dimx; j++) {
            if (j === 0){
              ply = arr[i][j];
              cnt = 1;
            } else {
              if (ply === arr[i][j] && ply != null) {
                cnt = cnt + 1;
                if (cnt === 4) {
                  // console.log('horizontal WIN!!!!', ply)
                  return {
                    win: true, 
                    player: ply, 
                    discs: [
                      {row: i, col: j-3}, 
                      {row: i, col: j-2}, 
                      {row: i, col: j-1}, 
                      {row: i, col: j}
                    ]
                  };
                }
              } else {
                ply = arr[i][j];
                cnt = 1;
              }
            }
          }
        }
        return {win: false};
      }

      var verticalWin = function() {
        var i, j, cnt, ply;
        for (i = 0; i < dimx; i++) {
          //check row
          for (j = 0; j < dimy; j++) {
            if (j === 0){
              ply = arr[j][i];
              cnt = 1;
            } else {
              if (ply === arr[j][i] && ply != null) {
                cnt = cnt + 1;
                if (cnt === 4) {
                  // console.log('vertical WIN!!!! Player: ', ply)
                  return {
                    win: true, 
                    player: ply, 
                    discs: [
                      {row: j-3, col: i}, 
                      {row: j-2, col: i}, 
                      {row: j-1, col: i}, 
                      {row: j, col: i}
                    ]
                  };
                }
              } else {
                ply = arr[j][i];
                cnt = 1;
              }
            }
          }
        }
        return {win: false};
      }

      var diagonalWin = function(row, col) {
        var maxCol = dimx - 1;
        var maxRow = dimy - 1;
        var ply;

        if (row + 3 <= maxRow && col + 3 <= maxCol) {
          ply = arr[row][col];

          if (arr[row+1][col+1] === ply && arr[row+2][col+2] === ply 
            && arr[row+3][col+3] === ply && ply !== null) {
            console.log('some diagonalWin1');
              return {
                win: true, 
                player: ply, 
                discs: [
                  {row: row, col: col}, 
                  {row: row+1, col: col+1}, 
                  {row: row+2, col: col+2}, 
                  {row: row+3, col: col+3}
                ]
              };
          }
        }

        if (row + 3 <= maxRow && col - 3 >= 0) {
          ply = arr[row][col];

          if (arr[row+1][col-1] === ply && arr[row+2][col-2] === ply 
            && arr[row+3][col-3] === ply && ply !== null) {
              console.log('some diagonalWin2');
              return {
                win: true, 
                player: ply, 
                discs: [
                  {row: row, col: col}, 
                  {row: row+1, col: col-1}, 
                  {row: row+2, col: col-2}, 
                  {row: row+3, col: col-3}
                ]
              };
          }
        }
        return false;
      }


      check1 = horizontalWin();
      if (check1.win) {
        return check1;
      } else {
        check2 = verticalWin();
        if (check2.win) {
          return check2;
        } else {
          for (i = 0; i < dimy; i++) {
            for (j = 0; j < dimx; j++) {
              check3 = diagonalWin(i,j);
              if (check3.win) {
                return check3;
              }
            }
          }
        }
      }

      return {win: false};
    },
    applyGravity(arr, dimx, dimy) {
      var row, col;
      var res = arr.slice();
      for(col = 0; col < dimx; col++) {
        for (row = 0; row < dimy; row++) {
          if(row < dimy - 1) {
            if (res[row][col] != null && res[row+1][col] == null) {
              res[row+1][col] = res[row][col];
              res[row][col] = null;
            }
          }
        }
      }
      return res;
    }
  }
}
</script>

<style>
table, th, td {
    border: 2px solid #b3b3b3;
    border-collapse: collapse;
    background-color: lightgrey;
}
table {
  margin: auto;
}

#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: darkslategrey;
  margin-top: 60px;
}

.header {
  font-size: 4vw;
  margin-bottom: 10px;
}
/*h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}*/
</style>
