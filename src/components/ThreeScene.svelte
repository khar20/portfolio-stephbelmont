<script>
    import { onMount } from "svelte";
    import * as THREE from "three";

    let canvasContainer;

    onMount(() => {
        // scene setup
        const scene = new THREE.Scene();
        scene.fog = new THREE.FogExp2(0x000000, 0.02);

        const camera = new THREE.PerspectiveCamera(
            75,
            window.innerWidth / window.innerHeight,
            0.1,
            1000,
        );
        camera.position.z = 5;

        const renderer = new THREE.WebGLRenderer({
            alpha: true,
            antialias: true,
        });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
        canvasContainer.appendChild(renderer.domElement);

        // star field
        const particlesGeometry = new THREE.BufferGeometry();
        const particlesCount = 2000;
        const posArray = new Float32Array(particlesCount * 3);

        for (let i = 0; i < particlesCount * 3; i++) {
            posArray[i] = (Math.random() - 0.5) * 20;
        }

        particlesGeometry.setAttribute(
            "position",
            new THREE.BufferAttribute(posArray, 3),
        );

        const particlesMaterial = new THREE.PointsMaterial({
            size: 0.02,
            color: 0xffffff,
            transparent: true,
            opacity: 0.8,
            blending: THREE.AdditiveBlending,
        });

        const particlesMesh = new THREE.Points(
            particlesGeometry,
            particlesMaterial,
        );
        scene.add(particlesMesh);

        // main model
        const geoGeometry = new THREE.IcosahedronGeometry(1.5, 0);
        const geoMaterial = new THREE.MeshStandardMaterial({
            color: 0xffffff,
            roughness: 0.1,
            metalness: 0.1,
            flatShading: true,
        });

        const geoMesh = new THREE.Mesh(geoGeometry, geoMaterial);
        geoMesh.position.set(3, 0, 0);
        scene.add(geoMesh);

        // lighting
        const ambientLight = new THREE.AmbientLight(0xffffff, 0.2);
        scene.add(ambientLight);

        const mainLight = new THREE.DirectionalLight(0xffffff, 2);
        mainLight.position.set(2, 2, 5);
        scene.add(mainLight);

        const fillLight = new THREE.PointLight(0x8888ff, 0.5);
        fillLight.position.set(-5, 0, 2);
        scene.add(fillLight);

        // mouse interaction
        let mouseX = 0;
        let mouseY = 0;
        let targetX = 0;
        let targetY = 0;

        const windowHalfX = window.innerWidth / 2;
        const windowHalfY = window.innerHeight / 2;

        const onDocumentMouseMove = (event) => {
            mouseX = event.clientX - windowHalfX;
            mouseY = event.clientY - windowHalfY;
        };

        document.addEventListener("mousemove", onDocumentMouseMove);

        // animation loop
        const clock = new THREE.Clock();

        function animate() {
            const elapsedTime = clock.getElapsedTime();

            targetX = mouseX * 0.001;
            targetY = mouseY * 0.001;

            // rotation
            geoMesh.rotation.y += 0.005;
            geoMesh.rotation.x += 0.002;

            // floating
            geoMesh.position.y = Math.sin(elapsedTime * 0.5) * 0.2;

            // parallax stars
            particlesMesh.rotation.y = elapsedTime * 0.05;
            particlesMesh.rotation.x = mouseY * 0.00005;
            particlesMesh.position.x +=
                (mouseX * 0.0005 - particlesMesh.position.x) * 0.05;
            particlesMesh.position.y +=
                (-mouseY * 0.0005 - particlesMesh.position.y) * 0.05;

            renderer.render(scene, camera);
            requestAnimationFrame(animate);
        }

        animate();

        const handleResize = () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);

            if (window.innerWidth < 768) {
                geoMesh.position.set(0, 1.5, 0);
                geoMesh.scale.set(0.8, 0.8, 0.8);
            } else {
                geoMesh.position.set(3, 0, 0);
                geoMesh.scale.set(1, 1, 1);
            }
        };

        window.addEventListener("resize", handleResize);
        handleResize();

        return () => {
            window.removeEventListener("resize", handleResize);
            document.removeEventListener("mousemove", onDocumentMouseMove);
            renderer.dispose();
        };
    });
</script>

<div bind:this={canvasContainer} class="webgl-container"></div>

<style>
    .webgl-container {
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        z-index: 1;
    }
</style>
