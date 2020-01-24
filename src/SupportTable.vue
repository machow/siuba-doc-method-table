<template>
  <div class = "support-table">
    <h1><a href="https://github.com/machow/siuba">Siuba</a></h1>
    <h2>Table of supported Pandas Methods</h2>
    <input type="checkbox" v-model="showUnplanned"><span>Show unsupported methods</span><br>
    <button @click="reset()">reset</button>
    <table class="methods">
      <tr>
          <th v-for="colname in colnames"
              v-bind:key="colname"
              @click="toggleSortKey(colname)"
              >
            {{colname}}
            <span class="arrow-holder">
              <i v-show="getSortOrder(colname) != 'desc'"
                  class="arrow asc"></i>
              <i v-show="getSortOrder(colname) != 'asc'"
                  class="arrow desc"></i>
            </span>
          </th>
      </tr>
      <tbody v-for="(row, indx) in rows" v-bind:key="row.name">
        <tr @click="() => toggleRow(row)"
             :class="{expanded: row.expanded}"
             class="main-row"
             >
          <td>
            <template v-if="(rows[indx-1] || {}).category != row.category">
              {{row.category}}
            </template>
          </td>
          <td><a :href="getPandasDocUrl(row.name)">{{row.name}}</a></td>
          <td>{{row['fast grouped']}}</td>
          <td>{{row['postgresql']}}</td>
          <td>{{row['note']}}</td>
        </tr>
        <tr v-if="row.expanded"
            class="example"
            >
          <td></td>
          <td colspan="4">
            <h2>Examples</h2>
            <div v-for="(example, ii) in row.examples" :key="ii">
              <h3>{{example.name}}</h3>
              <div v-html="example.input"></div>
              <div class="rendered_html" 
                  v-html="example.output"></div>
            </div>
          </td>
        </tr>
      </tbody>
    </table>
  </div>
</template>

<script>
const sort = require('fast-sort').default;

const data = require('../data.json');


console.log(sort)  // eslint-disable-line

//const rows = {}

const rows = data
  .map(v => Object.assign({expanded: false}, v))


module.exports = {
  data: function () {
    return {
      colnames: ['category', 'name', 'fast grouped', 'postgresql', 'note'],
      rowsRaw: rows,
      showUnplanned: false,
      sortKeys: [],
    }
  },
  computed: {
    rows () {
      var result = this.rowsRaw;

      if (!this.showUnplanned) {
        result = result.filter(v => v['fast grouped'] != "❌")
      }

      if (this.sortKeys.length) {
          result = sort(result).asc(this.sortKeys)
      }

      return result
    }
  },
  methods: {
    toggleRow (row) {
      console.log('updating')  // eslint-disable-line
      row.expanded = !row.expanded
    },
    toggleSortKey (key) {
      console.log(key)  // eslint-disable-line
      const keyIndx = this.sortKeys.findIndex(v => v.asc == key || v.desc == key);

      if (keyIndx != -1) {
        const newOrder = 'asc' in this.sortKeys[keyIndx] ? 'desc' : 'asc'

        console.log("flipping order") // eslint-disable-line
        this.$set(this.sortKeys, keyIndx, {[newOrder]: key})

      }
      else {
        this.sortKeys.push({asc: key})
      }
    },
    getSortOrder (key) {
      const keyIndx = this.sortKeys.findIndex(v => v.asc == key || v.desc == key);
      if (keyIndx != -1) {
        return 'asc' in this.sortKeys[keyIndx] ? 'asc' : 'desc'
      } 
    },
    reset () {
      this.sortKeys = []
      this.removeUnplanned = false
    },
    getPandasDocUrl (name) {
        return `https://pandas.pydata.org/pandas-docs/version/0.25.3/reference/api/pandas.Series.${name}.html`
    }
  }
}
</script>


<style>

div.support-table {
  font-family: "Source Sans Pro", "Helvetica Neue", Arial, sans-serif;
  font-size: 10pt;
}

