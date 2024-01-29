<script>
    import {onMount} from "svelte";
    import * as THREE from 'three';
    import {OrbitControls} from 'three/addons/controls/OrbitControls.js';
    import {GLTFLoader} from 'three/addons/loaders/GLTFLoader.js';
    import Modal from "./lib/Modal.svelte";
    let showModal = false;
    let meshName = null;
    onMount(async () => {

        const scene = new THREE.Scene();
        const camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);

        const renderer = new THREE.WebGLRenderer();
        renderer.setSize(window.innerWidth, window.innerHeight);
        document.getElementById("canvas").appendChild(renderer.domElement);

        const ambientLight = new THREE.AmbientLight(0xcccccc, 3);
        scene.add(ambientLight);
        const directionalLight = new THREE.DirectionalLight(0xffffff, 10);
        directionalLight.position.set(1, 1, 0).normalize();
        scene.add(directionalLight);

        const loader = new GLTFLoader();
        loader.load('apple.glb', function (gltf) {
            scene.add(gltf.scene);
        }, undefined, function (error) {
            console.error(error);
        });

        camera.position.set(0, 0.18, 0.2);
        camera.lookAt(new THREE.Vector3(0, 0, 0));

        const controls = new OrbitControls(camera, renderer.domElement);
        controls.enableDamping = true;
        controls.dampingFactor = 0.05;
        controls.screenSpacePanning = false;
        controls.maxPolarAngle = Math.PI / 2;

        const raycaster = new THREE.Raycaster();
        const mouse = new THREE.Vector2();

        function onMouseClick(event) {
            mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
            mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
            raycaster.setFromCamera(mouse, camera);
            const intersects = raycaster.intersectObjects(scene.children, true);
            if (intersects.length > 0) {
                console.log('Object clicked:', intersects[0].object);
                console.log(intersects[0].object.name);
                showModal = true;
                meshName = intersects[0].object.name;
            }
        }

        window.addEventListener('click', onMouseClick, false);

        function animate() {
            requestAnimationFrame(animate);
            controls.update();
            renderer.render(scene, camera);
        }

        animate();
    });
</script>
<div id="canvas"></div>
<Modal bind:showModal>
    <div slot="mesh-name">{meshName}</div>
</Modal>
<style>
</style>