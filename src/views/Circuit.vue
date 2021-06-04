<template>
  <div class="home"> 
    <div class="simulator" v-if="ready">
      
      <div @click="act" class="circuit-container"> 
        <h3 style="padding: 1rem"> Drawing Area </h3>
        <svg class="cables">
            <line v-for="(line, i) in lines" :key="line.id" 
            :x1="line.p1.x" :y1="line.p1.y" :x2="line.p2.x" :y2="line.p2.y"
            style="stroke:black;stroke-width:5"
            ></line>
        </svg>
        <bipole v-for="(bipole, i) in bipoles" :key="bipole.id" 
        :data="bipole" :index="i" @nodeTouched="pointSent"  >
        </bipole>
      </div> 

      <div class="commands">
        <h4>Component</h4>
        Select the component* you want to place: 
        <select v-model="action">
          <option value="resistor"> Resistor {{value}}Ω </option>
          <option value="voltage"> Voltage {{value}}V  </option>
          <option value="current"> Current {{value}}A  </option>
          <option value="connect"> Wire </option>
        </select><br>
        <p><small class="small">*You can also select the wire</small></p>
        <br>
        <br>
        <h5> Options </h5> 
        Value: 
        <input type="number" v-model="value"><br>
        <br>
        <input type="checkbox" v-model="rotated"> Vertical? (By default the component is placed horizontally)
        <br> 
        <input type="checkbox" v-model="flipped"> 
          <span v-if="rotated"> 
            Downwards? (By default the component point upwards)
          </span>
          <span v-else>
            Leftwards? (By default the component point leftward)
          </span>
          
          <br>
          <br>
        <button class="btn btn-success" style="margin-right: 10px" @click="calculate"> Calculate </button> 
        <button class="btn btn-danger" @click="clear"> Clear all </button>
        <br><br>
        <div ref="results" class="results">
        <h4> Results: </h4>
        <br>
        <table class="table table-striped" v-if="results.length">
          <thead>
            <th>Component</th>
            <th>Voltage (V)</th>
            <th>Current (A)</th>
          </thead>
          <tbody>
            <tr v-for="result in results" :key="result.id">
              <th scope="raw">{{result.id}}</th>
              <td>{{result.E}}</td>
              <td>{{result.I}}</td>
            </tr>
          </tbody>
        </table>

        </div>
      </div>
      
    </div>
    <div v-else>
      Loading...
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
// import HelloWorld from '@/components/HelloWorld.vue' 

import Bipole from '../components/Bipole.vue'; 
import { Matrix, solve } from 'ml-matrix';

let bipoles = 0;

function round(n, prec=3) {
  return Math.round(n*Math.pow(10, prec))/Math.pow(10, prec)
}

class BipoleData {
  constructor(type, value, x, y, rotated, flipped) {
    this.type = type;
    this.value = value;
    this.x = x;
    this.y = y;
    this.rotated = rotated;
    this.flipped = flipped;
    this.id = bipoles++;
    this.left = this.id+"-l";
    this.right = this.id+"-r";
    
    //vengono definiti in sede di simulazione
    //per stabilire a quale NODO è attaccato il bipolo corrente
    this.node_left = undefined;
    this.node_right = undefined;
    
  }
  matValueResistor() { 
    switch(this.type) {
      case "resistor": 
        return 1/this.value;
        break;
      case "current": 
        return 0;
        break;
      case "voltage": 
        return 0;
        break;
    }
  }
}

class Element {
  constructor(n1, n2, bipole) {
    this.n1 = n1;
    this.n2 = n2;
    this.bipole = bipole; 
  }
}