a {
  color: #0366d6;
  text-decoration: none;
}

table {
  /*
  border: 2px solid #42b983;
  border-radius: 3px;
  */
  background-color: #fff;
}

th {
  padding: 3px 13px;
  border: 1px solid #dfe2e5;
  /*
  background-color: #42b983;
  color: black;
  */
  cursor: pointer;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;
}

table {
  border-spacing: 0;
  border-collapse: collapse;
}

td {
  padding: 3px 13px;
  border: 1px solid #dfe2e5;
}

tr.main-row {
  border-top: 1px solid #c6cbd1;
  border-bottom: none;
}

tr.main-row:hover {
  background-color: #f6f8fa;
}

tr.example {
  border-top: none;
}

table.methods th, table.methods td {
  min-width: 120px;
  /*padding: 10px 20px;*/
}

th.active {
  color: #fff;
}

th.active .arrow {
  opacity: 1;
}

span.arrow-holder {
  position: relative;
}

.arrow {
  display: inline-block;
  vertical-align: middle;
  width: 0;
  height: 0;
  margin-left: 5px;
  opacity: 0.66;
}

.arrow.asc {
  border-left: 4px solid transparent;
  border-right: 4px solid transparent;
  border-bottom: 4px solid black;
  position: absolute;
  top: 2px;
  left: 5px;
}

.arrow.desc {
  border-left: 4px solid transparent;
  border-right: 4px solid transparent;
  border-top: 4px solid black;
  position: absolute;
  bottom: 2px;
  left: 5px;
}


/* Some additional styling taken from pandas / the Jupyter notebook CSS */
div.rendered_html table {
  border: none;
  border-collapse: collapse;
  border-spacing: 0;
  color: black;
  font-size: 12px;
  table-layout: fixed;
}
div.rendered_html thead {
  border-bottom: 1px solid black;
  vertical-align: bottom;
}
div.rendered_html tr,
div.rendered_html th,
div.rendered_html td {
  text-align: right;
  vertical-align: middle;
  padding: 0.5em 0.5em;
  line-height: normal;
  white-space: normal;
  max-width: none;
  min-width: 0;
  border: none;
  font-family: monospace;
}
div.rendered_html th {
  font-weight: bold;
}
div.rendered_html tbody tr:nth-child(odd) {
  background: #f5f5f5;
}
div.rendered_html tbody tr:hover {
  background: rgba(66, 165, 245, 0.2);
}

/* Code Background 
*/
div.input_area {
    border: 1px solid #e0e0e0;
    border-radius: 2px;
    background: #f5f5f5;
}

/* Pygments Highlighting CSS ----------------------------------------
*/

