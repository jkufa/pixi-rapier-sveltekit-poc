<script lang="ts">
	import type { Collider, World } from "@dimforge/rapier2d";
	import { Application, Color, Graphics } from "pixi.js";
	import { onMount } from "svelte";
  let element: HTMLDivElement;
  let app: Application<HTMLCanvasElement>;
    const cubeSize = 64;
    onMount(() => {
      import('@dimforge/rapier2d').then(RAPIER => {    
          // Use the RAPIER module here.
          // Rapier world settings
          const sprites: Graphics[] = [];
          const scaleFactor = 50;
          const numBodies = 1;
          let gravity = new RAPIER.Vector2(0.0, -9.81 * scaleFactor); //Band-aid solution for slow gravity bug
          let world = new RAPIER.World(gravity);
      
          // Create the ground
          let groundBodyDesc = RAPIER.RigidBodyDesc.fixed().setTranslation(0, -window.innerHeight);
          let groundBody = world.createRigidBody(groundBodyDesc);
          let groundColliderDesc = RAPIER.ColliderDesc.cuboid(window.innerWidth, cubeSize + 10);
          world.createCollider(groundColliderDesc, groundBody);

          // Create a dynamic rigid-body.
          let rigidBodyDesc = RAPIER.RigidBodyDesc.dynamic().setTranslation(0.0, 1.0);
          let rigidBody = world.createRigidBody(rigidBodyDesc);
      
          // Create a cuboid collider attached to the dynamic rigidBody.
          let colliderDesc = RAPIER.ColliderDesc.cuboid(cubeSize, cubeSize);
          const collider = world.createCollider(colliderDesc, rigidBody);

          const colliderMap = new Map();
          
          // Pixi canvas 
          app = new Application({
            background: '#0A020F',
            antialias: true,
            resizeTo: window
            // width: 800,
            // height: 600,
          });
          element.appendChild(app.view);
          
          const graphic = new Graphics();
          // Cuboid graphic
          graphic.beginFill(0xBCA3CC);
          graphic.drawRoundedRect(0.5, 0.5, cubeSize, cubeSize, 2);
          graphic.endFill();
          graphic.x = app.screen.width / 2;
          graphic.y = app.screen.height / 2;
          app.stage.addChild(graphic);
          sprites.push(graphic);

          
         world.forEachCollider(col => {
          addCollider(col);
         });
         requestAnimationFrame(update);

         function addCollider(collider: Collider) {
          const { x, y } = collider.translation();
          colliderMap.set(collider.handle, {
            type: RAPIER.ShapeType.Cuboid,
            xLoc: x,
            yLoc: y,
            rotation: collider.rotation()
          });
         }

        function updatePositions(world: World) {
            world.forEachCollider((elt) => {
                let CMapHandle = colliderMap.get(elt.handle);
                let translation = elt.translation();
                let rotation = elt.rotation();
                if (!!CMapHandle) {
                    CMapHandle.xLoc = translation.x;
                    CMapHandle.yLoc = translation.y;
                    CMapHandle.rotation = -rotation;
                }
            });
          }

        function render(world: World) {
          let cntr = 0;
          colliderMap.forEach((el) => {
            graphic.beginFill(0x0000ff);
              let curr = sprites[cntr];
              cntr = (cntr + 1) % numBodies;
              curr.position.x = el.xLoc + 100;
              curr.position.y = -el.yLoc + 100;
              curr.rotation = el.rotation;
              curr.pivot.set(curr.width / 2, curr.height / 2);
          });
        }

        function update() {
          // graphics.clear();
          render(world);
          renderDebug();
          updatePositions(world);
          world.step();
          requestAnimationFrame(update);
        }

        let GRAPHICS = [];
        function renderDebug () {
          const { vertices, colors } = world.debugRender();

          GRAPHICS.forEach(g => g.destroy());
          GRAPHICS = [];

          for (let i = 0; i < vertices.length / 4; i += 1) {
            const g = new Graphics();
            const c = new Color({
              r: colors[i * 4] * 255,
              g: colors[i * 4 + 1] * 255,
              b: colors[i * 4 + 2] * 255,
              a: colors[i * 4 + 3] * 255
            });

            g.lineStyle(0.5, c, 1);
            g.moveTo(vertices[i * 4],     vertices[i * 4 + 1]);
            g.lineTo(vertices[i * 4 + 2], vertices[i * 4 + 3]);
            g.closePath();

            GRAPHICS.push(g);
            app.stage.addChild(g);
          }
        }
      });
    });
</script>
<div bind:this={element} id="pixi"></div>
