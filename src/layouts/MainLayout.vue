<template>
  <div class="q-pa-md">
    <q-input
      v-model.number="nCandidates"
      v-on:change="nCandChange"
      type="number"
      filled
      style="max-width: 200px"
    />
    <table v-on:mouseleave="exitTable">
      <tr>
        <th
          v-for="c in nCandidates + 1"
          v-bind:key="c"
          v-bind:class="['cell', {active: activeCol === c - 2}]">
          {{c > 1 ? c - 1 : ''}}
        </th>
      </tr>
      <tr v-for="r in nCandidates" v-bind:key="r">
        <td
          v-for="c in nCandidates + 1"
          v-bind:key="c"
          class='cell'
          v-bind:class="[c === 1 ? 'rowname' : '', (c === 1 && activeRow === r-1) ? 'active' : '', classes[r-1][c-2]]"
          v-on:click="handleClick(r - 1, c - 2)"
          v-on:mouseenter="handleEnter(r-1, c-2)">
          {{c === 1 ? r : ''}}
        </td>
      </tr>
    </table>

    <q-btn v-on:click="resetHits">
      Reset
    </q-btn>

    <h6>
      Die glücklichen matches
    </h6>
    <ul>
      <li v-for="m in matches" v-bind:key="m">
        {{m}}
      </li>
    </ul>
  </div>
</template>

<script>
import Vue from 'vue'

export default {
  name: 'MainLayout',
  methods: {
    handleClick: function (row, col) {
      const newRow = this.hits[row].map((x, i) => i === col ? !x : x)
      Vue.set(this.hits, row, newRow)
    },
    handleEnter: function (row, col) {
      this.activeRow = row
      this.activeCol = col
    },
    exitTable: function () {
      this.activeRow = null
      this.activeCol = null
      console.log('imma out!')
    },
    nCandChange: function (e) {
      const n = +e.target.value
      if (n < 2) {
        e.preventDefault()
      }

      if (n > this.hits.length) {
        const extendedHits = this.hits.map((r, ri) => [...r, false])
        this.hits = [...extendedHits, new Array(n).fill(false)]
      }
    },
    resetHits: function () {
      this.hits = this.hits.map((r) => new Array(r.length).fill(false))
    }
  },
  data () {
    return {
      nCandidates: 3,
      activeRow: null,
      activeCol: null,
      hits: [[false, false, false], [false, false, false], [false, false, false]]
    }
  },
  computed: {
    classes: function () {
      return this.hits.map((row, ri) =>
        row.map((x, ci) => {
          if (ri === ci) {
            return 'diagonal'
          } else if (this.hits[ri][ci] && this.hits[ci][ri]) {
            return 'match'
          } else if (this.hits[ri][ci]) {
            return 'possibleMatch'
          } else if (this.activeRow === ri || this.activeCol === ci) {
            return 'active'
          }
        })
      )
    },
    matches: function () {
      const matches = []

      for (let r = 0; r < this.nCandidates; r++) {
        for (let c = r + 1; c < this.nCandidates; c++) { // heh heh... c++
          if (this.hits[r][c] && this.hits[c][r]) {
            matches.push(`${r + 1} - ${c + 1}`)
          }
        }
      }

      return matches
    }
  }
}
</script>

<style scoped>
  table {
    border-collapse: collapse;
  }

  table, th, td {
    border: 1px solid black;
  }

  .diagonal {
    background: darkslategray;
  }

  .possibleMatch {
    background: peachpuff;
  }

  .match {
    background: lightgreen;
  }

  .active {
    background: #dcdcdc;
  }

  .rowname {
    font-weight: bold;
    text-align: center;
  }

  .cell {
    width: 30px;
    height: 30px;
  }
</style>