.highlight .hll { background-color: #ffffcc }
.highlight  { background: #f8f8f8; }
.highlight .c { color: #408080; font-style: italic } /* Comment */
.highlight .err { border: 1px solid #FF0000 } /* Error */
.highlight .k { color: #008000; font-weight: bold } /* Keyword */
.highlight .o { color: #666666 } /* Operator */
.highlight .ch { color: #408080; font-style: italic } /* Comment.Hashbang */
.highlight .cm { color: #408080; font-style: italic } /* Comment.Multiline */
.highlight .cp { color: #BC7A00 } /* Comment.Preproc */
.highlight .cpf { color: #408080; font-style: italic } /* Comment.PreprocFile */
.highlight .c1 { color: #408080; font-style: italic } /* Comment.Single */
.highlight .cs { color: #408080; font-style: italic } /* Comment.Special */
.highlight .gd { color: #A00000 } /* Generic.Deleted */
.highlight .ge { font-style: italic } /* Generic.Emph */
.highlight .gr { color: #FF0000 } /* Generic.Error */
.highlight .gh { color: #000080; font-weight: bold } /* Generic.Heading */
.highlight .gi { color: #00A000 } /* Generic.Inserted */
.highlight .go { color: #888888 } /* Generic.Output */
.highlight .gp { color: #000080; font-weight: bold } /* Generic.Prompt */
.highlight .gs { font-weight: bold } /* Generic.Strong */
.highlight .gu { color: #800080; font-weight: bold } /* Generic.Subheading */
.highlight .gt { color: #0044DD } /* Generic.Traceback */
.highlight .kc { color: #008000; font-weight: bold } /* Keyword.Constant */
.highlight .kd { color: #008000; font-weight: bold } /* Keyword.Declaration */
.highlight .kn { color: #008000; font-weight: bold } /* Keyword.Namespace */
.highlight .kp { color: #008000 } /* Keyword.Pseudo */
.highlight .kr { color: #008000; font-weight: bold } /* Keyword.Reserved */
.highlight .kt { color: #B00040 } /* Keyword.Type */
.highlight .m { color: #666666 } /* Literal.Number */
.highlight .s { color: #BA2121 } /* Literal.String */
.highlight .na { color: #7D9029 } /* Name.Attribute */
.highlight .nb { color: #008000 } /* Name.Builtin */
.highlight .nc { color: #0000FF; font-weight: bold } /* Name.Class */
.highlight .no { color: #880000 } /* Name.Constant */
.highlight .nd { color: #AA22FF } /* Name.Decorator */
.highlight .ni { color: #999999; font-weight: bold } /* Name.Entity */
.highlight .ne { color: #D2413A; font-weight: bold } /* Name.Exception */
.highlight .nf { color: #0000FF } /* Name.Function */
.highlight .nl { color: #A0A000 } /* Name.Label */
.highlight .nn { color: #0000FF; font-weight: bold } /* Name.Namespace */
.highlight .nt { color: #008000; font-weight: bold } /* Name.Tag */
.highlight .nv { color: #19177C } /* Name.Variable */
.highlight .ow { color: #AA22FF; font-weight: bold } /* Operator.Word */
.highlight .w { color: #bbbbbb } /* Text.Whitespace */
.highlight .mb { color: #666666 } /* Literal.Number.Bin */
.highlight .mf { color: #666666 } /* Literal.Number.Float */
.highlight .mh { color: #666666 } /* Literal.Number.Hex */
.highlight .mi { color: #666666 } /* Literal.Number.Integer */
.highlight .mo { color: #666666 } /* Literal.Number.Oct */
.highlight .sa { color: #BA2121 } /* Literal.String.Affix */
.highlight .sb { color: #BA2121 } /* Literal.String.Backtick */
.highlight .sc { color: #BA2121 } /* Literal.String.Char */
.highlight .dl { color: #BA2121 } /* Literal.String.Delimiter */
.highlight .sd { color: #BA2121; font-style: italic } /* Literal.String.Doc */
.highlight .s2 { color: #BA2121 } /* Literal.String.Double */
.highlight .se { color: #BB6622; font-weight: bold } /* Literal.String.Escape */
.highlight .sh { color: #BA2121 } /* Literal.String.Heredoc */
.highlight .si { color: #BB6688; font-weight: bold } /* Literal.String.Interpol */
.highlight .sx { color: #008000 } /* Literal.String.Other */
.highlight .sr { color: #BB6688 } /* Literal.String.Regex */
.highlight .s1 { color: #BA2121 } /* Literal.String.Single */
.highlight .ss { color: #19177C } /* Literal.String.Symbol */
.highlight .bp { color: #008000 } /* Name.Builtin.Pseudo */
.highlight .fm { color: #0000FF } /* Name.Function.Magic */
.highlight .vc { color: #19177C } /* Name.Variable.Class */
.highlight .vg { color: #19177C } /* Name.Variable.Global */
.highlight .vi { color: #19177C } /* Name.Variable.Instance */
.highlight .vm { color: #19177C } /* Name.Variable.Magic */
.highlight .il { color: #666666 } /* Literal.Number.Integer.Long */

</style>