export default {
  name: 'Home',
  components: {
    // HelloWorld
    Bipole
  },
  data:()=>({
    bipoles: [
      
      // new BipoleData("resistor",1,78,247,false,false),new BipoleData("resistor",1,189,251,false,false), new BipoleData("voltage",1,122,319,false,false)
      // new BipoleData("resistor",1,122,552,false,false),new BipoleData("voltage",1,120,489,false,false)
      // new BipoleData("resistor",2,98,187,false,false),new BipoleData("resistor",4,98,251,false,false), new BipoleData("voltage",1,98,319,false,false)
      new BipoleData("resistor",2,78,247,false,false),new BipoleData("resistor",4,189,251,false,false), new BipoleData("current",1,122,319,false,false)
      // new BipoleData("resistor", 1,166,331,false,false),new BipoleData("resistor", 1,186,399,false,false),new BipoleData("resistor", 1,157,484,false,false),new BipoleData("resistor", 1,262,483,false,false),new BipoleData("resistor", 1,247,532,false,false),new BipoleData("resistor", 1,342,397,false,false),
    ],
    action: "resistor",
    lines: [
      
      // {"p1":{"x":77.5,"y":272.5,"index":"__vue_devtool_undefined__","point":"0-l"},"p2":{"x":121.5,"y":344.5,"index":"__vue_devtool_undefined__","point":"2-l"},"id":0},{"p1":{"x":172.5,"y":344.5,"index":"__vue_devtool_undefined__","point":"2-r"},"p2":{"x":239.5,"y":276.5,"index":"__vue_devtool_undefined__","point":"1-r"},"id":1},{"p1":{"x":188.5,"y":276.5,"index":"__vue_devtool_undefined__","point":"1-l"},"p2":{"x":128.5,"y":272.5,"index":"__vue_devtool_undefined__","point":"0-r"},"id":2}
      // {"p1":{"x":121.5,"y":577.5,"index":"__vue_devtool_undefined__","point":"0-l"},"p2":{"x":119.5,"y":514.5,"index":"__vue_devtool_undefined__","point":"1-l"},"id":0},{"p1":{"x":172.5,"y":577.5,"index":"__vue_devtool_undefined__","point":"0-r"},"p2":{"x":170.5,"y":514.5,"index":"__vue_devtool_undefined__","point":"1-r"},"id":1}
      // {"p1":{"x":97.5,"y":212.5,"index":"__vue_devtool_undefined__","point":"0-l"},"p2":{"x":97.5,"y":276.5,"index":"__vue_devtool_undefined__","point":"1-l"},"id":0},{"p1":{"x":97.5,"y":276.5,"index":"__vue_devtool_undefined__","point":"1-l"},"p2":{"x":97.5,"y":344.5,"index":"__vue_devtool_undefined__","point":"2-l"},"id":1},{"p1":{"x":148.5,"y":212.5,"index":"__vue_devtool_undefined__","point":"0-r"},"p2":{"x":148.5,"y":276.5,"index":"__vue_devtool_undefined__","point":"1-r"},"id":2},{"p1":{"x":148.5,"y":276.5,"index":"__vue_devtool_undefined__","point":"1-r"},"p2":{"x":148.5,"y":344.5,"index":"__vue_devtool_undefined__","point":"2-r"},"id":3}
      {"p1":{"x":77.5,"y":272.5,"index":"__vue_devtool_undefined__","point":"0-l"},"p2":{"x":121.5,"y":344.5,"index":"__vue_devtool_undefined__","point":"2-l"},"id":0},{"p1":{"x":172.5,"y":344.5,"index":"__vue_devtool_undefined__","point":"2-r"},"p2":{"x":239.5,"y":276.5,"index":"__vue_devtool_undefined__","point":"1-r"},"id":1},{"p1":{"x":188.5,"y":276.5,"index":"__vue_devtool_undefined__","point":"1-l"},"p2":{"x":128.5,"y":272.5,"index":"__vue_devtool_undefined__","point":"0-r"},"id":2}
      // {"p1":{"x":165.5,"y":356.5,"index":"__vue_devtool_undefined__","point":"0-l"},"p2":{"x":185.5,"y":424.5,"index":"__vue_devtool_undefined__","point":"1-l"},"id":0},{"p1":{"x":185.5,"y":424.5,"index":"__vue_devtool_undefined__","point":"1-l"},"p2":{"x":156.5,"y":509.5,"index":"__vue_devtool_undefined__","point":"2-l"},"id":1},{"p1":{"x":156.5,"y":509.5,"index":"__vue_devtool_undefined__","point":"2-l"},"p2":{"x":246.5,"y":557.5,"index":"__vue_devtool_undefined__","point":"4-l"},"id":2},{"p1":{"x":297.5,"y":557.5,"index":"__vue_devtool_undefined__","point":"4-r"},"p2":{"x":312.5,"y":508.5,"index":"__vue_devtool_undefined__","point":"3-r"},"id":3},{"p1":{"x":312.5,"y":508.5,"index":"__vue_devtool_undefined__","point":"3-r"},"p2":{"x":392.5,"y":422.5,"index":"__vue_devtool_undefined__","point":"5-r"},"id":4},{"p1":{"x":341.5,"y":422.5,"index":"__vue_devtool_undefined__","point":"5-l"},"p2":{"x":216.5,"y":356.5,"index":"__vue_devtool_undefined__","point":"0-r"},"id":5},{"p1":{"x":216.5,"y":356.5,"index":"__vue_devtool_undefined__","point":"0-r"},"p2":{"x":236.5,"y":424.5,"index":"__vue_devtool_undefined__","point":"1-r"},"id":6},{"p1":{"x":236.5,"y":424.5,"index":"__vue_devtool_undefined__","point":"1-r"},"p2":{"x":312.5,"y":508.5,"index":"__vue_devtool_undefined__","point":"3-r"},"id":7},{"p1":{"x":261.5,"y":508.5,"index":"__vue_devtool_undefined__","point":"3-l"},"p2":{"x":207.5,"y":509.5,"index":"__vue_devtool_undefined__","point":"2-r"},"id":8}
    ], 
    points: [],
    rotated: false,
    flipped: false,
    value: 1,
    ready: true,
    results: [],
    isplacingwire: false,
  }),
  async mounted() {
    // this.ready = false;
    // await languagePluginLoader;
    // this.ready = true;
  },
  computed: {
    //controllati in corrente (o comunque non in tensione)
    //come i generatore di tensione
    cc: {
      get: function() {
        let list = [];
        for(let bipole of this.bipoles) {
          if(bipole.type == "voltage") {
            list.push(bipole)
          }
        }

        return list;
      }
    }
  },
  methods: {
    act(event) {  
      if(this.action!="connect") {
        this.bipoles.push(new BipoleData(this.action, this.value, event.x+(-25*this.rotated), event.y+(-25*!this.rotated), this.rotated, this.flipped))
      }      
      
    },
    pointSent(event) {
      this.action = 'connect';
      let {data, index, point} = event;
      let {x, y} = event;
  
      this.points.push({x,y, index, point})
       
      if(!this.isplacingwire) {
        this.isplacingwire = true;
      }
      else {
        let circles = document.querySelectorAll('circle');
        for(let circle of circles) {
          circle.style.fill = "black";
        }
        this.isplacingwire = false;
      }

      if(this.points.length == 2) this.createLine();
       
    },
    createLine() {
      if(this.points.length == 2) {
        this.lines.push({p1:this.points[0] , p2:this.points[1], id:this.lines.length});
        this.points.pop();
        this.points.pop(); 
      }
    },
    clear() {
      
      this.results.splice(0, this.results.length);
      this.lines.splice(0, this.lines.length);
      this.bipoles.splice(0, this.bipoles.length);
      bipoles = 0;
    },
    calculate() { 

      /*
      ogni bipolo è attaccato a un nodo
      ogni nodo però può essere semplificato
      in "node_left", "node_right" ci sono 
      i riferimenti ai nodi dopo che il circuito 
      viene semplificato (contengono gli "indici" dei nodi)
      quindi quando si calcola la simulazione bisogna 
      "resettare" il contenuto di questi dati, in modo
      da poter inserire il riferimento al nodo corretto
      una volta che il circuito viene semplificato
      */
      
      
      for(let bipole of this.bipoles) {
        bipole.node_left = undefined;
        bipole.node_right = undefined;
      }

      let nodo_morsetto = this.trova_morsetti(this.lines); 
      let nodes_elements = this.nodes_elements(nodo_morsetto); 

      let nodes_bipole = nodes_elements.map(x => x.map(b=>[this.bipoles[parseInt(b[0])], b[0], b[1]]));
 

      let matrix = this.get_matrix(nodes_bipole);
      let known = this.get_known(nodes_bipole); 

      
      console.log(matrix);
      console.log(known);
      
      let A = new Matrix(matrix);
      let b = Matrix.columnVector(known);


      let e = solve(A, b).data.map(x=>x[0]);
      
      let N = nodes_bipole.length; 

      e.splice(N-1, 0, 0);//massa

      console.log(matrix);
      console.log(known);
      console.log(e);

      this.results.splice(0, this.results.length)
 
      for(let bipolo of this.bipoles) {
        let l = nodo_morsetto[bipolo.left];
        let r = nodo_morsetto[bipolo.right];
        
        let E = round(e[r]-e[l]); 
        let R = round(bipolo.value);
        let I = round(E/R);

        let comp = '?';

        switch(bipolo.type) {
          case "resistor":
            comp = 'R'; 
            this.results.push({id:comp+bipolo.id, E, I})
            break;
          case "voltage":
            comp = 'E'; 
            break;
          case "current":
            comp = 'J'; 
            break;
        } 
        
        
      }
    },
 
 

    trova_morsetti(lines) {
      let nodo_morsetto = {};
      let morsetto = 0; 
      for(let line of lines) {
        let n1 = line.p1.point;
        let n2 = line.p2.point;
        
        if(nodo_morsetto[n1]===undefined && nodo_morsetto[n2]===undefined) {
          nodo_morsetto[n1] = morsetto;
          nodo_morsetto[n2] = morsetto;
          morsetto++;
          continue;
        }
        if(nodo_morsetto[n1]===undefined && nodo_morsetto[n2]!==undefined) {
          nodo_morsetto[n1] = nodo_morsetto[n2];
          continue;
        }
        if(nodo_morsetto[n1]!==undefined && nodo_morsetto[n2]===undefined) {
          nodo_morsetto[n2] = nodo_morsetto[n1];
          continue;
        }
        if(nodo_morsetto[n1]!==undefined && nodo_morsetto[n2]!==undefined) {
          if(nodo_morsetto[n1] !== nodo_morsetto[n2]) {
            let morsetto_da_cambiare = nodo_morsetto[n2];
            let morsetto_da_usare = nodo_morsetto[n1];
            for(let nodo in nodo_morsetto) {
              if(nodo_morsetto[nodo] == morsetto_da_cambiare) {
                nodo_morsetto[nodo] = morsetto_da_usare;
              }
            }
          }
          continue;
        }

      } 
      return nodo_morsetto
    },

    nodes_elements(nodo_morsetto) { 
      let morsetto_elemento = {};

      for(let nodo in nodo_morsetto) {
        let morsetto = nodo_morsetto[nodo];
        if(!morsetto_elemento[morsetto]) morsetto_elemento[morsetto] = [];

        let element_info = nodo.split("-"); 
        //[numero elemento, destra/sx, nome comepleto]
        let e = [parseInt( element_info[0]),  element_info[1], nodo]; 

        //inserimento dei riferimenti al nodo nell'elemento
        if(element_info && element_info[0]) {
          let bipole = this.bipoles[parseInt( element_info[0] )]
          if(bipole) { 
            if(element_info[1] == "l") {
              bipole.node_left = morsetto;
            }
            if(element_info[1] == "r") {
              bipole.node_right = morsetto;
            }
          }
        }
       

        
          //non ha senso includere i morsetti che iniziano e finiscono nello stesso morsetto
          //non ha senso includere i componenti che sono legati solo da una parte
        if(nodo_morsetto[e[0]+"-l"]!==undefined && nodo_morsetto[e[0]+"-r"]!==undefined && 
          nodo_morsetto[e[0]+"-l"]!==nodo_morsetto[e[0]+"-r"]) { 
          morsetto_elemento[morsetto].push(e);
        }
      }

      let node_elements = [];
 

      //viene inserito il contenuto in "node_elements" come array anziché dizionario di morsetti
      for(let n in morsetto_elemento) {
        node_elements.push(   morsetto_elemento[n]  );

        //rinomino il morsetto corrispondente ai vari elementi con il nuovo indice dell'array
        for(let e of morsetto_elemento[n]) {
          nodo_morsetto[e[2]] = node_elements.length-1;
        }
      }


      return node_elements;


    },

    get_matrix(nodes_bipole) {
      let N = nodes_bipole.length;

      let Mat = [];
      for(var i=0; i<N-1; i++)
        Mat[i] = new Array(N-1);
 


      let resistance = (acc, val) => acc+val[0].matValueResistor();
 

      for(let j = 0; j<N-1; j++) {
        for(let k=0; k<N-1; k++) {
          if(j==k) {
            Mat[j][j] = nodes_bipole[j].map(val=>val[0].matValueResistor())
                                       .reduce((a, c)=>a+c, 0);
          }
          else {
            //creo due set con gli indici connessi ai nodi j e k
            let setJ = new Set(nodes_bipole[j].map(x=>x[1]));
            let setK = new Set(nodes_bipole[k].map(x=>x[1]));

            //metto solo gli indici connessi a entrambi j e k
            let nodes = [...setJ].filter(x => setK.has(x));
            Mat[j][k] = nodes.map(x=>this.bipoles[x].matValueResistor())
                             .reduce((a, c)=>a-c, 0);
          }
        }
      }
 
      //qui aggiungo le colonne per gli elementi contr. in corr.
      for(let nodo = 0; nodo<N-1; nodo++) { 
        for(let bipole of this.cc) {
          
          let val = 0;
          if(bipole.node_right == nodo) val += 1; //somma perchè se connesso sinistro e destro somma = 0
          if(bipole.node_left == nodo) val += -1;
          Mat[nodo].push(val);
        }
      }
      //qui aggiungo le righe per gli elementi contr. in corr
      for(let bipole of this.cc) {
        let raw = new Array(N-1+this.cc.length).fill(0);
        for(let nodo = 0; nodo<N-1; nodo++) {
          let val = 0;
          if(bipole.node_right == nodo) val += 1; //somma perchè se connesso sinistro e destro somma = 0
          if(bipole.node_left == nodo) val += -1;
          raw[nodo] = val;
        }
        Mat.push(raw);
      }
 
      return Mat;
      
    },

    get_known(nodes_bipole) {
      let N = nodes_bipole.length;
      let b = [];
      
      for(let i = 0; i < N-1; i++) {
        let sum = 0;
        for(let bipole of nodes_bipole[i]) {
          if(bipole[0].type === "current") {
            let coefficient = bipole[2]=="r"?1:-1;
            sum += coefficient*bipole[0].value;
          } 
        }
        b.push(sum);
      }

      for(let bipole of this.cc) {
        b.push(parseFloat(bipole.value));
      }

 
      return b;
    },
  }
}
</script>


<style lang="scss">

.simulator {
  display: grid;
  grid-template-columns: 60% 40%;
  height: 100vh;
  width: 100vw; 
  position: relative;
}

.cables {
  // height: 75vh;
  // width: 100vw;
  
  width: 100%;
  height: 100%;
  position: absolute;
    top: 0;
    left: 0;
}

.circuit-container {
  width: 100%;
  height: 100%;
  // height: 75vh; 
  position: relative;
  box-shadow: 0px 0px 20px -5px black;

  .bipole {
    position: absolute;
    
    &.flipped:not(&.rotated) {
      transform: rotate(180deg);
    }
    &.rotated:not(&.flipped) {
      transform: rotate(-90deg);
    }
    &.rotated.flipped {
      transform: rotate(90deg);
    }
  }


  // svg.cables {
  //   position: absolute;
  //   top: 0;
  //   left: 0;
  //   height: 100%;
  //   width: 100%;
  // }

  
}

.commands {
  padding: 2rem;
  overflow-y: auto;
}

.small, small {
    font-size: .675em;
}
</style>