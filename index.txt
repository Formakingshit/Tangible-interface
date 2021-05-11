const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 2000);
camera.position.set(0, 0, -10);

const renderer = new THREE.WebGLRenderer();
renderer.setSize(window.innerWidth, window.innerHeight);

let orbitControls = new THREE.OrbitControls(camera, renderer.domElement);

let windowHalfX = window.innerWidth / 2;
let windowHalfY = window.innerHeight / 2;

let geometry
let meshBasicMaterial
let mesh;

init();
animate();


function toDegrees(i) {
  return i * Math.PI / 180;
}

function surface(v, u, target) {
  v = v * (89 - 1) + 1
  u = u * (630 + 630) - 630

  const C = 2
  const a = 2 / (C + 1 - C * Math.pow(Math.sin(toDegrees(v)), 2)) * Math.pow(Math.cos(toDegrees(u)), 2)

  const r = (a / Math.sqrt(C)) * Math.sqrt((C + 1) * (1 + C * Math.pow(Math.sin(toDegrees(u)), 2))) * Math.sin(toDegrees(v))
  const fi = (-u / Math.sqrt(C + 1)) + Math.atan(Math.sqrt(C + 1) * Math.tan(toDegrees(u)))
  const calculatedX = r * Math.cos(toDegrees(fi))
  const calculatedY = r * Math.sin(toDegrees(fi))
  const calculatedZ = (Math.log(Math.tan(toDegrees(v / 2))) + a * (C + 1) * Math.cos(toDegrees(v))) / Math.sqrt(C)

  target.set(calculatedX, calculatedY, calculatedZ);
};

function initGeometry() {
  geometry = new THREE.ParametricGeometry(surface, 80, 80);
};

function init() {
  container = document.createElement('div');
  document.body.appendChild(container);


  THREE.ImageUtils.crossOrigin = '';

  const path = "https://stereocamera.s3.eu-central-1.amazonaws.com/";
  const format = '.png';
  const urls = [
    path + 'px' + format, path + 'nx' + format,
    path + 'py' + format, path + 'ny' + format,
    path + 'pz' + format, path + 'nz' + format
  ];

  const textureCube = new THREE.CubeTextureLoader().load(urls);

  scene.background = textureCube;

  initGeometry();
  meshBasicMaterial = new THREE.MeshBasicMaterial({ color: 0xffffff, envMap: textureCube });
  mesh = new THREE.Mesh(geometry, meshBasicMaterial);
  scene.add(mesh);

  renderer.setPixelRatio(window.devicePixelRatio);
  container.appendChild(renderer.domElement);

  const width = window.innerWidth || 2;
  const height = window.innerHeight || 2;

  effect = new THREE.AnaglyphEffect(renderer);
  effect.setSize(width, height);

  window.addEventListener('resize', onWindowResize);
  window.addEventListener('deviceorientation', onDeviceOrientation);
}

function onWindowResize() {

  windowHalfX = window.innerWidth / 2;
  windowHalfY = window.innerHeight / 2;

  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();

  effect.setSize(window.innerWidth, window.innerHeight);
}

function onDeviceOrientation(event) {
  const m = new THREE.Matrix4();
  m.set(...getRotationMatrix(event.alpha, event.beta, event.gamma));

  geometry.applyMatrix4(m);
}

function getRotationMatrix(alpha, beta, gamma) {
  var degtorad = Math.PI / 180 / 90;

  var _x = beta ? beta * degtorad : 0; // beta value
  var _y = gamma ? gamma * degtorad : 0; // gamma value
  var _z = alpha ? alpha * degtorad : 0; // alpha value

  var cX = Math.cos(_x);
  var cY = Math.cos(_y);
  var cZ = Math.cos(_z);
  var sX = Math.sin(_x);
  var sY = Math.sin(_y);
  var sZ = Math.sin(_z);

  //
  // ZXY rotation matrix construction.
  //

  var m11 = cZ * cY - sZ * sX * sY;
  var m12 = - cX * sZ;
  var m13 = cY * sZ * sX + cZ * sY;

  var m21 = cY * sZ + cZ * sX * sY;
  var m22 = cZ * cX;
  var m23 = sZ * sY - cZ * cY * sX;

  var m31 = - cX * sY;
  var m32 = sX;
  var m33 = cX * cY;

  return [
    m11, m12, m13, 0, 
    m21, m22, m23, 0,
    m31, m32, m33, 0,
    0,   0,   0,   1
  ];

};

function animate() {

  requestAnimationFrame(animate);

  render();

}

function render() {
  effect.render(scene, camera);
}
