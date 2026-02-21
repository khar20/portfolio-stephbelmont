<script lang="ts">
    import { onMount } from "svelte";
    import * as THREE from "three";
    import { gsap } from "gsap";

    let canvasContainer: HTMLDivElement;

    onMount(() => {
        const scene = new THREE.Scene();

        const camera = new THREE.PerspectiveCamera(
            35,
            window.innerWidth / window.innerHeight,
            0.1,
            1000,
        );
        camera.position.z = 8;
        camera.position.x = 0;

        const renderer = new THREE.WebGLRenderer({
            alpha: true,
            antialias: true,
        });
        renderer.setSize(window.innerWidth, window.innerHeight);
        renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));

        renderer.toneMapping = THREE.ACESFilmicToneMapping;
        renderer.toneMappingExposure = 1.2;

        canvasContainer.appendChild(renderer.domElement);

        const geometry = new THREE.TorusKnotGeometry(1.2, 0.4, 150, 20);
        const material = new THREE.MeshPhysicalMaterial({
            color: 0x111111,
            roughness: 0.15,
            metalness: 0.1,
            clearcoat: 1.0,
            clearcoatRoughness: 0.1,
            side: THREE.DoubleSide,
        });

        const torus = new THREE.Mesh(geometry, material);

        torus.position.set(1.5, 0, 0);

        torus.rotation.set(0.5, -0.5, 0);

        scene.add(torus);

        const ambientLight = new THREE.AmbientLight(0xffffff, 1);
        scene.add(ambientLight);

        const rectLight = new THREE.RectAreaLight(0xffffff, 5, 10, 10);
        rectLight.position.set(5, 5, 5);
        rectLight.lookAt(0, 0, 0);
        scene.add(rectLight);

        const blueRim = new THREE.SpotLight(0x4444ff, 10);
        blueRim.position.set(-5, -5, 2);
        scene.add(blueRim);

        gsap.fromTo(
            torus.rotation,
            { y: -Math.PI, x: 1 },
            {
                y: -0.2,
                x: 0.4,
                duration: 3.5,
                ease: "power3.out",
            },
        );

        gsap.from(torus.scale, {
            x: 0,
            y: 0,
            z: 0,
            duration: 2.5,
            ease: "elastic.out(1, 0.75)",
        });

        function animate() {
            requestAnimationFrame(animate);
            renderer.render(scene, camera);
        }

        animate();

        const handleResize = () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);

            if (window.innerWidth < 900) {
                torus.position.set(0, 1, 0);
            } else {
                torus.position.set(1.5, 0, 0);
            }
        };

        handleResize();

        window.addEventListener("resize", handleResize);

        return () => {
            window.removeEventListener("resize", handleResize);
            renderer.dispose();
            geometry.dispose();
            material.dispose();
        };
    });
</script>

<div bind:this={canvasContainer} class="webgl-container"></div>

<style>
    .webgl-container {
        width: 100%;
        height: 100%;
        display: block;
    }
</style>
