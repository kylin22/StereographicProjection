<template>
  <div id="diagram" @mousedown="onMouseDown" @mouseup="onMouseUp" @mousemove="onMouseMove">
    <TresCanvas>
      <TresOrthographicCamera 
        :args="[-5, 5, 5, -5, 0.1, 1000]" 
        :manual="true"
        :position="[0, 0, 5]"
        ref="camera"
      />
      <TresLine :geometry="circleGeometry" >
        <TresLineBasicMaterial :color="0x000000" />
      </TresLine>
      <TresLine :geometry="lineGeometry" ref="line">
        <TresLineBasicMaterial :color="0x000000" />
      </TresLine>
      <TresMesh v-if="intersectionPoint" :position="intersectionPoint">
        <TresCircleGeometry :args="[0.05, 64]" />
        <TresMeshBasicMaterial :color="0xff0000" />
      </TresMesh>
      <TresMesh v-if="circleIntersectionPoint" :position="circleIntersectionPoint">
        <TresCircleGeometry :args="[0.05, 64]" />
        <TresMeshBasicMaterial :color="0x00ff00" />
      </TresMesh>
      <TresMesh :position="[0, 1, 0]">
        <TresCircleGeometry :args="[0.05, 64]" />
        <TresMeshBasicMaterial :color="0xffff00" />
      </TresMesh>
      <TresLine v-if="intersectionPoint" :geometry="dynamicLineGeometry">
        <TresLineBasicMaterial :color="0x0000ff" />
      </TresLine>
    </TresCanvas>
  </div>
</template>

<script lang="ts" setup>
  import { TresCanvas } from "@tresjs/core";
  import { BufferGeometry, Vector3, Path, Raycaster, Vector2, Camera, Object3D } from "three";

  const camera = ref<Camera | null>(null);
  const line = ref<Object3D | null>(null);
  const intersectionPoint = ref<Vector3 | null>(null);
  const circleIntersectionPoint = ref<Vector3 | null>(null);
  const isDragging = ref(false);
  
  const circlePoints = new Path().absarc(0, 0, 1, 0, Math.PI * 2).getPoints(64);
  const circleGeometry = new BufferGeometry().setFromPoints(circlePoints);

  const lineGeometry = new BufferGeometry().setFromPoints([
    new Vector3(-5, -1, 0),
    new Vector3(5, -1, 0)
  ]);

  const raycaster = new Raycaster();
  const mouse = new Vector2();

  const dynamicLineGeometry = computed(() => {
    if (!intersectionPoint.value) return undefined;
    const topOfCircle = new Vector3(0, 1, 0);
    return new BufferGeometry().setFromPoints([intersectionPoint.value, topOfCircle]);
  });

  const calculateCircleIntersection = () => {
    if (!intersectionPoint.value) return null;
    const radius = 1;
    const x1 = intersectionPoint.value.x;
    const y1 = intersectionPoint.value.y;
    const x2 = 0;
    const y2 = 1;

    const dx = x2 - x1;
    const dy = y2 - y1;

    const a = dx * dx + dy * dy;
    const b = 2 * (x1 * dx + y1 * dy);
    const c = x1 * x1 + y1 * y1 - radius * radius;

    const discriminant = b * b - 4 * a * c;
    if (discriminant < 0) return null;

    const t1 = (-b + Math.sqrt(discriminant)) / (2 * a);
    const t2 = (-b - Math.sqrt(discriminant)) / (2 * a);

    const intersection1 = new Vector3(x1 + t1 * dx, y1 + t1 * dy, 0);
    const intersection2 = new Vector3(x1 + t2 * dx, y1 + t2 * dy, 0);

    //return the intersection point that is not at the top of the circle
    return intersection1.distanceTo(new Vector3(0, 1, 0)) > intersection2.distanceTo(new Vector3(0, 1, 0))
      ? intersection1
      : intersection2;
  };

  const onMouseDown = (event: MouseEvent) => {
    if (!camera.value || !line.value) return;
    if (!(event.target instanceof HTMLElement)) return;
    const rect = event.target.getBoundingClientRect();
    mouse.x = ((event.clientX - rect.left) / rect.width) * 2 - 1;
    mouse.y = -((event.clientY - rect.top) / rect.height) * 2 + 1;

    raycaster.setFromCamera(mouse, camera.value);
    const intersects = raycaster.intersectObject(line.value, true);
    if (intersects.length > 0) {
      intersectionPoint.value = intersects[0].point;
      circleIntersectionPoint.value = calculateCircleIntersection();
      isDragging.value = true;
      window.addEventListener('mousemove', onMouseMove);
      window.addEventListener('mouseup', onMouseUp);
    }
  };

  const onMouseMove = (event: MouseEvent) => {
    if (!isDragging.value || !camera.value || !line.value) return;
    const rect = document.getElementById('diagram')!.getBoundingClientRect();
    mouse.x = ((event.clientX - rect.left) / rect.width) * 2 - 1;
    mouse.y = -((event.clientY - rect.top) / rect.height) * 2 + 1;

    raycaster.setFromCamera(mouse, camera.value);
    const intersects = raycaster.intersectObject(line.value, true);
    if (intersects.length > 0) {
      intersectionPoint.value = intersects[0].point;
      circleIntersectionPoint.value = calculateCircleIntersection();
    }
  };

  const onMouseUp = () => {
    isDragging.value = false;
  };

  onUnmounted(() => {
    window.removeEventListener('mousemove', onMouseMove);
    window.removeEventListener('mouseup', onMouseUp);
  });
</script>

<style scoped lang="scss">
  #diagram {
    position: absolute;
    width: 60em;
    height: 60em;
    border: 1px solid black;
    overflow: hidden;
  }
</style>