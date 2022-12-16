<script>
// @ts-nocheck
import debounce from 'lodash.debounce';

let container;
let copy
let quality = 1;
let fileinp;
let filec = null;
// converts any svg component (paths,ellipses,etc) to an array of points
function svgToPoints(svg) {
  let points = [];
  let paths = svg.querySelectorAll("path");
  paths = [...paths,...svg.querySelectorAll("circle")];
  paths = [...paths,...svg.querySelectorAll("ellipse")];
  paths = [...paths,...svg.querySelectorAll("rect")];
  paths = [...paths,...svg.querySelectorAll("line")];
  paths = [...paths,...svg.querySelectorAll("polyline")];
  paths = [...paths,...svg.querySelectorAll("polygon")];
  console.log(paths);
  paths.forEach((path) => {
    let pathLength = path.getTotalLength();
    let pathPoints = [];
    for (let i = 0; i < pathLength; i += quality) {
      let point = path.getPointAtLength(i);
      pathPoints.push([point.x, point.y]);
    }
    points.push(pathPoints);
  });
  return points;
}
 
//function that takes an svgToPoints array and returns an svg element
function pointsToSvg(points,prevSVG) {
  let svg = document.createElementNS("http://www.w3.org/2000/svg", "svg");
  svg.setAttribute("viewBox", prevSVG.getAttribute("viewBox"));
  svg.setAttribute("fill", "none");
  svg.setAttribute("stroke", "black");
  svg.setAttribute("stroke-width", "0.5");
  svg.setAttribute("preserveAspectRatio", "xMidYMid meet");

  points.forEach((point) => {
    let path = document.createElementNS("http://www.w3.org/2000/svg", "path");
    path.setAttribute("d", `M${point[0][0]} ${point[0][1]} L${point
      .slice(1)
      .map((point) => `${point[0]} ${point[1]}`)
      .join(" ")}`);
    svg.appendChild(path);
  });
  return svg;
}

function handleFile(file = filec){
  copy.value = "Loading...";
  filec = file;
  console.log(filec)
    file.target.files[0].text().then((text)=>{
      // console.log(text);
      container.innerHTML = text;
      let points = (svgToPoints(container.querySelector("svg")));
      container.innerHTML = pointsToSvg(points,container.querySelector("svg")).outerHTML;
      let count = 0;
      let segments = points.map((point)=>{
          return new Array(point.length).fill(0).map(()=>{
            return count++;
          })
      })
      console.log({vertexes:points,segments:segments});

      copy.value = `meshes={{vertexes=${JSON.stringify(points.reduce((acc,pointar)=>{
        return [...acc,...pointar];
      },[])).replaceAll('[','{').replaceAll(']','}')},segments=${JSON.stringify(segments).replaceAll('[','{').replaceAll(']','}')}}}`

    })
}

</script>

<main>
  <input bind:this={fileinp} type="file" name="svg" id="svgfile" accept=".svg" on:change={(file)=>{
    // console.log(file.target.files[0]);
    handleFile(file);
}} />
    <label for="quality" style="background:white;">Sharpness (lesser is better quality but larger)</label>

 <input type="number" name="quality" min="1" max="200" bind:value={quality} on:change={
  debounce(()=>{
    handleFile()
  },400)
}>
    <div bind:this={container} id="svgcontainer"/>
    <textarea name="something" id="copy" cols="30" rows="10" bind:this={copy}></textarea>
   
    <button on:click={
      ()=>{
        copy.select();
        copy.setSelectionRange(0, 99999);

        navigator.clipboard.writeText(copy.value);
      }
    }>Copy</button>
    <footer>
      <p>Sorry about this, but the output is actually upside down. So make sure to either add <code>pewpew.customizable_entity_add_rotation_to_mesh(background, fmath.tau() / 2fx, 0fx, 0fx, 0fx)</code> or whatever rotation value you want on the mesh.</p>
    </footer>
</main>

<style>
main{

  min-height: 100vh;

  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background-image: url("data:image/svg+xml,<svg id='patternId' width='100%' height='100%' xmlns='http://www.w3.org/2000/svg'><defs><pattern id='a' patternUnits='userSpaceOnUse' width='40' height='40' patternTransform='scale(2) rotate(65)'><rect x='0' y='0' width='100%' height='100%' fill='hsla(0, 23%, 19%, 1)'/><path d='M27.26 5.415c-.55 0-.9.55-.65 1l2.45 4.75c.2.5.85.5 1.15.1l3.15-4.5c.3-.4 0-1.05-.55-1.1zM10.689 8.068c-.406-.051-.822.31-.778.748l.5 5.3c.05.5.6.8 1.05.5l4.55-3.05c.45-.3.4-.95-.05-1.15l-5.1-2.3a.605.605 0 0 0-.172-.048zM2.406 24.584a.635.635 0 0 0-.345.081l-4.75 2.4c-.45.2-.5.85-.1 1.15l4.45 3.15c.4.3 1 0 1.1-.5l.3-5.55c0-.412-.31-.712-.655-.73zm40 0a.635.635 0 0 0-.345.081l-4.75 2.4c-.45.2-.5.85-.1 1.15l4.45 3.15c.4.3 1 0 1.1-.5l.3-5.55c0-.412-.31-.712-.655-.73zm-22.17 3.108a.744.744 0 0 0-.675.723l.4 5.55c.05.5.6.8 1.05.5l4.45-2.95c.45-.25.4-.9-.05-1.15l-4.8-2.6a.702.702 0 0 0-.376-.073z'  stroke-width='1' stroke='none' fill='hsla(259, 60%, 38%, 1)'/></pattern></defs><rect width='800%' height='800%' transform='translate(-68,-102)' fill='url(%23a)'/></svg>")
}
input{
  padding: 10px;
}
#svgcontainer{
  height:25vh;
  width: 25vw;
  background-color: white;
  display: flex;
  align-items: center;
  justify-content: center;

}
p{
  width:80vw;
  max-width: 700px;
  padding: 20px 0;
  background-color: white;
  line-break: loose;
  word-wrap: break-word;
}
@media (max-width: 1000px){
  #svgcontainer{
    width: 70vw;
  }
}
@media (max-width: 800px){
  #svgcontainer{
    width: 90vw;
  }
}

</style>
