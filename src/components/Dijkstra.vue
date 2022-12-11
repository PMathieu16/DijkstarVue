<template>
  <div id="all">
    <h1> Dijkstra</h1>
    <div id="container"></div>
  </div>
</template>

<script>
import * as Three from 'three';
export default {
  name: 'Dijkstra',
  data () {
    return {
      scene:undefined,
      camera:undefined,
      renderer:undefined
    }
  },
  methods: {
    init: function() {
        let container = document.getElementById('container');
        let tailleTab = 16
        this.camera = new Three.PerspectiveCamera(70, container.clientWidth/container.clientHeight, 0.01, 20);
        this.camera.position.z = 15;
        this.camera.position.x = tailleTab/2;
        this.camera.position.y = tailleTab/2;

        this.scene = new Three.Scene();
        let tab =[]
        let tile = []

        const geometry = new Three.PlaneGeometry( 1, 1 );
        const terre = new Three.MeshBasicMaterial( {color: 0x32CD32, side: Three.DoubleSide} );
        const lave = new Three.MeshBasicMaterial( {color: 0xed0000, side: Three.DoubleSide} );
        //generation map
        for (let i = 0; i < tailleTab; i++) {
          tab[i] = []; 
          tile[i]=[];
          for (let j = 0; j < tailleTab; j++) {
            let mat = terre
            tab[i][j] = "terre";
            if(Math.floor(Math.random() * 3)==0){ 
              mat = lave
              tab[i][j] = "lave";
            }
            tile[i][j] = new Three.Mesh( geometry, mat )
            this.scene.add(tile[i][j]);
            //console.log(i + "  " +j)
            tile[i][j].position.x = i
            tile[i][j].position.y = j
          }
        }
        // generation clé
        let xkey = Math.floor(Math.random() * tailleTab)
        let ykey = Math.floor(Math.random() * tailleTab)
        tab[xkey][ykey] = "key" 
        tile[xkey][ykey].material = new Three.MeshBasicMaterial( {color: 0xffff00, side: Three.DoubleSide} );
        //generation hero
        let x = xkey 
        let y = ykey
        while(x == xkey && y == ykey){
          x = Math.floor(Math.random() * tailleTab)
          y = Math.floor(Math.random() * tailleTab)
        }
        tab[x][y] = "Hero" 
        tile[x][y].material = new Three.MeshBasicMaterial( {color: 0xf4fefe, side: Three.DoubleSide} );
//------------------------------------------------------------------------------------------------------------------
     
//------------------------------------------------------------------------------------------------------------------
      function HighlightCheckCase(scene, tiletab, colortile) {
        // Pour chaque position de la liste, on crée un plan et on l'ajoute à la scène
        for (let i = 0; i < tiletab.length; i++) {
          const geometry = new Three.PlaneGeometry( 0.7, 0.7 );
          const mat = new Three.MeshBasicMaterial( {color: colortile, side: Three.DoubleSide} );
          const plane = new Three.Mesh( geometry, mat )
          scene.add(plane);
          plane.position.x = tiletab[i][0]
          plane.position.y = tiletab[i][1]
        }
      }

      const getCases = (map, x, y) => {
        // Créer un tableau pour stocker les cases adjacentes
        const cases = [];
        // Vérifie les cases adjacentes dans chaque direction
        if (x > 0 && map[x - 1][y] !== "lave") {
          cases.push([x - 1, y]);
        }
        if (x < map.length - 1 && map[x + 1][y] !== "lave") {
          cases.push([x + 1, y]);
        }
        if (y > 0 && map[x][y - 1] !== "lave") {
          cases.push([x, y - 1]);
        }
        if (y < map[x].length - 1 && map[x][y + 1] !== "lave") {
          cases.push([x, y + 1]);
        }
        // Renvoie le tableau des cases adjacentes
        return cases;
      }

      const sleep = (ms) => {
          return new Promise(resolve => setTimeout(resolve, ms));
      }

      const findShortestWay = async (map, heroX, heroY, scene) => {
        // Créer une file vide pour stocker les cases à traiter
        const queue = [];
        // Marque la position initiale du héro comme visitée
        const visited = new Set([heroX + "," + heroY]);
        // Ajoute la position initiale du héro à la queue
        queue.push([heroX, heroY]);
        // Créer un tableau pour stocker les informations sur le chemin vers chaque case
        const path = new Array(map.length);
        for (let i = 0; i < path.length; i++) {
          path[i] = new Array(map[i].length);
        }
        // Boucle tant que la queue n'est pas vide
        while (queue.length > 0) {
          // Récupére la première case de la queue
          const currentPos = queue.shift();
          const x = currentPos[0];
          const y = currentPos[1];
          // Vérifie si la case courante est la clé
          if (map[x][y] === "key") {
            // Chemin trouvé
            // Retrace le chemin depuis la case courante jusqu'à la case initiale
            let curX = x;
            let curY = y;
            const shortestPath = [];
            while (curX !== heroX || curY !== heroY) {
              // Ajoute la case courante au chemin
              shortestPath.unshift([curX, curY]);
              // Récupére la position précédente dans le chemin
              const prevPos = path[curX][curY];
              curX = prevPos[0];
              curY = prevPos[1];
            }
            // Ajoute la position initiale au chemin
            shortestPath.unshift([heroX, heroY]);

            HighlightCheckCase(scene, shortestPath, 0x00ffff);
            // Renvoie le chemin le plus court
            return shortestPath;
          }
          // Récupére les cases adjacentes à la case courante
          const cases = getCases(map, x, y);

          //Animation de recherche
          HighlightCheckCase(scene, cases, 0x800080)
          await sleep(25)

          // Pour chaque case adjacente à la case courante...
          for (const c of cases) {
            const nx = c[0];
            const ny = c[1];
            // Si la case adjacente n'a pas été visitée...
            if (!visited.has(nx + "," + ny)) {
              // Marque la case adjacente comme visitée
              visited.add(nx + "," + ny);
              // Ajoute la case adjacente à la queue
              queue.push([nx, ny]);
              // Enregistre la position de la case courante dans le chemin vers la case adjacente
              path[nx][ny] = [x, y];
            }
          }
        }
        // Renvoie null si aucun chemin n'a été trouvé
        return null;
      }

      findShortestWay(tab, x, y, this.scene).then(res => {
        if (res) {
          console.log("clé trouvé")
        } else {
          console.log("clé inaccessible")
        }
      })

      this.renderer = new Three.WebGLRenderer({antialias: true});
      this.renderer.setSize(container.clientWidth, container.clientHeight);
      container.appendChild(this.renderer.domElement)

    },
    animate: function() {
        requestAnimationFrame(this.animate);
        //this.mesh.rotation.x += 0.01;
        //this.mesh.rotation.y += 0.02;
        this.renderer.render(this.scene, this.camera);
    },


  },
  mounted() {
      this.init();
      this.animate();
  }

}
</script>

<style scoped>
  #container{
    width: 1000px;
    height: 700px;
    margin-left: auto;
    margin-right: auto;
  }

</style>
